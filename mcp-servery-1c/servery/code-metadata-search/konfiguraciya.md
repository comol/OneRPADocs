# Конфигурация CodeMetadataSearchServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `METADATA_PATH` | Путь к отчету в контейнере | `/app/metadata` |
| `CODE_PATH` | Путь к коду в контейнере | `/app/code` |

### Настройки сервера

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_HOST` | Хост для привязки сервера | `0.0.0.0` |
| `MCP_PORT` | Порт сервера | `8000` |
| `MCP_PATH` | Путь MCP-эндпоинта | `/mcp` |
| `USESSE` | Включить SSE-транспорт для legacy-клиентов. По умолчанию используется `streamable-http` | `false` |
| `VECTOR_DB_PATH` | Путь к общей директории векторного хранилища и служебных индексов | `/app/chroma_db` |
| `CHROMA_DB_PATH` | Совместимый старый алиас для `VECTOR_DB_PATH`; внутри каталога теперь используются zvec-коллекции и SQLite-индексы | `/app/chroma_db` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `BACKGROUND_INDEXING` | Выполнять стартовую индексацию в фоне, не блокируя запуск MCP | `true` |
| `INCREMENTAL_INDEXING` | Сверять SHA-256 файлов и обновлять только изменившиеся данные | `true` |
| `INCREMENTAL_HASH_CHUNK_BYTES` | Размер блока чтения при расчёте SHA-256 | `65536` |
| `REINDEX_INTERVAL_SEC` | Интервал периодической инкрементальной индексации в секундах. `0` — отключить | `7200` |
| `REINDEX_INTERVAL_HOURS` | Пользовательский алиас интервала в часах; явный `REINDEX_INTERVAL_SEC` имеет приоритет | *(не задано)* |
| `PARSE_WORKERS` | Число параллельных обработчиков файлов | от 1 до 8, по числу CPU |
| `EMBEDDING_CONCURRENCY` | Максимум параллельных запросов эмбеддингов | `6` |
| `SUB_INDEX_WORKERS` | Число параллельно строящихся вспомогательных индексов | `4` |
| `SUB_INDEX_BULK_LOAD` | Загружать вспомогательные индексы пакетно | `true` |
| `SHUTDOWN_GRACE_SEC` | Время ожидания активных HTTP-запросов при остановке | `5` |
| `INDEX_METADATA` | Индексировать метаданные | `true` |
| `INDEX_CODE` | Индексировать BSL-код | `true` |
| `INDEX_HELP` | Индексировать HTML-справку | `true` |
| `INDEX_PHASE_ORDER` | Порядок фаз; метаданные всегда выполняются первыми | `metadata,code,help` |

### Качество поиска

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `BM25_ALPHA` | Вес семантического поиска при гибридном ранжировании (0–1). Остаток `1 - alpha` уходит на BM25. Меньше — больший вес точным совпадениям, больше — больший вес семантической близости | `0.5` |
| `OVERFETCH_MULTIPLIER` | Множитель расширения выборки для запросов по пути или идентификатору | `4` |
| `SEMANTIC_OVERFETCH_MULTIPLIER` | Множитель расширения выборки для семантических запросов | `6` |
| `OVERFETCH_HARD_CAP` | Жёсткий предел пула кандидатов до ранжирования | `200` |
| `OVERFETCH_COLLECTION_FRACTION` | Максимальная доля коллекции в пуле кандидатов | `0.1` |
| `MAX_CHUNKS_PER_SOURCE` | Максимум фрагментов одного источника в итоговой выдаче | `2` |
| `RRF_K` | Константа reciprocal-rank fusion | `20` |
| `FUSION_PRIOR_WEIGHT` | Вес априорной релевантности при объединении результатов | `0.2` |
| `FALLBACK_ON_EMPTY` | Выполнять резервный поиск, если основной маршрут ничего не вернул | `true` |
| `MIN_SCORE_THRESHOLD` | Минимальный порог combined_score для включения результата (0–1). `0` — без отсечения, `0.3` — агрессивное | `0.15` |
| `EMBEDDING_CACHE_SIZE` | Размер LRU-кэша для эмбеддингов поисковых запросов. Сокращает вызовы API при повторных запросах. `0` — отключить | `256` |

### Векторное хранилище zvec

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `VECTOR_PROFILE` | Профиль zvec: `fast_index`, `balanced`, `memory_saver` или `quality` | `fast_index` |
| `VECTOR_INDEX_TYPE` | Тип индекса: `hnsw`, `ivf` или `flat` | из профиля |
| `VECTOR_QUANTIZATION` | Квантизация: `none`, `fp16` или `int8` | из профиля |
| `VECTOR_STORAGE_MODE` | Режим хранения: `memory` или `mmap` | из профиля |
| `VECTOR_HNSW_M` | Параметр связности HNSW | из профиля |
| `VECTOR_HNSW_EF_CONSTRUCTION` | Глубина построения HNSW | из профиля |
| `VECTOR_HNSW_EF_SEARCH` | Явная глубина поиска HNSW | *(авто)* |
| `VECTOR_IVF_NLIST` | Число кластеров IVF | из профиля |
| `VECTOR_IVF_NPROBE` | Число просматриваемых кластеров IVF | из профиля |
| `VECTOR_MEMORY_LIMIT_MB` | Ограничение памяти zvec в МБ | *(не задано)* |
| `VECTOR_QUERY_THREADS` | Число потоков поиска | *(авто)* |
| `VECTOR_OPTIMIZE_THREADS` | Число потоков оптимизации | *(авто)* |

### Нейронный реранкер (cross-encoder)

Опциональный нейронный реранкер для повышения качества семантического поиска. При включении оценивает пары (запрос, документ) совместно, что значительно улучшает релевантность результатов.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ENABLE_RERANKER` | Включить нейронный реранкер. `true` — использовать cross-encoder, `false` — использовать BM25 | `false` |
| `RERANKER_MODEL` | Модель реранкера. Если не указано — автовыбор в зависимости от окружения | *(авто)* |
| `RERANKER_TOP_K` | Максимальное количество кандидатов, передаваемых реранкеру | `20` |

