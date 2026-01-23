# Установка TemplatesSearchServer

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен (рекомендуется)

## Создание папки для данных

```powershell
New-Item -ItemType Directory -Force -Path "E:\bases\mcp_templates"
```

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_templates:/app/data" `
  comol/template-search-mcp:latest
```

### С CPU (без GPU)

```powershell
docker run -d -p 8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=ai-forever/FRIDA `
  -v "E:/bases/mcp_templates:/app/data" `
  comol/template-search-mcp:latest
```

## Переменные окружения

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `true` |
| `RESET_CACHE` | Перезагрузить модель | `true` |
| `EMBEDDING_MODEL` | CPU модель | `ai-forever/FRIDA` |

## Первый запуск

При первом запуске:
1. Скачивается образ
2. Индексируются встроенные шаблоны
3. Запускается веб-интерфейс

### Мониторинг

```powershell
docker logs -f template_search_mcp
```

## Проверка работы

### MCP endpoint

```powershell
docker ps --filter name=template_search_mcp
```

### Веб-интерфейс

Откройте в браузере: `http://localhost:8004/extend/`

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-templates-mcp": {
      "url": "http://localhost:8004/mcp",
      "connection_id": "1c_templates_service_001"
    }
  }
}
```
