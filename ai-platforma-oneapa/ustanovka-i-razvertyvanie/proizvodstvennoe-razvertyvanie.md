# Производственное развёртывание

Рекомендации и инструкции по развёртыванию OneAPA в производственной среде.

## Архитектура production-среды

### Рекомендуемая схема

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Production Environment                           │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                      Load Balancer (nginx)                       │    │
│  │                        SSL Termination                           │    │
│  └─────────────────────────┬───────────────────────────────────────┘    │
│                            │                                             │
│            ┌───────────────┼───────────────┐                            │
│            ▼               ▼               ▼                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                     │
│  │  Proxy #1   │  │  Proxy #2   │  │  Proxy #3   │                     │
│  │  (Docker)   │  │  (Docker)   │  │  (Docker)   │                     │
│  └─────────────┘  └─────────────┘  └─────────────┘                     │
│            │               │               │                            │
│            └───────────────┼───────────────┘                            │
│                            ▼                                             │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    1С Cluster / Server                             │  │
│  │                      + OneAPA                                      │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │
│  │   Monitoring    │  │    Logging      │  │   Backup Storage        │  │
│  │   (Prometheus)  │  │   (ELK Stack)   │  │                         │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

## SSL/TLS настройка

### Nginx как reverse proxy

Конфигурация `/etc/nginx/sites-available/oneapa`:

```nginx
upstream oneapa_proxy {
    least_conn;
    server proxy1:9000 weight=1;
    server proxy2:9000 weight=1;
    server proxy3:9000 weight=1;
}

server {
    listen 80;
    server_name oneapa.company.local;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name oneapa.company.local;

    # SSL сертификаты
    ssl_certificate /etc/nginx/ssl/oneapa.crt;
    ssl_certificate_key /etc/nginx/ssl/oneapa.key;
    
    # SSL настройки
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    
    # Заголовки безопасности
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    
    # Проксирование API
    location / {
        proxy_pass http://oneapa_proxy;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Таймауты
        proxy_connect_timeout 60s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;
    }
    
    # Health check endpoint
    location /health {
        proxy_pass http://oneapa_proxy/health;
        proxy_connect_timeout 5s;
        proxy_read_timeout 5s;
    }
}
```

### Самоподписанный сертификат (для тестирования)

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/oneapa.key \
  -out /etc/nginx/ssl/oneapa.crt \
  -subj "/CN=oneapa.company.local"
```

## Балансировка нагрузки

### Docker Swarm

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  proxy:
    image: oneapa-proxy:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          memory: 4G
          cpus: '2'
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
    networks:
      - oneapa-net

  nginx:
    image: nginx:latest
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - proxy
    deploy:
      placement:
        constraints:
          - node.role == manager
    networks:
      - oneapa-net

networks:
  oneapa-net:
    driver: overlay
```

### Kubernetes

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oneapa-proxy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: oneapa-proxy
  template:
    metadata:
      labels:
        app: oneapa-proxy
    spec:
      containers:
      - name: proxy
        image: oneapa-proxy:latest
        ports:
        - containerPort: 9000
        env:
        - name: LICENSE_KEY
          valueFrom:
            secretKeyRef:
              name: oneapa-secrets
              key: license-key
        resources:
          limits:
            memory: "4Gi"
            cpu: "2"
          requests:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 9000
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health
            port: 9000
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: oneapa-proxy-service
spec:
  selector:
    app: oneapa-proxy
  ports:
  - port: 9000
    targetPort: 9000
  type: ClusterIP
```

## Мониторинг

### Prometheus metrics

Добавьте endpoint `/metrics` в Proxy для сбора метрик:

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'oneapa-proxy'
    static_configs:
      - targets: ['proxy1:9000', 'proxy2:9000', 'proxy3:9000']
    metrics_path: /metrics
    scrape_interval: 15s
```

### Grafana Dashboard

Основные метрики для мониторинга:

| Метрика | Описание | Алерт |
|---------|----------|-------|
| `oneapa_requests_total` | Общее количество запросов | - |
| `oneapa_request_duration_seconds` | Время обработки запросов | > 30s |
| `oneapa_llm_calls_total` | Вызовы LLM | - |
| `oneapa_llm_errors_total` | Ошибки LLM | > 10/min |
| `oneapa_tool_executions_total` | Выполнение инструментов | - |

### Health Checks

