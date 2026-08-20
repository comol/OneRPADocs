# Переменные окружения

Сводная таблица переменных окружения для всех MCP-серверов.

## Общие переменные

Эти переменные используются большинством серверов:

| Переменная | Описание | Обязательная | По умолчанию |
|------------|----------|--------------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да | — |
| `RESET_DATABASE` | Переиндексировать данные | Нет | `true` / `false` (зависит от сервера) |
| `RESET_CACHE` | Перезагрузить модель | Нет | `true` |
| `USESSE` | Включить SSE-транспорт для legacy-клиентов. При `false` используется `streamable-http` | Нет | `false` |

## Embedding модели (LM Studio / Ollama / OpenRouter)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_API_BASE` | URL API. Суффикс `/v1` добавляется автоматически | `http://host.docker.internal:1234/v1` |
| `EMBEDDING_API_KEY` | Ключ API | `lm-studio` |
| `EMBEDDING_MODEL` | Модель embedding для API или локального режима | `Qwen3-Embedding-4B` |
| `EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов (для моделей с переменной размерностью) | *(авто)* |

## Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face | `intfloat/multilingual-e5-base` |

{% hint style="info" %}
Если задан `EMBEDDING_API_BASE` или используется light-образ, сервер обращается к внешнему API. Старые `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` поддерживаются CodeMetadataSearchServer, SSLSearchServer и TemplatesSearchServer как совместимые алиасы. CloudEmbeddingsServer по-прежнему использует provider-specific ключи (`OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `COHERE_API_KEY`, `JINA_API_KEY`).
{% endhint %}

## Настройки индексации

Эти переменные управляют процессом индексации и доступны в серверах, где указано:

