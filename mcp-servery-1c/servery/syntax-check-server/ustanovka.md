# Установка SyntaxCheckServer

## Предварительные требования

- Docker Desktop запущен

## Команда запуска

```powershell
docker run -d -p 8002:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest
```

Это всё! Сервер не требует дополнительной настройки.

## Переменные окружения

| Переменная | Описание | Обязательная |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да |
| `USESSE` | SSE транспорт для legacy клиентов | Нет |

### SSE транспорт

Если ваш клиент требует SSE:

```powershell
docker run -d -p 8002:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e USESSE=true `
  comol/1c_syntaxcheck_mcp:latest
```

## Проверка работы

### Статус контейнера

```powershell
docker ps --filter name=1c_syntaxcheck_mcp
```

### Просмотр логов

```powershell
docker logs 1c_syntaxcheck_mcp
```

## Конфигурация Cursor

### mcp.json

```json
{
  "mcpServers": {
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    }
  }
}
```

## Управление контейнером

```powershell
# Остановить
docker stop 1c_syntaxcheck_mcp

# Запустить
docker start 1c_syntaxcheck_mcp

# Перезапустить
docker restart 1c_syntaxcheck_mcp

# Удалить
docker rm -f 1c_syntaxcheck_mcp
```

## Устранение проблем

### Контейнер не запускается

```powershell
docker logs 1c_syntaxcheck_mcp
```

### Порт занят

```powershell
netstat -ano | findstr :8002
```

Если порт занят, используйте другой порт:

```powershell
docker run -d -p 8102:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest
```

И измените порт в mcp.json: `http://localhost:8102/mcp`
