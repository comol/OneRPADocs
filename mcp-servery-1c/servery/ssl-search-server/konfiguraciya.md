# Конфигурация SSLSearchServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `SSL_VERSION` | Версия БСП | `3.1.11` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `true` |
| `RESET_CACHE` | Перезагрузить embedding модель | `true` |

### Транспорт

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `USESSE` | SSE транспорт | `false` |

### Embedding модели (LM Studio / Ollama)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API сервера | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API | `lm-studio` |
| `OPENAI_MODEL` | Название модели | `Qwen3-Embedding-4B` |

## Монтируемые тома

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/bases/mcp_ssl` | `/app/chroma_db` | Векторная база данных |

## Доступные версии БСП

Сервер поддерживает различные версии БСП. Укажите вашу версию в `SSL_VERSION`.

Примеры:
- `3.1.11`
- `3.1.9`
- `3.2.1`
- `2.4.6`

## Примеры конфигураций

### Минимальная (CPU)

```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  comol/mcp_ssl_server:latest
```

### Рекомендуемая (LM Studio)

```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_ssl:/app/chroma_db" `
  comol/mcp_ssl_server:latest
```

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-ssl-mcp": {
      "url": "http://localhost:8008/mcp",
      "connection_id": "1c_ssl_service_001"
    }
  }
}
```