{% hint style="info" %}
**Автовыбор модели реранкера:**
- Если `OPENAI_API_BASE` задан → API-режим, модель по умолчанию: `Qwen/Qwen3-Reranker-8B`
- Если `OPENAI_API_BASE` не задан → локальный режим, модель: `Qwen/Qwen3-Reranker-0.6B`

Цепочка деградации: cross-encoder → BM25 → только вектор
{% endhint %}

### Embedding модели (LM Studio / Ollama)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_API_BASE` | URL API сервера. При использовании OpenRouter: `https://openrouter.ai/api`. Суффикс `/v1` добавляется автоматически | `http://host.docker.internal:1234/v1` |
| `EMBEDDING_API_KEY` | Ключ API (для LM Studio — любой, для OpenRouter — ваш ключ) | `lm-studio` |
| `EMBEDDING_MODEL` | Название модели для API или локального режима (для OpenRouter: `qwen/qwen3-embedding-8b`) | `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` |
| `EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов. Поддерживается моделями с переменной размерностью (Qwen3, text-embedding-3). Если не указано — определяется автоматически | *(авто)* |

{% hint style="info" %}
**Поддержка OpenRouter:** Сервер автоматически определяет OpenRouter по URL и добавляет необходимые HTTP-заголовки (`HTTP-Referer`, `X-Title`).
{% endhint %}

### Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face; та же переменная выбирает модель и в API-режиме | `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` |

Старые имена `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` пока принимаются как совместимые алиасы, но новые конфигурации следует создавать с `EMBEDDING_*`.

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
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
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
  -e EMBEDDING_API_BASE=https://openrouter.ai/api `
  -e EMBEDDING_API_KEY=YOUR_OPENROUTER_KEY `
  -e EMBEDDING_MODEL=qwen/qwen3-embedding-8b `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```

### Облегчённый образ (light) с LM Studio

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:light
```

### С реранкером и тюнингом поиска

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e ENABLE_RERANKER=true `
  -e BM25_ALPHA=0.5 `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
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
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
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