| Переменная | Описание | По умолчанию | Серверы |
|------------|----------|--------------|---------|
| `INDEX_BATCH_SIZE` | Размер пакета при добавлении в векторное хранилище | `512` | Graph |
| `MAX_TOKENS_PER_BATCH` | Максимум токенов в одном пакете API | `28000` | Graph |
| `EMBEDDING_MAX_TOKENS` | Максимум токенов на текст для эмбеддингов | *(авто)* | Graph |
| `REINDEX_INTERVAL_SEC` | Интервал автоматической инкрементальной индексации (секунды) | `7200` | CodeMetadata |
| `REINDEX_INTERVAL_HOURS` | Алиас интервала в часах; `REINDEX_INTERVAL_SEC` имеет приоритет | *(не задано)* | CodeMetadata |
| `ENABLE_RERANKER` | Нейронный реранкер (cross-encoder) | `false` | CodeMetadata |
| `RERANKER_MODEL` | Модель реранкера | *(авто)* | CodeMetadata |
| `RERANKER_TOP_K` | Макс. кандидатов для реранкера | `20` | CodeMetadata |

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
| `MCP_HOST` | Хост для привязки сервера | `0.0.0.0` |
| `MCP_PORT` | Порт сервера | `8000` |
| `MCP_PATH` | Путь MCP-эндпоинта | `/mcp` |
| `FASTMCP_STATELESS_HTTP` | Stateless-режим HTTP | `true` |
| `MCP_SESSION_IDLE_TTL_SEC` | Таймаут простоя MCP-сессии | `1800` |
| `MCP_SESSION_MAX_LIFETIME_SEC` | Максимальное время жизни MCP-сессии | `86400` |
| `MCP_SESSION_MAX_CONCURRENT` | Максимум одновременных MCP-сессий | `64` |
| `MCP_SESSION_CLEANUP_INTERVAL_SEC` | Интервал очистки сессий | `60` |
| `MCP_SESSION_BOUNDS_MODE` | Применять лимиты (`enforce`) или только сообщать (`report`) | `enforce` |
| `MCP_IMAGE_REF` | Неизменяемая ссылка на образ с digest для release identity | *(не задано)* |
| `VECTOR_DB_PATH` | Путь к директории векторного хранилища zvec | `/app/chroma_db` |
| `CHROMA_DB_PATH` | Устаревший совместимый алиас `VECTOR_DB_PATH` | `/app/chroma_db` |
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Модель API или локальная модель | `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` |
| `RESET_DATABASE` | Переиндексировать | `false` |
| `BACKGROUND_INDEXING` | Индексировать в фоне, не блокируя запуск MCP | `true` |
| `INCREMENTAL_INDEXING` | Обновлять только изменившиеся файлы по SHA-256 | `true` |
| `INDEX_NESTED_CONFIGURATIONS` | Индексировать вложенные конфигурации поставщика отдельными источниками | `false` |
| `NESTED_CONFIGURATION_PATHS` | Каталоги-контейнеры вложенных конфигураций через запятую | `Ext/ParentConfigurations` |
| `REINDEX_INTERVAL_SEC` | Интервал автоматической индексации (секунды) | `7200` |
| `REINDEX_INTERVAL_HOURS` | Алиас интервала в часах | *(не задано)* |
| `BM25_ALPHA` | Вес семантического поиска (0–1) | `0.5` |
| `OVERFETCH_MULTIPLIER` | Множитель выборки для запросов по пути/идентификатору | `4` |
| `SEMANTIC_OVERFETCH_MULTIPLIER` | Множитель выборки для семантических запросов | `6` |
| `MIN_SCORE_THRESHOLD` | Минимальный порог оценки результата (0–1) | `0.15` |
| `EMBEDDING_CACHE_SIZE` | Размер LRU-кэша эмбеддингов запросов | `256` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `ENABLE_RERANKER` | Включить нейронный реранкер (cross-encoder) | `false` |
| `RERANKER_MODEL` | Модель реранкера | *(авто)* |
| `RERANKER_TOP_K` | Макс. кандидатов для реранкера | `20` |
| `EMBEDDING_PROVIDER` | Явный выбор `remote` или `local` | *(авто)* |
| `EMBEDDING_PROVIDER_AMBIGUITY` | Политика неоднозначной legacy-конфигурации | `error` |
| `EMBEDDING_MEMORY_BUDGET_MB` | Бюджет памяти локальной embedding-модели | `4096` |
| `EMBEDDING_MEMORY_BUDGET_MODE` | `refuse`, `warn` или `off` | `refuse` |
| `VECTOR_PROFILE` | Профиль zvec: `fast_index`, `balanced`, `memory_saver`, `quality` | `fast_index` |
| `VECTOR_OPTIMIZE_ENABLED` | Разрешить оптимизацию zvec | `true` |
| `VECTOR_OPTIMIZE_DEADLINE_SEC` | Таймаут фоновой оптимизации | `1800` |
| `VECTOR_OPTIMIZE_CANCEL_DEADLINE_SEC` | Таймаут оптимизации из запроса | `5` |
| `VECTOR_WRITE_FAILURE_THRESHOLD` | Ошибки записи до терминального состояния | `5` |

### CloudEmbeddingsServer (порт 8000 по умолчанию)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `USESSE` | Включить SSE-транспорт. При `false` используется `streamable-http` | `false` |
| `EMBEDDING_PROVIDER` | Провайдер embedding | `openai` |
| `OPENAI_API_KEY` | Ключ OpenAI-совместимого API | Обязательно для cloud-режима |
| `OPENROUTER_API_KEY` | Ключ OpenRouter | — |
| `COHERE_API_KEY` | Ключ Cohere | — |
| `JINA_API_KEY` | Ключ Jina | — |
| `OPENAI_API_BASE` | URL OpenAI-совместимого API | — |
| `EMBEDDING_MODEL` | Явное имя модели провайдера | *(по умолчанию провайдера)* |
| `SOURCE_PATH` | Каталог исходных данных для индексации | `/data/source` |
| `CHROMA_PATH` | Каталог векторной БД | `/data/chroma_db` |
| `HOST` | Адрес привязки приложения | `0.0.0.0` |
| `PORT` | Внутренний порт приложения | `8000` |
| `AUTO_INDEX` | Индексировать каталог при запуске | `true` |
| `CHUNK_SIZE` | Размер чанка | `1000` |
| `CHUNK_OVERLAP` | Перекрытие чанков | `100` |
| `MAX_BATCH_SIZE` | Максимальный размер пакета индексации | `100` |
| `DEFAULT_SEARCH_LIMIT` | Количество результатов поиска по умолчанию | `10` |
| `EMBEDDING_CONCURRENCY` | Количество параллельных embedding-запросов | `1` |
| `EMBEDDING_BATCH_SIZE` | Размер пакета embedding-запроса | `10` |

