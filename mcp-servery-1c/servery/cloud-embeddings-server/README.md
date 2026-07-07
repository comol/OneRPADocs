# CloudEmbeddingsServer

MCP-сервер для поиска по коду, метаданным и справке 1С с параллельным построением embedding через OpenAI-совместимый API.

## Назначение

CloudEmbeddingsServer индексирует каталог выгрузки 1С и предоставляет базовые инструменты поиска: по метаданным, BSL-коду и документации. Сервер рассчитан на внешний embedding-провайдер и параллельную обработку запросов к нему.

{% hint style="warning" %}
По умолчанию сервер использует порт `8000`, как и CodeMetadataSearchServer. При совместном запуске задайте отдельный `MCP_PORT`, например `8001`.
{% endhint %}

## Доступные инструменты MCP

| Инструмент | Описание |
|------------|----------|
| `metadatasearch` | Поиск объектов метаданных 1С |
| `codesearch` | Поиск по BSL-коду |
| `helpsearch` | Поиск по документации |
| `reindex` | Повторная индексация каталога |
| `stats` | Статистика индексатора и коллекций |

## Переменные окружения

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `USESSE` | Включить SSE-транспорт. При `false` используется `streamable-http` | `false` |
| `EMBEDDING_PROVIDER` | Провайдер embedding | `openai` |
| `OPENAI_API_KEY` | Ключ OpenAI-совместимого API | Обязательно для cloud-режима |
| `SOURCE_PATH` | Каталог исходных данных для индексации | `./src` |
| `CHROMA_PATH` | Каталог векторной БД | `./chroma_db` |
| `MCP_PORT` | Порт MCP-сервера | `8000` |
| `AUTO_INDEX` | Индексировать каталог при запуске | `true` |
| `CHUNK_SIZE` | Размер чанка | `1000` |
| `CHUNK_OVERLAP` | Перекрытие чанков | `100` |
| `MAX_BATCH_SIZE` | Максимальный размер пакета индексации | `100` |
| `DEFAULT_SEARCH_LIMIT` | Количество результатов поиска по умолчанию | `10` |
| `EMBEDDING_CONCURRENCY` | Количество параллельных embedding-запросов | `10` |
| `EMBEDDING_BATCH_SIZE` | Размер пакета embedding-запроса | `10` |

## Endpoint

MCP endpoint: `/mcp`.

Дополнительные HTTP endpoints:

- `/health` — состояние сервера и коллекций
- `/reindex` — запуск переиндексации через HTTP POST

## Быстрый старт

```powershell
docker run -d -p 8001:8000 `
  --name 1c_cloud_embeddings_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e OPENAI_API_KEY=YOUR_OPENAI_API_KEY `
  -e SOURCE_PATH=/app/src `
  -v "E:/1C_Export:/app/src" `
  -v "E:/bases/1c_cloud_embeddings:/app/chroma_db" `
  comol/1c-cloud-mcp:latest
```

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-cloud-embeddings-mcp": {
      "url": "http://localhost:8001/mcp",
      "connection_id": "1c_cloud_embeddings_001"
    }
  }
}
```
