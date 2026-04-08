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
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `INDEX_CODE` | Индексация BSL кода. При отключении убираются: `codesearch`, `search_function`, `get_module_structure`, `get_method_call_hierarchy`, `search_metadata_forms`, `get_metadata_form_details`, `graph_dependencies`, `bsl_scope_members` | `true` |
| `INDEX_METADATA` | Индексация метаданных. При отключении убираются: `metadatasearch`, `get_metadata_details` | `true` |
| `INDEX_HELP` | Индексация HTML-справки конфигурации. При отключении убирается: `helpsearch` | `true` |
| `INDEX_METADATA_XML` | Индексация XML-определений метаданных. При отключении убирается: `search_metadata_xml` | `true` |
| `INDEX_XSD_SCHEMAS` | Генерация XSD-схем из конфигурации. При отключении убираются: `get_xsd_schema`, `verify_xml` | `true` |
| `INDEX_BATCH_SIZE` | Размер пакета при добавлении записей в ChromaDB. Увеличение ускоряет индексацию, но требует больше памяти | `25` |
| `CHUNK_SIZE` | Размер фрагмента текста при разбивке. Больше — меньше фрагментов, но менее точный поиск. Меньше — больше фрагментов и точнее поиск | `1000` |

### Периодическая переиндексация

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `REINDEX_INTERVAL_HOURS` | Интервал автоматической переиндексации в часах. Сервер периодически проверяет изменения файлов и обновляет индексы | `24` |

### Транспорт

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `USESSE` | SSE транспорт (для legacy клиентов) | `false` |

### Embedding модели (LM Studio / Ollama)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API сервера. При использовании OpenRouter: `https://openrouter.ai/api`. Суффикс `/v1` добавляется автоматически | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API (для LM Studio — любой, для OpenRouter — ваш ключ) | `lm-studio` |
| `OPENAI_MODEL` | Название модели (для OpenRouter: `qwen/qwen3-embedding-8b`) | `Qwen3-Embedding-4B` |
| `EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов. Поддерживается моделями с переменной размерностью (Qwen3, text-embedding-3). Если не указано — определяется автоматически | *(авто)* |

{% hint style="info" %}
**Поддержка OpenRouter:** Сервер автоматически определяет OpenRouter по URL и добавляет необходимые HTTP-заголовки (`HTTP-Referer`, `X-Title`).
{% endhint %}

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

### С OpenRouter (облачные эмбеддинги)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=https://openrouter.ai/api `
  -e OPENAI_API_KEY=YOUR_OPENROUTER_KEY `
  -e OPENAI_MODEL=qwen/qwen3-embedding-8b `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```

### С настройкой индексации

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e INDEX_CODE=true `
  -e INDEX_METADATA=true `
  -e INDEX_HELP=true `
  -e INDEX_BATCH_SIZE=25 `
  -e CHUNK_SIZE=1000 `
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
