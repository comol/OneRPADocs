# Конфигурация CodeMetadataSearchServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `METADATA_PATH` | Путь к отчету в контейнере | `/app/metadata` |
| `CODE_PATH` | Путь к коду в контейнере | `/app/code` |

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

### Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face | `intfloat/multilingual-e5-base` |

## Монтируемые тома

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/1C_Export/Report` | `/app/metadata` | Отчет из конфигурации |
| `E:/1C_Export/Files` | `/app/code` | Выгрузка в файлы |
| `E:/bases/mcp_codemetadata` | `/app/chroma_db` | Векторная база данных |

## Примеры конфигураций

### Минимальная (CPU)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  comol/1c_code_metadata_mcp:latest
```

### Рекомендуемая (LM Studio)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-code-metadata-mcp": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_metadata_service_001"
    }
  }
}
```
