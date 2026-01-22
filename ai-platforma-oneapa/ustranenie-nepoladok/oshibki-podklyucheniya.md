# Ошибки подключения

Диагностика и устранение сетевых проблем.

## 1С не подключается к Proxy

### Симптомы

- Ошибка при проверке подключения
- Ошибка при обмене с Proxy
- Таймаут при отправке сообщения

### Диагностика

```bash
# С сервера 1С проверьте доступность Proxy
curl http://proxy:9000/health

# Если не работает - проверьте:
# 1. Proxy запущен
# 2. Правильный IP/hostname
# 3. Порт открыт
```

### Решения

| Причина | Решение |
|---------|---------|
| Proxy не запущен | `docker start oneapa-proxy` |
| Неверный URL | Проверьте настройки в 1С |
| Firewall | Откройте порт 9000 |
| Разные сети | Используйте IP вместо hostname |

## Proxy не подключается к LLM

### Симптомы

- Ошибки в логах Proxy
- Timeout при запросе к агенту

### Диагностика

```bash
# Из контейнера Proxy проверьте доступ к API
docker exec oneapa-proxy curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer sk-..."
```

### Решения

| Причина | Решение |
|---------|---------|
| Нет интернета | Проверьте сеть контейнера |
| Прокси-сервер | Настройте HTTP_PROXY |
| DNS | Проверьте DNS в контейнере |

## Proxy не подключается к 1С

### Симптомы

- Инструменты не выполняются
- Ошибка "Connection refused"

### Диагностика

```bash
# Из Proxy проверьте доступ к 1С HTTP сервису
curl http://1c-server/hs/oneapa/tools
```

### Решения

| Причина | Решение |
|---------|---------|
| HTTP сервис не опубликован | Опубликуйте в 1С |
| Неверный URL | Проверьте toolsendpoint |
| Авторизация | Настройте доступ к сервису |

## Проверка сети

### Между компонентами

```
1С → Proxy:
  curl http://proxy:9000/health

Proxy → LLM:
  curl https://api.openai.com/v1/models

Proxy → 1С:
  curl http://1c/hs/oneapa/tools
```

### Firewall

```bash
# Linux
sudo iptables -L | grep 9000

# Windows
netsh firewall show state
```

## Типичные конфигурации

### Всё на одном сервере

```
1С: localhost
Proxy: localhost:9000
```

### Раздельные серверы

```
1С: 192.168.1.10
Proxy: 192.168.1.20:9000
URL в 1С: http://192.168.1.20:9000
toolsendpoint: http://192.168.1.10/hs/oneapa/tools
```
