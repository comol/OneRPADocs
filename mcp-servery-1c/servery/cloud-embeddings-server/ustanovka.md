# Установка CloudEmbeddingsServer

## Предварительные требования

1. Docker Desktop запущен
2. API-ключ выбранного провайдера (OpenRouter, OpenAI, Cohere или Jina)
3. [Подготовлены данные](../code-metadata-search/podgotovka-dannyh.md) из Конфигуратора

## Создание папок

```powershell
New-Item -ItemType Directory -Force -Path "E:\bases\mcp_cloud"
```

## Команды запуска

### С OpenRouter (рекомендуется)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_cloud_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_PROVIDER=openrouter `
  -e OPENROUTER_API_KEY=YOUR_OPENROUTER_KEY `
  -e EMBEDDING_CONCURRENCY=10 `
  -e EMBEDDING_BATCH_SIZE=10 `
  -e SOURCE_PATH=/data/source `
  -v "E:/1C_Export/Files:/data/source:ro" `
  -v "E:/bases/mcp_cloud:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```

### С OpenAI

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

### С Cohere

```powershell
docker run -d -p 8000:8000 `
  --name 1c_cloud_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_PROVIDER=cohere `
  -e COHERE_API_KEY=YOUR_COHERE_KEY `
  -e SOURCE_PATH=/data/source `
  -v "E:/1C_Export/Files:/data/source:ro" `
  -v "E:/bases/mcp_cloud:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```

### С локальной моделью (без API)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_cloud_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_PROVIDER=local `
  -e SOURCE_PATH=/data/source `
  -v "E:/1C_Export/Files:/data/source:ro" `
  -v "E:/bases/mcp_cloud:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```

## Docker Compose (рекомендуется)

Создайте файл `docker-compose.yml`:

```yaml
services:
  1c-cloud-mcp:
    image: comol/1c_cloud_mcp:latest
    container_name: 1c_cloud_mcp
    restart: unless-stopped
    ports:
      - "${MCP_PORT:-8000}:8000"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - EMBEDDING_PROVIDER=${EMBEDDING_PROVIDER:-openrouter}
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY:-}
      - OPENAI_API_KEY=${OPENAI_API_KEY:-}
      - COHERE_API_KEY=${COHERE_API_KEY:-}
      - JINA_API_KEY=${JINA_API_KEY:-}
      - EMBEDDING_MODEL=${EMBEDDING_MODEL:-}
      - EMBEDDING_CONCURRENCY=${EMBEDDING_CONCURRENCY:-1}
      - EMBEDDING_BATCH_SIZE=${EMBEDDING_BATCH_SIZE:-10}
      - CHUNK_SIZE=${CHUNK_SIZE:-1000}
      - CHUNK_OVERLAP=${CHUNK_OVERLAP:-100}
      - AUTO_INDEX=${AUTO_INDEX:-true}
      - SOURCE_PATH=/data/source
      - CHROMA_PATH=/data/chroma_db
    volumes:
      - ${SOURCE_PATH}:/data/source:ro
      - ${CHROMA_PATH:-./chroma_db}:/data/chroma_db
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 120s
```

Создайте файл `.env`:

```env
LICENSE_KEY=YOUR_LICENSE_KEY
EMBEDDING_PROVIDER=openrouter
OPENROUTER_API_KEY=YOUR_OPENROUTER_KEY
EMBEDDING_CONCURRENCY=10
EMBEDDING_BATCH_SIZE=10
SOURCE_PATH=E:/1C_Export/Files
CHROMA_PATH=E:/bases/mcp_cloud
```

Запуск:

```powershell
docker compose up -d
docker compose logs -f
```

## Первый запуск

При первом запуске:

1. Скачивается образ (~500 МБ)
2. Парсятся файлы конфигурации (метаданные, код, справка)
3. Текст разбивается на фрагменты (chunk)
4. Фрагменты отправляются в облачный API для генерации эмбеддингов
5. Эмбеддинги сохраняются в локальную ChromaDB

### Ожидаемое время индексации

| Параллелизм | Время | Запросов/мин |
|-------------|-------|--------------|
| `EMBEDDING_CONCURRENCY=1` | ~8 часов | ~6 |
| `EMBEDDING_CONCURRENCY=5` | ~2–3 часа | ~120 |
| `EMBEDDING_CONCURRENCY=10` | ~1–2 часа | ~240 |

{% hint style="warning" %}
Высокие значения `EMBEDDING_CONCURRENCY` могут вызвать ошибки rate-limit у провайдера. Рекомендуемое значение: `10`.
{% endhint %}

### Мониторинг

```powershell
docker logs -f 1c_cloud_mcp
```

### Проверка состояния

```powershell
curl http://localhost:8000/health
```

## Проверка работы

```powershell
docker ps --filter name=1c_cloud_mcp
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

## Переиндексация

### Инкрементальная (только изменённые файлы)

Через MCP инструмент `reindex` или HTTP:

```powershell
curl -X POST http://localhost:8000/reindex
```

### Полная переиндексация

```powershell
docker stop 1c_cloud_mcp
docker rm 1c_cloud_mcp
# Запустить заново с тем же docker run или docker compose up -d
```

Или через MCP инструмент: `reindex(force=True)`

## Мультипроектная конфигурация

Для нескольких конфигураций 1С запускайте контейнеры на разных портах:

```powershell
# Проект 1
docker run -d -p 8000:8000 --name 1c_cloud_project1 `
  -e LICENSE_KEY=YOUR_KEY -e EMBEDDING_PROVIDER=openrouter `
  -e OPENROUTER_API_KEY=YOUR_KEY `
  -v "E:/Project1/Export:/data/source:ro" `
  -v "E:/bases/mcp_cloud_p1:/data/chroma_db" `
  comol/1c_cloud_mcp:latest

# Проект 2
docker run -d -p 8001:8000 --name 1c_cloud_project2 `
  -e LICENSE_KEY=YOUR_KEY -e EMBEDDING_PROVIDER=openrouter `
  -e OPENROUTER_API_KEY=YOUR_KEY `
  -v "E:/Project2/Export:/data/source:ro" `
  -v "E:/bases/mcp_cloud_p2:/data/chroma_db" `
  comol/1c_cloud_mcp:latest
```