```bash
# Скрипт проверки здоровья
#!/bin/bash

HOSTS=("proxy1" "proxy2" "proxy3")

for host in "${HOSTS[@]}"; do
    response=$(curl -s -o /dev/null -w "%{http_code}" http://$host:9000/health)
    if [ "$response" != "200" ]; then
        echo "ALERT: $host is unhealthy (HTTP $response)"
        # Отправка уведомления
    fi
done
```

## Логирование

### Централизованное логирование (ELK)

```yaml
# docker-compose.logging.yml
version: '3.8'

services:
  proxy:
    image: oneapa-proxy:latest
    logging:
      driver: "fluentd"
      options:
        fluentd-address: "fluentd:24224"
        tag: "oneapa.proxy"

  fluentd:
    image: fluent/fluentd:v1.14-1
    volumes:
      - ./fluentd.conf:/fluentd/etc/fluent.conf
    ports:
      - "24224:24224"
```

### Конфигурация Fluentd

```xml
<source>
  @type forward
  port 24224
</source>

<match oneapa.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix oneapa
</match>
```

## Резервное копирование

### Стратегия резервного копирования

| Компонент | Частота | Хранение |
|-----------|---------|----------|
| База 1С | Ежедневно | 30 дней |
| Конфигурация Proxy | При изменениях | 10 версий |
| Логи | Ежедневно | 90 дней |
| Docker images | При обновлении | 5 версий |

### Скрипт резервного копирования

```bash
#!/bin/bash

BACKUP_DIR="/backup/oneapa/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup Docker images
docker save oneapa-proxy:latest | gzip > $BACKUP_DIR/oneapa-proxy.tar.gz

# Backup configuration
cp -r /opt/oneapa-proxy/*.py $BACKUP_DIR/
cp /opt/oneapa-proxy/requirements.txt $BACKUP_DIR/

# Backup logs
cp -r /var/log/oneapa/* $BACKUP_DIR/logs/

# Cleanup old backups (keep 30 days)
find /backup/oneapa -type d -mtime +30 -exec rm -rf {} \;

echo "Backup completed: $BACKUP_DIR"
```

## Безопасность

### Чек-лист безопасности

- [ ] SSL/TLS для всех соединений
- [ ] Firewall настроен (только необходимые порты)
- [ ] API ключи в секретах (не в коде)
- [ ] Логирование всех действий
- [ ] Регулярное обновление зависимостей
- [ ] Мониторинг аномалий
- [ ] Резервное копирование

### Хранение секретов

**Docker Secrets:**

```bash
# Создание секрета
echo "your_license_key" | docker secret create oneapa_license -

# Использование в compose
services:
  proxy:
    secrets:
      - oneapa_license
    environment:
      - LICENSE_KEY_FILE=/run/secrets/oneapa_license
```

**Kubernetes Secrets:**

```bash
kubectl create secret generic oneapa-secrets \
  --from-literal=license-key=your_license_key
```

## Обновление в production

### Rolling Update

```bash
#!/bin/bash

# Сборка нового образа
docker build -t oneapa-proxy:new .

# Проверка нового образа
docker run --rm oneapa-proxy:new python -c "import main; print('OK')"

# Rolling update (Docker Swarm)
docker service update --image oneapa-proxy:new oneapa_proxy

# Или для Kubernetes
kubectl set image deployment/oneapa-proxy proxy=oneapa-proxy:new
```

### Откат

```bash
# Docker Swarm
docker service rollback oneapa_proxy

# Kubernetes
kubectl rollout undo deployment/oneapa-proxy
```

## Отказоустойчивость

### Рекомендации

1. **Минимум 3 инстанса Proxy** для отказоустойчивости
2. **Health checks** с автоматическим перезапуском
3. **Таймауты** на всех уровнях
4. **Circuit breaker** для LLM вызовов
5. **Graceful shutdown** при обновлениях

### Тестирование отказоустойчивости

```bash
# Chaos testing - остановка случайного инстанса
docker stop $(docker ps -q --filter "name=proxy" | shuf -n 1)

# Проверка продолжения работы
curl https://oneapa.company.local/health
```

## Оптимизация производительности

### Настройки Python

```bash
# Увеличение количества worker'ов
uvicorn main:app --workers 4 --host 0.0.0.0 --port 9000
```

### Настройки Docker

```yaml
services:
  proxy:
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: '2'
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

## Далее

- [Мониторинг](../administrirovanie/monitoring.md)
- [Логирование](../administrirovanie/logirovanie.md)
- [Резервное копирование](../administrirovanie/rezervnoe-kopirovanie.md)
