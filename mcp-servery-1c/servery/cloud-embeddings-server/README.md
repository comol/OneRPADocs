# CloudEmbeddingsServer

MCP-сервер для поиска по коду, метаданным и справке 1С с параллельным построением embedding через OpenAI-совместимый API.

## Назначение

CloudEmbeddingsServer индексирует каталог выгрузки 1С и предоставляет базовые инструменты поиска: по метаданным, BSL-коду и документации. Сервер рассчитан на внешний embedding-провайдер и параллельную обработку запросов к нему.

{% hint style="warning" %}
По умолчанию сервер слушает порт контейнера `8000`, как и CodeMetadataSearchServer. При совместном запуске публикуйте его на свободном порту хоста, например `-p 8001:8000`. В `docker-compose.yml` внешний порт задаётся подстановкой `MCP_PORT`, а внутри приложения порт настраивается переменной `PORT`.
{% endhint %}

## Доступные инструменты MCP

| Инструмент | Описание |
|------------|----------|
| `metadatasearch` | Поиск объектов метаданных 1С |
| `codesearch` | Поиск по BSL-коду |
| `helpsearch` | Поиск по документации |
| `reindex` | Повторная индексация каталога |
| `stats` | Статистика индексатора и коллекций |

### Параметры инструментов

| Инструмент | Параметры |
|------------|-----------|
| `metadatasearch` | `query` (string, обязательно), `limit` (int, по умолчанию `10`) |
| `codesearch` | `query` (string, обязательно), `limit` (int, по умолчанию `10`) |
| `helpsearch` | `query` (string, обязательно), `limit` (int, по умолчанию `10`) |
| `reindex` | `force` (bool, по умолчанию `false`) |
| `stats` | Без параметров |

## Переменные окружения

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `USESSE` | Включить SSE-транспорт. При `false` используется `streamable-http` | `false` |
| `EMBEDDING_PROVIDER` | Провайдер embedding | `openai` |
| `OPENAI_API_KEY` | Ключ OpenAI-совместимого API | Обязательно для cloud-режима |
| `OPENROUTER_API_KEY` | Ключ OpenRouter при `EMBEDDING_PROVIDER=openrouter` | — |
| `COHERE_API_KEY` | Ключ Cohere при `EMBEDDING_PROVIDER=cohere` | — |
| `JINA_API_KEY` | Ключ Jina при `EMBEDDING_PROVIDER=jina` | — |
| `OPENAI_API_BASE` | Необязательный URL OpenAI-совместимого API | — |
| `EMBEDDING_MODEL` | Явное имя модели провайдера | *(по умолчанию провайдера)* |
| `SOURCE_PATH` | Каталог исходных данных для индексации | `/data/source` |
| `CHROMA_PATH` | Каталог векторной БД | `/data/chroma_db` |
| `HOST` | Адрес, на котором слушает приложение | `0.0.0.0` |
| `PORT` | Внутренний порт приложения | `8000` |
| `AUTO_INDEX` | Индексировать каталог при запуске | `true` |
| `CHUNK_SIZE` | Размер чанка | `1000` |
| `CHUNK_OVERLAP` | Перекрытие чанков | `100` |
| `MAX_BATCH_SIZE` | Максимальный размер пакета индексации | `100` |
| `DEFAULT_SEARCH_LIMIT` | Количество результатов поиска по умолчанию | `10` |
| `EMBEDDING_CONCURRENCY` | Количество параллельных embedding-запросов | `1` |
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

## Доработка

Система плагинов в этом сервере не поставляется. Поведение задаётся окружением: провайдер и модель эмбеддингов, каталог исходных данных (`SOURCE_PATH`, `AUTO_INDEX`), нарезка (`CHUNK_SIZE`, `CHUNK_OVERLAP`), параллелизм индексации (`EMBEDDING_CONCURRENCY`, `EMBEDDING_BATCH_SIZE`) и размер выдачи по умолчанию.

Подробнее: [Серверы без плагинов](../../sistema-pluginov/dorabotka-bez-pluginov.md).
