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
| `USESSE` | SSE транспорт (для legacy клиентов) | `false` |
| `CHROMA_DB_PATH` | Путь к директории ChromaDB | `/app/chroma_db` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `INDEX_BATCH_SIZE` | Размер пакета при добавлении записей в ChromaDB. Увеличение ускоряет индексацию, но требует больше памяти | `25` |
| `REINDEX_INTERVAL_HOURS` | Интервал автоматической переиндексации в часах. `0` — отключить | `24` |

### Настройки разбиения текста (чанкинг)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `CHUNK_SIZE` | Максимальный размер фрагмента текста. Больше — меньше фрагментов, но менее точный поиск | `1000` |
| `CHUNK_SIZE_UNIT` | Единица измерения: `chars` (символы) или `tokens` (токены через tiktoken) | `chars` |
| `CHUNK_OVERLAP` | Перекрытие чанков при дроблении крупных процедур/объектов. Не применяется к атомарным чанкам | `100` |
| `CHUNK_OVERLAP_CODE` | Перекрытие чанков для BSL-кода. Приоритет над `CHUNK_OVERLAP` | `100` |
| `CHUNK_OVERLAP_TEXT` | Перекрытие чанков для метаданных, XML и справки. Приоритет над `CHUNK_OVERLAP` | `200` |

### Расширение контекста результатов поиска

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `CONTEXT_EXPANSION` | Режим расширения контекста: `none` — без расширения, `siblings` — все суб-фрагменты той же процедуры/функции (для кода), `window` — соседние фрагменты из того же файла (для метаданных/справки) | `none` |
| `CONTEXT_WINDOW_SIZE` | Количество соседних фрагментов с каждой стороны для режима `window`. `1` = 3 фрагмента итого, `2` = 5 фрагментов | `1` |

### Качество поиска

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `BM25_ALPHA` | Вес семантического поиска при гибридном ранжировании (0–1). Остаток `1 - alpha` уходит на BM25. Меньше — больший вес точным совпадениям, больше — больший вес семантической близости | `0.5` |
| `OVERFETCH_MULTIPLIER` | Множитель расширения выборки перед BM25-ранжированием. Больше — шире пул кандидатов, но медленнее | `2` |
| `MIN_SCORE_THRESHOLD` | Минимальный порог combined_score для включения результата (0–1). `0` — без отсечения, `0.3` — агрессивное | `0.15` |
| `EMBEDDING_CACHE_SIZE` | Размер LRU-кэша для эмбеддингов поисковых запросов. Сокращает вызовы API при повторных запросах. `0` — отключить | `256` |

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

### Отладка

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LOG_LEVEL` | Уровень логирования: `debug` — все сообщения, `normal` — основные, `none` — отключено | `normal` |

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

### Облегчённый образ (light) с LM Studio

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
  -e CONTEXT_EXPANSION=siblings `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
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
  -e INDEX_FORMS_SEMANTIC=true `
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
