# Конфигурация CloudEmbeddingsServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `EMBEDDING_PROVIDER` | Провайдер эмбеддингов: `openrouter`, `openai`, `cohere`, `jina`, `local` | `openrouter` |
| `SOURCE_PATH` | Путь к исходным файлам конфигурации внутри контейнера | `/data/source` |

### API-ключи провайдеров

Укажите ключ для выбранного провайдера:

| Переменная | Описание | Когда нужен |
|------------|----------|-------------|
| `OPENROUTER_API_KEY` | Ключ OpenRouter | `EMBEDDING_PROVIDER=openrouter` |
| `OPENAI_API_KEY` | Ключ OpenAI | `EMBEDDING_PROVIDER=openai` |
| `COHERE_API_KEY` | Ключ Cohere | `EMBEDDING_PROVIDER=cohere` |
| `JINA_API_KEY` | Ключ Jina AI | `EMBEDDING_PROVIDER=jina` |

### Настройки индексации

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `AUTO_INDEX` | Автоматическая индексация при запуске | `true` |
| `CHUNK_SIZE` | Размер фрагмента текста (символы) при разбивке | `1000` |
| `CHUNK_OVERLAP` | Перекрытие между фрагментами (символы) | `100` |
| `MAX_BATCH_SIZE` | Количество фрагментов, накапливаемых перед отправкой в API эмбеддингов | `100` |
| `DEFAULT_SEARCH_LIMIT` | Количество результатов поиска по умолчанию | `10` |

### Параллельная обработка

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `EMBEDDING_CONCURRENCY` | Количество параллельных HTTP-запросов к API эмбеддингов. Значение `10` ускоряет индексацию в ~8 раз | `1` |
| `EMBEDDING_BATCH_SIZE` | Количество текстов в одном API-запросе. В сочетании с `CONCURRENCY=10` обрабатывается 100 текстов за раунд | `10` |

{% hint style="info" %}
**Рекомендуемые значения для OpenRouter:** `EMBEDDING_CONCURRENCY=10`, `EMBEDDING_BATCH_SIZE=10`. Это даёт ~240 запросов/мин и индексацию за 1–2 часа.
{% endhint %}

{% hint style="warning" %}
Высокие значения `EMBEDDING_CONCURRENCY` могут вызвать ошибки rate-limit у провайдера. Начните с `5` и увеличивайте при отсутствии ошибок.
{% endhint %}

### Модель эмбеддингов

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `EMBEDDING_MODEL` | Переопределение модели по умолчанию для выбранного провайдера | *(зависит от провайдера)* |
| `OPENAI_API_BASE` | Пользовательский базовый URL для OpenAI-совместимых API (LM Studio, Ollama) | — |

Модели по умолчанию для провайдеров:

| Провайдер | Модель по умолчанию |
|-----------|-------------------|
| `openrouter` | `qwen/qwen3-embedding-8b` |
| `openai` | `text-embedding-3-small` |
| `cohere` | `embed-multilingual-v3.0` |
| `jina` | `jina-embeddings-v3` |
| `local` | `intfloat/multilingual-e5-small` |

### Сервер

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `HOST` | Хост для привязки сервера | `0.0.0.0` |
| `PORT` | Порт сервера | `8000` |
| `MCP_PORT` | Внешний порт в docker-compose | `8000` |
| `USESSE` | SSE транспорт (для legacy клиентов) | `false` |

### Пути

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `SOURCE_PATH` | Путь к исходным файлам 1С | `/data/source` |
| `CHROMA_PATH` | Путь к хранилищу ChromaDB | `/data/chroma_db` |

## Монтируемые тома

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/1C_Export/Files` | `/data/source` | Исходные файлы конфигурации (только чтение) |
| `E:/bases/mcp_cloud` | `/data/chroma_db` | Векторная база данных ChromaDB |

## Примеры конфигураций

### OpenRouter с параллельной индексацией

```powershell
docker run -d -p 8000:8000 `
  --name 1c_cloud_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_PROVIDER=openrouter `
  -e OPENROUTER_API_KEY=YOUR_OPENROUTER_KEY `
  -e EMBEDDING_MODEL=qwen/qwen3-embedding-8b `
  -e EMBEDDING_CONCURRENCY=10 `
  -e EMBEDDING_BATCH_SIZE=10 `
  -e CHUNK_SIZE=1000 `
  -e CHUNK_OVERLAP=100 `
  -e SOURCE_PATH=/data/source `
  -v "E:/1C_Export/Files:/data/source:ro" `
  -v "E:/bases/mcp_cloud:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```

### OpenAI (минимальная настройка)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_cloud_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_PROVIDER=openai `
  -e OPENAI_API_KEY=YOUR_OPENAI_KEY `
  -e SOURCE_PATH=/data/source `
  -v "E:/1C_Export/Files:/data/source:ro" `
  -v "E:/bases/mcp_cloud:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```

### Локальная модель через LM Studio

```powershell
docker run -d -p 8000:8000 `
  --name 1c_cloud_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_PROVIDER=openai `
  -e OPENAI_API_BASE=http://host.docker.internal:1234 `
  -e OPENAI_API_KEY=lm-studio `
  -e SOURCE_PATH=/data/source `
  -v "E:/1C_Export/Files:/data/source:ro" `
  -v "E:/bases/mcp_cloud:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-cloud-mcp": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_cloud_service_001"
    }
  }
}
```
