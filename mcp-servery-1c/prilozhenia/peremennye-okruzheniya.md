# Переменные окружения

Сводная таблица переменных окружения для всех MCP-серверов.

## Общие переменные

Эти переменные используются большинством серверов:

| Переменная | Описание | Обязательная | По умолчанию |
|------------|----------|--------------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да | — |
| `RESET_DATABASE` | Переиндексировать данные | Нет | `true` / `false` (зависит от сервера) |
| `RESET_CACHE` | Перезагрузить модель | Нет | `true` |
| `USESSE` | SSE транспорт (для legacy клиентов) | Нет | `false` |

## Embedding модели (LM Studio / Ollama / OpenRouter)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API. Суффикс `/v1` добавляется автоматически | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API | `lm-studio` |
| `OPENAI_MODEL` | Модель embedding | `Qwen3-Embedding-4B` |
| `EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов (для моделей с переменной размерностью) | *(авто)* |

## Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face | `intfloat/multilingual-e5-base` |

{% hint style="info" %}
Если указан `OPENAI_API_KEY`, используется внешнее API. Иначе — встроенная CPU модель.
{% endhint %}

## Настройки индексации (новые параметры)

Эти переменные управляют процессом индексации и доступны в серверах, где указано:

| Переменная | Описание | По умолчанию | Серверы |
|------------|----------|--------------|---------|
| `INDEX_BATCH_SIZE` | Размер пакета при добавлении в ChromaDB | `25`–`50` | CodeMetadata, Graph |
| `CHUNK_SIZE` | Размер фрагмента текста при разбивке (символы) | `1000` | CodeMetadata, Cloud |
| `CHUNK_OVERLAP` | Перекрытие фрагментов (символы) | `100` | Cloud |
| `MAX_TOKENS_PER_BATCH` | Максимум токенов в одном пакете API | `7500` | Graph |
| `EMBEDDING_MAX_TOKENS` | Максимум токенов на текст для эмбеддингов | *(авто)* | Graph |
| `EMBEDDING_CONCURRENCY` | Параллельные запросы к API | `1` | Cloud |
| `EMBEDDING_BATCH_SIZE` | Текстов в одном API-запросе | `10` | Cloud |
| `INDEX_CODE` | Индексация BSL кода | `true` | CodeMetadata |
| `INDEX_METADATA` | Индексация метаданных | `true` | CodeMetadata |
| `INDEX_HELP` | Индексация HTML-справки | `true` | CodeMetadata |

## Переменные по серверам

### HelpSearchServer (порт 8003)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `1C_BIN_PATH` | Путь к bin в контейнере | `/1c_docs` |
| `RESET_DATABASE` | Переиндексировать | `true` |
| `RESET_CACHE` | Перезагрузить модель | `true` |

### CodeMetadataSearchServer (порт 8000)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `METADATA_PATH` | Путь к метаданным | `/app/metadata` |
| `CODE_PATH` | Путь к коду | `/app/code` |
| `RESET_DATABASE` | Переиндексировать | `false` |
| `INDEX_CODE` | Индексация BSL кода | `true` |
| `INDEX_METADATA` | Индексация метаданных | `true` |
| `INDEX_HELP` | Индексация HTML-справки | `true` |
| `INDEX_BATCH_SIZE` | Размер пакета индексации | `25` |
| `CHUNK_SIZE` | Размер фрагмента текста | `1000` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |

### CloudEmbeddingsServer (порт 8000)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `EMBEDDING_PROVIDER` | Провайдер: `openrouter`, `openai`, `cohere`, `jina`, `local` | Обязательно |
| `SOURCE_PATH` | Путь к исходным файлам | `/data/source` |
| `CHROMA_PATH` | Путь к ChromaDB | `/data/chroma_db` |
| `EMBEDDING_CONCURRENCY` | Параллельные запросы | `1` |
| `EMBEDDING_BATCH_SIZE` | Текстов в запросе | `10` |
| `CHUNK_SIZE` | Размер фрагмента | `1000` |
| `CHUNK_OVERLAP` | Перекрытие фрагментов | `100` |
| `AUTO_INDEX` | Автоиндексация при запуске | `true` |

### SSLSearchServer (порт 8008)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `SSL_VERSION` | Версия БСП | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `false` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `EMBEDDING_INPUT_TYPE_ENABLED` | Различение query/document для эмбеддингов | `true` |
| `FORCE_REINDEX_ON_DIMENSION_MISMATCH` | Автопересоздание при несовпадении размерности | `true` |

### Graph Metadata Search (порт 8006)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `NEO4J_URI` | URI Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь | `neo4j` |
| `NEO4J_PASSWORD` | Пароль | Обязательно |
| `METADATA_DIRECTORY` | Путь к метаданным | `/app/metadata` |
| `PROJECT_NAME` | Название проекта | `1C Metadata Project` |
| `INDEX_BATCH_SIZE` | Размер пакета индексации | `50` |
| `MAX_TOKENS_PER_BATCH` | Макс. токенов на пакет API | `7500` |
| `OPENAI_EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `ENABLE_CODE_SEARCH` | Поиск по BSL-коду | `true` |
| `ENABLE_BUSINESS_SEARCH` | Семантический поиск по бизнес-описаниям | `true` |

### 1CCodeChecker (порт 8007)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `ONEC_AI_TOKEN` | Токен 1С:Напарник | Обязательно |
| `MCP_TOOL_CALL_MODE` | Режим вызова upstream: `direct` (прямые вызовы) или `standard` (промпты) | `direct` |
| `ONEC_AI_BASE_URL` | Базовый URL API 1С.ai | `https://code.1c.ai` |
| `ONEC_AI_TIMEOUT` | Таймаут запросов к API (секунды) | `30` |
| `ONEC_AI_SKILL_NAME` | Режим сессии: `custom` (с инструментами) или `raw` | `custom` |
| `ONEC_AI_INPUT_MAX_LENGTH` | Максимальная длина входных данных | `100000` |
| `ONEC_AI_UI_LANGUAGE` | Язык интерфейса | `russian` |
| `ONEC_AI_PROGRAMMING_LANGUAGE` | Язык программирования | *(пусто)* |
| `ONEC_AI_SCRIPT_LANGUAGE` | Скриптовый язык (`ru` / `en`) | `ru` |
| `ONEC_CONFIG_NAME` | Конфигурация 1С по умолчанию для `config_help` | *(пусто)* |
| `MAX_ACTIVE_SESSIONS` | Макс. активных сессий | `10` |
| `SESSION_TTL` | Время жизни сессии (секунды) | `3600` |
| `HTTP_PORT` | Порт HTTP-сервера | `8007` |

### SyntaxCheckServer (порт 8002)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `USESSE` | SSE транспорт | `false` |

### FormsServer (порт 8011)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `PORT` | Порт сервера | `8011` |

### TemplatesSearchServer (порт 8004)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `true` |
| `HTTP_PORT` | Порт HTTP-сервера | `8004` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |

## Примеры

### Минимальный набор (CPU)

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY
```

### С LM Studio

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
-e OPENAI_API_KEY=lm-studio `
-e OPENAI_MODEL=Qwen3-Embedding-4B
```

### С OpenRouter

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e OPENAI_API_BASE=https://openrouter.ai/api `
-e OPENAI_API_KEY=YOUR_OPENROUTER_KEY `
-e OPENAI_MODEL=qwen/qwen3-embedding-8b
```

### С Ollama

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e OPENAI_API_BASE=http://host.docker.internal:11434/v1 `
-e OPENAI_API_KEY=ollama `
-e OPENAI_MODEL=qwen3:embedding-4b
```