### SSLSearchServer (порт 8008)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `SSL_VERSION` | Версия БСП | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `false` |
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Модель API или локальная модель | `intfloat/multilingual-e5-small` |
| `INDEXING_THREADS` | Потоки индексации | `5` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `EMBEDDING_INPUT_TYPE_ENABLED` | Различение query/document для эмбеддингов | `true` |
| `FORCE_REINDEX_ON_DIMENSION_MISMATCH` | Автопересоздание при несовпадении размерности | `true` |
| `MIN_SCORE` | Порог cosine similarity для результатов `ssl_search` | `0.3826` |
| `LOG_QUERIES` | Записывать полный текст поисковых запросов вместо длины и хеша | `false` |
| `PLUGIN_DIR` | Каталог Python-плагинов | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Останавливать индексацию при ошибке derived-state hook | `false` |

### Graph Metadata Search (порт 8006)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `NEO4J_URI` | URI Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь | `neo4j` |
| `NEO4J_PASSWORD` | Пароль | Обязательно |
| `METADATA_DIRECTORY` | Путь к метаданным | `/app/metadata` |
| `NEO4J_DATABASE` | Имя базы Neo4j | `neo4j` |
| `PROJECT_NAME` | Название проекта | `1C Metadata Project` |
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `INDEX_BATCH_SIZE` | Размер пакета индексации | `512` |
| `MAX_TOKENS_PER_BATCH` | Макс. токенов на пакет API | `28000` |
| `EMBEDDING_REQUEST_CONCURRENCY` | Параллельные запросы к API эмбеддингов | `6` |
| `OPENAI_EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `EMBEDDING_API_BASE` | URL API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Модель API эмбеддингов | `text-embedding-ada-002` |
| `LOCAL_EMBEDDING_MODEL` | Резервная локальная CPU-модель | `intfloat/multilingual-e5-small` |
| `ENABLE_CODE_SEARCH` | Поиск по BSL-коду | `true` |
| `ENABLE_BUSINESS_SEARCH` | Семантический поиск по бизнес-описаниям | `true` |
| `CALCULATE_BUSINESS_INFO` | Генерировать AI бизнес-описания | `false` |
| `ENABLE_METADATA_DESCRIPTION_EMBEDDING` | Эмбеддинги для описательных полей | `true` |
| `MCP_HOST` | Хост MCP-сервера | `0.0.0.0` |
| `MCP_PORT` | Порт MCP | `8006` |
| `MCP_PATH` | URL-путь MCP эндпоинта | `/mcp` |
| `MCP_TOOL_PROFILE` | Профиль публикуемых инструментов: `admin` или `read-only` | `admin` |
| `MCP_NAMESPACE` | Namespace регистрации графовых проектов | `default` |
| `GRAPH_SCOPE_ENFORCED` | Требовать явный `project_id` в project-scoped вызовах | `false` |
| `GRAPH_SCOPE_MIGRATION_WINDOW` | Разрешить вызовы без `project_id` при единственном проекте | `false` |
| `GRAPH_MAX_ITEMS` | Жёсткий предел элементов в ответе | `200` |
| `GRAPH_TOOL_TIMEOUT_SECONDS` | Таймаут выполнения инструмента | `300` |
| `EMBEDDING_ALLOW_OFFLINE_FALLBACK` | Автопереход на локальную модель | `true` |
| `TEMPLATE_MODE_ENABLED` | Шаблонный режим (JSON-запросы без LLM) | `true` |
| `TEMPLATE_MODE_ONLY` | Только шаблоны, без LLM | `false` |
| `CODE_SEARCH_MAX_FILE_SIZE` | Макс. размер BSL-файла (байт) | `50000` |
| `CODE_EXPORT_PATH` | Путь к XML-выгрузке в файлы | — |
| `LOAD_BSL_SIGNATURES` | Загружать BSL-граф (Module/Routine/CALLS) | `true` |
| `ENABLE_ROUTINE_EMBEDDINGS` | Эмбеддинги для процедур/функций | `true` |
| `LOAD_FORMS_FROM_XML` | Загружать структуру управляемых форм из XML | `false` |
| `LOAD_ORDINARY_FORMS` | Загружать структуру обычных форм | `true` |
| `LOAD_EVENT_SUBSCRIPTIONS` | Загружать подписки на события | `false` |
| `LOAD_PREDEFINED_VALUES` | Загружать предопределённые элементы | `false` |
| `LOAD_ROLE_RIGHTS` | Загружать права ролей | `false` |
| `LOAD_HELP_FROM_HTML` | Загружать справку из HTML | `false` |
| `LOAD_DCS_TEMPLATES` | Загружать схемы компоновки данных (СКД) | `false` |
| `EXTENSION_NAME` | Имя расширения | — |
| `EXTENSION_BASE_PROJECT` | Имя базового проекта для расширения | — |
| `EXTENSION_APPLY_ORDER` | Порядок применения слоя расширения | `1` |
| `GRAPH_PLUGINS_ENABLED` | Загружать плагины Graph Metadata Search | `false` |
| `GRAPH_PLUGINS_DIRECTORY` | Каталог плагинов | `plugins` |
| `GRAPH_PLUGIN_STRICT_BUILD` | Останавливать построение поколения при ошибке derived-state hook | `false` |
| `GRAPH_PLUGIN_HOOK_TIMEOUT_SECONDS` | Бюджет времени одного plugin hook; `0` отключает контроль | `5.0` |

Полный список переменных Graph Metadata Search, включая лимиты графовых ответов, поколения и загрузку данных: [Конфигурация Graph Metadata Search](../servery/graph-metadata-search/konfiguraciya.md).

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
| `FILES_DIR` | Каталог с файлами BSL внутри контейнера. Если каталог задан и существует, сервер регистрирует инструмент `syntaxcheck_file` | *(пусто)* |
| `PLUGINS_DIR` | Каталог Python-плагинов; пустое значение использует `/app/plugins` | *(пусто)* |
| `LOG_LEVEL` | Уровень журналирования | `INFO` |
| `MCP_HTTP_PATH` | Endpoint `streamable-http` | `/mcp` |
| `MCP_SSE_PATH` | Endpoint потока событий при `USESSE=true` | `/sse` |
| `MCP_MESSAGE_PATH` | Endpoint сообщений при `USESSE=true` | `/messages/` |
| `BSL_ANALYZER_TIMEOUT_SECONDS` | Таймаут анализатора | `30` |
| `BSL_ANALYZER_STDOUT_LIMIT_BYTES` | Лимит JSONL-отчёта анализатора | `16777216` |
| `BSL_ANALYZER_STDERR_LIMIT_BYTES` | Лимит диагностического вывода анализатора | `4194304` |
| `BSL_ANALYZER_KILL_GRACE_SECONDS` | Ожидание перед принудительной остановкой | `2` |
| `BSL_SOURCE_ENCODING` | Явная кодировка файлов; если не задана, пробуются UTF-8 с BOM и CP1251 | *(пусто)* |

### TemplatesSearchServer (порт 8004)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `true` |
| `HTTP_PORT` | Порт HTTP-сервера | `8004` |
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Модель API или локальная модель | `intfloat/multilingual-e5-small` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `TEMPLATES_DB_PATH` | Путь к SQLite-базе шаблонов и заметок | `/app/chroma_db/templates.db` |
| `ZVEC_DB_PATH` | Каталог векторного индекса zvec | `/app/chroma_db/zvec_db` |
| `RECALL_RELEVANCE_THRESHOLD` | Максимальная cosine-distance для `recall` | `1.0` |
| `PLUGIN_DIR` | Каталог Python-плагинов | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Останавливать индексацию при ошибке derived-state hook | `false` |

## Примеры

### Минимальный набор (CPU)

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY
```

### С LM Studio

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
-e EMBEDDING_API_KEY=lm-studio `
-e EMBEDDING_MODEL=Qwen3-Embedding-4B
```

### С OpenRouter

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e EMBEDDING_API_BASE=https://openrouter.ai/api `
-e EMBEDDING_API_KEY=YOUR_OPENROUTER_KEY `
-e EMBEDDING_MODEL=qwen/qwen3-embedding-8b
```

### С Ollama

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e EMBEDDING_API_BASE=http://host.docker.internal:11434/v1 `
-e EMBEDDING_API_KEY=ollama `
-e EMBEDDING_MODEL=qwen3:embedding-4b
```
