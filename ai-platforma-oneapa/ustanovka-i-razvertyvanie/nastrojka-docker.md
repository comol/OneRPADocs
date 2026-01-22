# Настройка Docker

Подробная инструкция по настройке Docker для Proxy-сервера OneAPA.

## Dockerfile

### Содержимое стандартного Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Установка системных зависимостей
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Копирование файла зависимостей
COPY requirements.txt .

# Установка Python зависимостей
RUN pip install --no-cache-dir -r requirements.txt

# Копирование исходного кода
COPY . .

# Порт приложения
EXPOSE 9000

# Запуск приложения
CMD ["python", "main.py"]
```

### Оптимизированный Dockerfile

```dockerfile
# Этап сборки
FROM python:3.11-slim as builder

WORKDIR /app

# Установка зависимостей для сборки
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Копирование и установка зависимостей
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Этап выполнения
FROM python:3.11-slim

WORKDIR /app

# Копирование установленных пакетов
COPY --from=builder /root/.local /root/.local

# Добавление в PATH
ENV PATH=/root/.local/bin:$PATH

# Копирование исходного кода
COPY . .

# Создание пользователя
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# Порт приложения
EXPOSE 9000

# Healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:9000/health || exit 1

# Запуск приложения
CMD ["python", "main.py"]
```

## Сборка образа

### Базовая сборка

```bash
docker build -t oneapa-proxy:latest .
```

### Сборка с тегом версии

```bash
docker build -t oneapa-proxy:1.0.1 .
docker tag oneapa-proxy:1.0.1 oneapa-proxy:latest
```

### Сборка без кэша

```bash
docker build --no-cache -t oneapa-proxy:latest .
```

## Запуск контейнера

### Базовый запуск

```bash
docker run -d \
  --name oneapa-proxy \
  -p 9000:9000 \
  -e LICENSE_KEY=your_license_key \
  oneapa-proxy:latest
```

### Полная конфигурация

```bash
docker run -d \
  --name oneapa-proxy \
  --hostname oneapa-proxy \
  -p 9000:9000 \
  -e LICENSE_KEY=your_license_key \
  -e LOG_LEVEL=INFO \
  -v /var/log/oneapa:/app/log \
  -v /etc/localtime:/etc/localtime:ro \
  --restart unless-stopped \
  --memory=4g \
  --cpus=2 \
  oneapa-proxy:latest
```

### Параметры запуска

| Параметр | Описание |
|----------|----------|
| `-d` | Запуск в фоновом режиме |
| `--name` | Имя контейнера |
| `-p 9000:9000` | Проброс порта |
| `-e VAR=value` | Переменная окружения |
| `-v host:container` | Монтирование тома |
| `--restart` | Политика перезапуска |
| `--memory` | Лимит памяти |
| `--cpus` | Лимит CPU |

## Docker Compose

### docker-compose.yml

```yaml
version: '3.8'

services:
  oneapa-proxy:
    build: .
    image: oneapa-proxy:latest
    container_name: oneapa-proxy
    ports:
      - "9000:9000"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - LOG_LEVEL=INFO
    volumes:
      - ./log:/app/log
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: '2'
        reservations:
          memory: 1G
          cpus: '0.5'

networks:
  default:
    name: oneapa-network
```

### Файл .env для Docker Compose

```env
LICENSE_KEY=your_license_key
```

### Команды Docker Compose

```bash
# Запуск
docker-compose up -d

# Остановка
docker-compose down

# Перезапуск
docker-compose restart

# Просмотр логов
docker-compose logs -f

# Пересборка и запуск
docker-compose up -d --build
```

## Работа с Ollama

### Docker Compose с Ollama

```yaml
version: '3.8'

services:
  oneapa-proxy:
    build: .
    image: oneapa-proxy:latest
    container_name: oneapa-proxy
    ports:
      - "9000:9000"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
    depends_on:
      - ollama
    restart: unless-stopped

  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama
    restart: unless-stopped
    # Для GPU раскомментируйте:
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: 1
    #           capabilities: [gpu]

volumes:
  ollama-data:

networks:
  default:
    name: oneapa-network
```

### Загрузка модели в Ollama

```bash
# Подключение к контейнеру
docker exec -it ollama ollama pull llama3

# Проверка
docker exec -it ollama ollama list
```

## Сетевая конфигурация

### Создание сети

```bash
docker network create oneapa-network
```

### Подключение контейнеров к сети

```bash
docker run -d \
  --name oneapa-proxy \
  --network oneapa-network \
  -p 9000:9000 \
  oneapa-proxy:latest
```

### Связь между контейнерами

Контейнеры в одной сети могут обращаться друг к другу по имени:

```
# Из контейнера oneapa-proxy к ollama
http://ollama:11434
```

## Мониторинг и логирование

### Просмотр логов

```bash
# Все логи
docker logs oneapa-proxy

# Последние 100 строк
docker logs --tail 100 oneapa-proxy

# В реальном времени
docker logs -f oneapa-proxy

# С временными метками
docker logs -t oneapa-proxy
```

### Настройка логирования Docker

```bash
docker run -d \
  --name oneapa-proxy \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  oneapa-proxy:latest
```

### Мониторинг ресурсов

```bash
# Статистика
docker stats oneapa-proxy

# Информация о контейнере
docker inspect oneapa-proxy
```

## Обновление

### Скрипт обновления (Linux)

```bash
#!/bin/bash

# Остановка текущего контейнера
docker stop oneapa-proxy

# Сохранение старого образа
docker tag oneapa-proxy:latest oneapa-proxy:backup

# Сборка нового образа
docker build -t oneapa-proxy:latest .

# Удаление старого контейнера
docker rm oneapa-proxy

# Запуск нового контейнера
docker run -d \
  --name oneapa-proxy \
  -p 9000:9000 \
  -e LICENSE_KEY=$LICENSE_KEY \
  --restart unless-stopped \
  oneapa-proxy:latest

# Проверка
sleep 5
curl http://localhost:9000/health
```

### Скрипт обновления (Windows PowerShell)

```powershell
# rebuild-container.ps1

# Остановка
docker stop oneapa-proxy

# Сборка
docker build -t oneapa-proxy:latest .

# Удаление
docker rm oneapa-proxy

# Запуск
docker run -d `
  --name oneapa-proxy `
  -p 9000:9000 `
  -e LICENSE_KEY=$env:LICENSE_KEY `
  --restart unless-stopped `
  oneapa-proxy:latest

# Проверка
Start-Sleep -Seconds 5
Invoke-RestMethod http://localhost:9000/health
```

## Резервное копирование

### Экспорт образа

```bash
docker save oneapa-proxy:latest | gzip > oneapa-proxy-backup.tar.gz
```

### Импорт образа

```bash
docker load < oneapa-proxy-backup.tar.gz
```

## Устранение проблем

### Контейнер не запускается

```bash
# Проверьте логи
docker logs oneapa-proxy

# Запустите в интерактивном режиме
docker run -it --rm oneapa-proxy:latest /bin/bash
```

### Проблемы с сетью

```bash
# Проверьте сеть
docker network ls
docker network inspect oneapa-network

# Проверьте порты
docker port oneapa-proxy
```

### Нехватка ресурсов

```bash
# Проверьте использование
docker stats

# Очистка неиспользуемых ресурсов
docker system prune -a
```

## Далее

{% content-ref url="proizvodstvennoe-razvertyvanie.md" %}
[proizvodstvennoe-razvertyvanie.md](proizvodstvennoe-razvertyvanie.md)
{% endcontent-ref %}
