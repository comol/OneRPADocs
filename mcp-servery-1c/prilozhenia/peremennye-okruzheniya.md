# Переменные окружения

Сводная таблица переменных окружения для всех MCP-серверов.

## Общие переменные

Эти переменные используются большинством серверов:

| Переменная | Описание | Обязательная | По умолчанию |
|------------|----------|--------------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да | — |
| `RESET_DATABASE` | Переиндексировать данные в серверах, которые поддерживают сохранённый индекс | Нет | `false` |
| `RESET_CACHE` | Очистить кэш моделей при старте | Нет | `false` (в HelpSearchServer и TemplatesSearchServer) |
| `USESSE` | Включить SSE-транспорт для legacy-клиентов. При `false` используется `streamable-http`; Graph Metadata Search использует отдельную переменную `MCP_USE_SSE` | Нет | `false` |

## Переменные дистрибутива

Эти значения читаются установочным дистрибутивом из `config.env` и не передаются контейнеру под тем же именем:

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `IMAGE_VARIANT` | Базовый вариант: `latest`, `light` или `arm64` | `latest` |
| `IMAGE_TAG` | Явный тег; если задан, перекрывает вариант | *(пусто)* |
| `PATH_1C_BIN` | Путь на хосте к каталогу `bin` платформы 1С для HelpSearchServer | — |
| `PATH_CODE` | Designer XML-выгрузка или корень базового проекта 1C:EDT на хосте | — |
| `PATH_EXTENSIONS` | Каталог выгрузок/проектов расширений; для 1C:EDT — общий родитель workspace | — |
| `PATH_BASES` | Корневой каталог постоянных данных серверов на хосте | — |
| `CODE_METADATA_SOURCE_FORMAT` | Поставочное значение `SOURCE_FORMAT` для CodeMetadataSearchServer: `auto`, `designer_xml` или `edt` | `auto` |
| `CODE_METADATA_INDEX_EXTENSIONS` | Поставочное значение `INDEX_EXTENSIONS`; команды установки передают его явно | `false` |
| `CODE_METADATA_EXTENSION_PATHS` | Список путей проектов-расширений внутри контейнера для `EXTENSION_PATHS` | *(пусто)* |
| `CHAT_API_BASE` / `CHAT_API_KEY` / `CHAT_MODEL` | OpenAI-совместимый LLM-провайдер для функций Graph, которым нужна генерация текста | зависит от профиля |
| `USE_GPU` | Добавлять GPU к full/arm64-команде запуска, если сервер и хост его поддерживают | `false` |
| `SUPPORT_KEY` / `SUPPORT_API_URL` / `SUPPORT_EMAIL` | Настройки команды `/support` из правил 1c-rules; контейнерам MCP не передаются | зависит от поставки |
| `LICENSE_KEY_<СЕРВЕР>` | Лицензионный ключ сервера; контейнеру передаётся как `LICENSE_KEY` | — |

Набор вариантов различается по серверам. См. [Теги и ключи образов](../kanaly-obrazov.md).

## Embedding модели (LM Studio / Ollama / OpenRouter)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_API_BASE` | OpenAI-совместимый base URL. CodeMetadataSearch, SSLSearch и TemplatesSearch автоматически добавляют `/v1`; для остальных серверов используйте формат из их профильной страницы | `http://host.docker.internal:1234/v1` |
| `EMBEDDING_API_KEY` | Ключ API | `lm-studio` |
| `EMBEDDING_MODEL` | Модель embedding для API или локального режима | `Qwen3-Embedding-4B` |
| `EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов (для моделей с переменной размерностью) | *(авто)* |

## Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face | `intfloat/multilingual-e5-base` |

{% hint style="info" %}
Если задан `EMBEDDING_API_BASE` или используется light-образ, сервер обращается к внешнему API. Старые `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` поддерживаются HelpSearchServer, CodeMetadataSearchServer, SSLSearchServer и TemplatesSearchServer как совместимые алиасы.
{% endhint %}

## Настройки индексации

Эти переменные управляют процессом индексации и доступны в серверах, где указано:

| Переменная | Описание | По умолчанию | Серверы |
|------------|----------|--------------|---------|
| `INDEX_BATCH_SIZE` | Размер пакета при добавлении в векторное хранилище | `512` | Graph |
| `MAX_TOKENS_PER_BATCH` | Максимум токенов в одном пакете API | `28000` | Graph |
| `EMBEDDING_MAX_TOKENS` | Максимум токенов на текст для эмбеддингов | *(авто)* | Graph |
| `BATCH_MAX_RETRIES` | Повторы пакета эмбеддингов после временной ошибки провайдера (обрыв, `503`, лимит); упавшие пакеты получают ещё один проход в конце лейна | `10` | Graph, CodeMetadata |
| `BATCH_BACKOFF_BASE` | Основание экспоненциальной паузы между повторами пакета | `2.0` | Graph, CodeMetadata |
| `BATCH_BACKOFF_MAX` | Максимальная пауза между повторами пакета, секунды | `60.0` | Graph, CodeMetadata |
| `REINDEX_INTERVAL_SEC` | Интервал автоматической инкрементальной индексации (секунды); `0` — отключить | `3600` | CodeMetadata |
| `REINDEX_INTERVAL_HOURS` | Алиас интервала в часах; `REINDEX_INTERVAL_SEC` имеет приоритет | *(не задано)* | CodeMetadata |
| `ENABLE_RERANKER` | Нейронный реранкер (cross-encoder) | `false` | CodeMetadata |
| `RERANKER_MODEL` | Модель реранкера | *(авто)* | CodeMetadata |
| `RERANKER_TOP_K` | Макс. кандидатов для реранкера | `20` | CodeMetadata |

## Переменные по серверам

### HelpSearchServer (порт 8003)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `1C_BIN_PATH` | Смонтированный каталог с `shcntx_ru.hbk`; не задан — берётся архив из образа | *(не задано)* |
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API, включая `/v1` | `http://host.docker.internal:1234/v1` |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | `lm-studio` |
| `EMBEDDING_MODEL` | Модель API или локальная модель | `intfloat/multilingual-e5-small` |
| `EMBEDDING_API_TIMEOUT` | Секунд ожидания ответа на один запрос к embedding API (с 23.09.2026) | `600` |
| `EMBEDDING_MAX_ATTEMPTS` | Сколько раз отправляется один запрос к API эмбеддингов: лимит запросов (`429`) и ответы `500`/`502`/`503` повторяются до этого числа; клиент сам запросы не повторяет | `6` |
| `BATCH_MAX_RETRIES` | Сколько раз пробуется пакет индексации, прежде чем он записывается как потерянный; каждая попытка — до `EMBEDDING_MAX_ATTEMPTS` запросов | `5` |
| `EMBEDDING_ALLOW_OFFLINE_FALLBACK` | Можно ли заменить недоступный на старте embedding API встроенной моделью. `false` — старт ждёт API (5 с, удваивая до 60 с, без ограничения числа попыток); при отказе 400/401/403/404 сервер переходит в `degraded`, индекс не трогается. `true` — как раньше: после неудачной проверки грузится встроенная модель (с 27.09.2026) | `false`, если задан `EMBEDDING_API_BASE`; иначе `true` |
| `HF_HOME` | Каталог кэша модели | `/app/model_cache` |
| `HF_HUB_OFFLINE` | Запрет загрузок при старте; `0` разрешает докачку | `1` |
| `RESET_CACHE` | Очистить кэш моделей при старте | `false` |
| `RESET_DATABASE` | Разрешить разрушающие операции и очистить индекс | `false` |
| `INDEX_RETAIN_GENERATIONS` | Сколько прошлых поколений индекса хранить | `1` |
| `INDEXING_WORKERS` | Максимум параллельных батчей индексации Help, включая запросы к embedding API; локальная модель и запись в индекс сериализованы | `5` |
| `INDEXING_BATCH_SIZE` | Строк в одной записи при индексации | `100` |
| `USESSE` | SSE-транспорт вместо streamable-http | `false` |
| `MCP_SESSION_IDLE_TTL_SECONDS` | Простой сессии Streamable HTTP до уборки | `1800` |
| `MCP_SESSION_MAX_LIFETIME_SECONDS` | Максимальное время жизни сессии | `86400` |
| `MCP_SESSION_CAP` | Максимум одновременных stateful-сессий | `1000` |
| `MCP_SESSION_CLEANUP_INTERVAL_SECONDS` | Интервал уборки сессий | `60` |
| `PLUGIN_DIR` | Каталог Python-плагинов | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Ронять сборку индекса при ошибке hook `on_document` | `false` |
| `RELEVANCE_MAX_VECTOR_DISTANCE` | Максимальная cosine-distance векторной дорожки | `0.35` |
| `RELEVANCE_MAX_LEXICAL_RANK` | Максимальный допустимый ранг лексического кандидата; отдельный score-порог не используется | `2` |
| `LEXICAL_PROFILE` | Токенизация лексической дорожки; смена вызывает переиндексацию | `stemmed` |

{% hint style="warning" %}
Индекс HelpSearchServer лежит в `/app/index` (поколения zvec), а не в `/app/chroma_db`, как в образах до 27.09.2026. Не подключайте один каталог данных одновременно к двум контейнерам. Полный список параметров: [Конфигурация HelpSearchServer](../servery/help-search-server/konfiguraciya.md).
{% endhint %}

### CodeMetadataSearchServer (порт 8000)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `LICENSE_KEY_FILE` | Путь к файлу с лицензионным ключом; предпочтительнее `LICENSE_KEY`, значение ключа не попадает в окружение процесса | *(не задано)* |
| `LICENSE_KEY_FILE_CONSUME` | Удалить файл ключа сразу после чтения | `false` |
| `METADATA_PATH` | Каталог готового текстового отчёта только для совместимого режима `METADATA_SOURCE=report`; в стандартном XML-режиме не требуется | *(пусто)* |
| `METADATA_SOURCE` | `xml` — метаданные из `CODE_PATH`; `report` — готовый отчёт; `auto` — выгрузка, иначе отчёт | `xml` |
| `CODE_PATH` | Путь к коду | `/app/code` |
| `MCP_HOST` | Хост для привязки сервера | `0.0.0.0` |
| `MCP_PORT` | Порт сервера; его же использует встроенная проверка здоровья контейнера | `8000` |
| `MCP_PATH` | Путь MCP-эндпоинта | `/mcp` |
| `FASTMCP_STATELESS_HTTP` | Stateless-режим HTTP | `true` |
| `MCP_STRUCTURED_CONTENT` | Дублировать ответы-словари и списки в `structuredContent`; при `false` публикуется только один JSON-блок `text content` | `false` |
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
| `EMBEDDING_MODEL` | Модель API или локальная модель. Полный образ без явной настройки использует `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`; удалённый профиль поставки закрепляет `qwen/qwen3-embedding-8b` | зависит от профиля |
| `RESET_DATABASE` | Переиндексировать | `false` |
| `BACKGROUND_INDEXING` | Индексировать в фоне, не блокируя запуск MCP | `true` |
| `INCREMENTAL_INDEXING` | Обновлять только изменившиеся файлы по SHA-256. Само переключение флага не требует повторного расчёта сохранённых embeddings | `true` |
| `INDEX_NESTED_CONFIGURATIONS` | Индексировать вложенные конфигурации поставщика отдельными источниками | `false` |
| `PROJECT_ID` | Явное закрепление идентификатора проекта индекса | *(выводится из путей)* |
| `GENERATION_RETENTION_COUNT` | Сколько поколений индекса хранить | `2` |
| `SUB_INDEX_INTEGRITY_GATE` | Режим проверки целостности вспомогательных индексов: `auto`, `blocking`, `report_only`, `off` | `auto` |
| `INDEX_STRUCTURAL` | Строить структурный индекс; `false` отключает дорожку `symbols` без блокировки `/ready` | `true` |
| `INDEX_DEPENDENCY_GRAPH` | Строить граф зависимостей; `false` также отключает запись объектов расширений из XML-прохода графа | `true` |
| `INDEX_FORM_INDEX` | Строить индекс форм; `false` отключает дорожку `forms` без блокировки `/ready` | `true` |
| `INDEX_XSD_SCHEMAS` | Генерировать XSD-схемы в help-фазе; `false` отключает XSD отдельно от HTML-справки | `true` |
| `SUB_INDEX_PROGRESS_WARN_SEC` | Порог предупреждения о долгой работе над файлом/стадией; `0` отключает диагностику | `300` |
| `SUB_INDEX_PROGRESS_HEARTBEAT_SEC` | Интервал повторных предупреждений о залипшей единице; живой проход пишет INFO `N/M file(s) done` не реже раза в минуту (или с этим интервалом, если он короче) | `300` |
| `SUB_INDEX_LIVENESS_WITNESS` | Независимый свидетель живости в отдельном процессе; сообщает о зависании, даже когда наблюдаемый интерпретатор не выполняет ни строки Python | `false` |
| `STRUCTURAL_PARSE_TIMEOUT_SEC` | Граница времени на структурный разбор одного BSL-модуля; положительное значение выносит разбор в дочерний процесс, `0` отключает границу | `0` |
| `STRUCTURAL_EXCLUDE` | JSON-массив glob-шаблонов (не больше 64) относительно `CODE_PATH`; модули исключаются только из структурного парсера | *(не задано)* |
| `LIVE_XML_FALLBACK` | Дочитывать XML-выгрузку, когда индекс не содержит факта | `true` |
| `NESTED_CONFIGURATION_PATHS` | Каталоги-контейнеры вложенных конфигураций через запятую | `Ext/ParentConfigurations` |
| `CHUNK_SIZE` | Совместимая настройка отпечатка и предела контекстного окна; текущий splitter использует фиксированные 1000 символов, поэтому переменная не меняет нарезку, но инвалидирует поколение | `1000` |
| `CHUNK_OVERLAP` | Совместимая настройка отпечатка; текущий splitter использует фиксированные 200 символов, поэтому переменная не меняет нарезку, но инвалидирует поколение | `200` |
| `INDEX_EXTENSIONS` | Индексировать найденные расширения конфигурации; обнаружение не отключается | `true` |
| `EXTENSION_PATHS` | Явные корни расширений через запятую (абсолютные или относительно `CODE_PATH`) | *(не задано)* |
| `EXTENSION_DISCOVERY_DEPTH` | Сколько уровней выше `CODE_PATH` просматривать при поиске соседних расширений; `0` — не искать | `1` |
| `EXTENSION_EXCLUDE` | Идентификаторы или имена расширений через запятую, которые не индексируются | *(не задано)* |
| `RESPONSE_MAX_CHARS` | Предел размера ответа в символах, если вызов не передал `max_chars` (сервер ограничивает диапазоном 2048–262144). `0` снимает предел | `32768` |
| `RESPONSE_MAX_ITEMS` | Максимум элементов на странице по умолчанию (потолок сервера — `500`) | `0` *(без предела)* |
| `RESPONSE_DETAIL_LEVEL` | Уровень детализации по умолчанию: `outline` или `full` | `full` |
| `REINDEX_INTERVAL_SEC` | Интервал автоматической индексации (секунды); `0` — отключить | `3600` |
| `REINDEX_INTERVAL_HOURS` | Алиас интервала в часах | *(не задано)* |
| `BM25_ALPHA` | Вес семантического поиска (0–1) | `0.5` |
| `MMR_ENABLED` | Снижать повторение похожих результатов после ранжирования, до хуков и ограничения выдачи | `false` |
| `MMR_LAMBDA` | Вес релевантности в MMR (0–1); меньший вес усиливает разнообразие | `0.5` |
| `CONTEXT_EXPANSION` | Контекст поиска: `siblings`, `window` или `none`; в режиме `siblings` справка получает соседние фрагменты | `siblings` |
| `CONTEXT_WINDOW_SIZE` | Число соседей с каждой стороны для расширения контекста | `1` |
| `OVERFETCH_MULTIPLIER` | Множитель выборки для запросов по пути/идентификатору | `4` |
| `SEMANTIC_OVERFETCH_MULTIPLIER` | Множитель выборки для семантических запросов | `6` |
| `MIN_SCORE_THRESHOLD` | Минимальный порог оценки результата (0–1) | `0.15` |
| `EMBEDDING_CACHE_SIZE` | Размер LRU-кэша эмбеддингов запросов | `256` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `EMBED_BATCH_SIZE_API` | Размер пакета для внешнего embedding API | `64` |
| `EMBED_BATCH_SIZE_LOCAL` | Размер пакета для локальной embedding-модели | `64` |
| `EMBED_QUEUE_CAPACITY` | Ёмкость очереди подготовленных пакетов | *(авто: четыре размера пакета)* |
| `BATCH_MAX_RETRIES` | Максимум попыток пакета при временной ошибке провайдера | `10` |
| `BATCH_BACKOFF_BASE` | Основание экспоненциальной паузы между попытками | `2.0` |
| `BATCH_BACKOFF_MAX` | Максимальная пауза между попытками, секунды | `60.0` |
| `EMBEDDING_API_TIMEOUT` | Секунд ожидания ответа на один запрос к OpenAI-совместимому API эмбеддингов (с 01.10.2026, вечер). Повторов внутри клиента нет — повторяет сервер по `BATCH_MAX_RETRIES` | `600` |
| `ENABLE_RERANKER` | Включить нейронный реранкер (cross-encoder) | `false` |
| `RERANKER_MODEL` | Модель реранкера | *(авто)* |
| `RERANKER_TOP_K` | Макс. кандидатов для реранкера | `20` |
| `EMBEDDING_PROVIDER` | Явный выбор `remote` или `local` | *(авто)* |
| `EMBEDDING_PROVIDER_AMBIGUITY` | Политика неоднозначной legacy-конфигурации | `error` |
| `EMBEDDING_MEMORY_BUDGET_MB` | Бюджет памяти локальной embedding-модели | `4096` |
| `EMBEDDING_MEMORY_BUDGET_MODE` | `refuse`, `warn` или `off` | `refuse` |
| `VECTOR_PROFILE` | Профиль zvec: `fast_index`, `balanced`, `memory_saver`, `quality` | `fast_index` |
| `VECTOR_OPTIMIZE_ENABLED` | Разрешить оптимизацию zvec | `true` |
| `VECTOR_OPTIMIZE_EVERY` | Число новых документов между промежуточными слияниями векторного индекса; `0` — только финальное слияние в конце фазы (в образах до 27.09.2026 значение `0` не действовало) | `100000` |
| `VECTOR_OPTIMIZE_FINAL_MIN_DOCS` | Финальное слияние не запускается, пока его ждут меньше N документов (исход `skipped_below_threshold`): они находятся поиском, переходят в следующее поколение и сливаются, когда их наберётся N; переиндексацию не вызывает. `0` — сливать любой непустой хвост | `0` |
| `VECTOR_FLUSH_EVERY` | Число новых документов между промежуточными сбросами на диск; `0` — только финальный сброс | `2000` |
| `VECTOR_OPTIMIZE_DEADLINE_SEC` | Таймаут фоновой оптимизации | `1800` |
| `VECTOR_OPTIMIZE_CANCEL_DEADLINE_SEC` | Таймаут оптимизации из запроса | `5` |
| `VECTOR_WRITE_FAILURE_THRESHOLD` | Ошибки записи до терминального состояния | `5` |
| `GREP_MAX_RESULTS` | Максимум результатов fallback-сканирования | `50` |
| `GREP_FILE_CACHE_SIZE` | Размер LRU-кэша прочитанных файлов | `500` |
| `GREP_BROAD_DIR_THRESHOLD` | Максимальная ширина дерева fallback-сканирования | `10000` |
| `GREP_DEADLINE_SEC` | Общий бюджет одного fallback-сканирования, секунды; `0` отключает | `10` |
| `GREP_MAX_CACHED_FILE_MB` | Файлы крупнее читаются, но не кэшируются | `8` |
| `MCP_TOOL_WORKERS` | Потоки для тел MCP-инструментов, чтобы долгий поиск не блокировал event loop | `4` |
| `PLUGIN_DIR` | Каталог Python-плагинов; читается при каждом старте, флага включения нет | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Ронять сборку целиком при ошибке derived-state hook (`on_source_file`, `on_chunk`, `on_metadata_object`) вместо пропуска единицы | `false` |
| `PLUGIN_HOOK_TIMEOUT_SECONDS` | Бюджет времени одного вызова hook; превысивший его hook считается упавшим | `5.0` |

{% hint style="warning" %}
`INDEX_STRUCTURAL`, `INDEX_DEPENDENCY_GRAPH`, `INDEX_FORM_INDEX`, `INDEX_XSD_SCHEMAS`, `SUB_INDEX_PROGRESS_WARN_SEC`, `SUB_INDEX_PROGRESS_HEARTBEAT_SEC`, `SUB_INDEX_LIVENESS_WITNESS`, `STRUCTURAL_PARSE_TIMEOUT_SEC`, `STRUCTURAL_EXCLUDE`, `GREP_DEADLINE_SEC`, `GREP_MAX_CACHED_FILE_MB`, `MCP_TOOL_WORKERS` и `VECTOR_OPTIMIZE_FINAL_MIN_DOCS` относятся к текущим исходникам CodeMetadataSearchServer. В ранее опубликованных образах их может ещё не быть.
{% endhint %}

### SSLSearchServer (порт 8008)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `SSL_VERSION` | Версия БСП | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `false` |
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Имя модели для API эмбеддингов | `qwen/qwen3-embedding-8b` |
| `LOCAL_EMBEDDING_MODEL` | Резервная локальная CPU-модель (Hugging Face repo id). Совместимый алиас — `OFFLINE_EMBEDDING_MODEL` | `intfloat/multilingual-e5-small` |
| `EMBEDDING_ALLOW_OFFLINE_FALLBACK` | Разрешить вариантам `latest`/`arm64` переходить на `LOCAL_EMBEDDING_MODEL`, если API эмбеддингов недоступен на старте. `true` — разрешить, любое другое значение — запретить. При запрете старт ждёт API (пауза 5 с, удваивается до 60 с), а при 401/403/404 завершается с «No embedding backend available», как `light`. Даже при разрешённом переходе коллекция, построенная через API, не удаляется и не пересобирается: старт останавливается, коллекция остаётся нетронутой. На `light` не влияет. С 27.09.2026 | `false`, если задан `EMBEDDING_API_BASE` (или `OPENAI_API_BASE`); иначе `true` |
| `INDEXING_THREADS` | Потоки индексации | `5` |
| `EMBEDDING_MAX_ATTEMPTS` | Сколько раз повторять один запрос к провайдеру эмбеддингов при временной ошибке (1–10) | `4` |
| `EMBEDDING_BATCH_MAX_ATTEMPTS` | Сколько раз пробовать один пакет индексации, прежде чем пропустить его (1–10). Пакет делает не больше `EMBEDDING_BATCH_MAX_ATTEMPTS` × `EMBEDDING_MAX_ATTEMPTS` запросов в пределах `EMBEDDING_BATCH_DEADLINE` | `5` |
| `MIGRATE_VECTOR_STORE` | Выполнить миграцию векторного хранилища на этом старте | `false` |
| `DEMOTE_VECTOR_STORE` | Вернуть обслуживание предыдущему поколению | `false` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `EMBEDDING_INPUT_TYPE_ENABLED` | Различение query/document для эмбеддингов | `true` |
| `FORCE_REINDEX_ON_DIMENSION_MISMATCH` | Автопересоздание при несовпадении размерности; иначе старт останавливается с ошибкой. Не действует, если старт перешёл на локальную модель поверх коллекции, построенной через API: такую коллекцию сервер не пересоздаёт (с 27.09.2026) | `false` |
| `MIN_SCORE` | Порог cosine similarity для результатов `ssl_search` | `0.3826` |
| `EXACT_LOOKUP` | Точный поиск по имени символа перед семантическим (`lane=exact`) | `true` |
| `HYBRID_SEARCH` | Гибридное извлечение: векторная + полнотекстовая (BM25) дорожки с RRF | `true` |
| `MAX_RESPONSE_CHARS` | Бюджет длины ответа `ssl_search` в символах (минимум 1024) | `4000` |
| `MAX_QUERY_CHARS` | Максимальная длина запроса до и после plugin/alias rewrite; минимум 256 | `2048` |
| `MAX_GUARDED_SEARCHES` | Максимум поисков внутри deadline guard | `64` |
| `SHUTDOWN_GRACE_SECONDS` | Бюджет корректного завершения контейнера, секунды | `10` |
| `SHUTDOWN_FLUSH_SECONDS` | Часть бюджета на сброс векторного хранилища, секунды | `5` |
| `LOG_QUERIES` | Записывать полный текст поисковых запросов вместо длины и хеша | `false` |
| `LOG_DIR` | Каталог файла журнала; относительный путь от корня приложения | `<app>/logs` |
| `SESSION_IDLE_TTL` | Время простоя HTTP-сессии до освобождения, секунды | `900` |
| `SESSION_MAX_LIFETIME` | Максимальное время жизни HTTP-сессии, секунды | `14400` |
| `SESSION_MAX_CONCURRENT` | Максимум одновременных HTTP-сессий | `128` |
| `SESSION_CLEANUP_INTERVAL` | Интервал очистки просроченных сессий, секунды | `30` |
| `PLUGIN_DIR` | Каталог Python-плагинов | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Останавливать индексацию при ошибке derived-state hook | `false` |

### Graph Metadata Search (порт 8006)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `NEO4J_URI` | URI Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь | `neo4j` |
| `NEO4J_USER` | Устаревший алиас `NEO4J_USERNAME`; при обеих заданных приоритет у `NEO4J_USERNAME` | — |
| `NEO4J_PASSWORD` | Пароль | Обязательно |
| `METADATA_DIRECTORY` | Каталог готового текстового отчёта; при `auto` выгрузка его перекрывает | `/app/metadata` |
| `METADATA_SOURCE` | `auto` — Designer XML, иначе EDT, иначе `*.txt`; `report` — только отчёт; `xml` — только Designer XML; `edt` — только проект EDT | `auto` |
| `METADATA_FALLBACK_DIR_1`, `METADATA_FALLBACK_DIR_2`, `METADATA_FALLBACK_DIR_3` | Дополнительные каталоги готового отчёта, проверяемые по порядку после `METADATA_DIRECTORY`; используются только если заданы явно | — |
| `NEO4J_DATABASE` | Имя базы Neo4j | `neo4j` |
| `NEO4J_PARALLEL_WRITE_WORKERS` | Число параллельных потоков записи в Neo4j при индексации, диапазон `1..16` | `1` |
| `NEO4J_READ_TRANSACTION_TIMEOUT_S` | Предел времени одного запроса чтения к Neo4j (серверный таймаут транзакции) для инструментов и веб-поиска; фоновые дорожки индексации читают без предела; `0` — без предела | `30.0` |
| `PROJECT_NAME` | Название проекта | `1C Metadata Project` |
| `RESET_DATABASE` | Сбросить векторы эмбеддингов при смене модели; граф не пересобирает (полная пересборка — очистка каталогов Neo4j и `/app/data`) | `false` |
| `INDEX_BATCH_SIZE` | Размер пакета индексации | `512` |
| `GRAPH_FORM_XML_BATCH_SIZE` | Сколько форм вместе проходят bulk-поиск владельцев и resume; запись может делиться на несколько транзакций по порогу строк | `50` |
| `GRAPH_FORM_XML_BATCH_MAX_ROWS` | Порог сброса, проверяемый после целой формы; не является жёстким пределом транзакции | `20000` |
| `MAX_TOKENS_PER_BATCH` | Макс. токенов на пакет API | `28000` |
| `EMBEDDING_REQUEST_CONCURRENCY` | Параллельные запросы к API эмбеддингов | `6` |
| `EMBEDDING_MAX_ATTEMPTS` | Сколько раз отправляется один запрос к API эмбеддингов, считая первую попытку (1–10), — и при индексации, и для эмбеддинга поискового запроса. На пакет индексации — не больше `EMBEDDING_MAX_ATTEMPTS` × (`BATCH_MAX_RETRIES` + 2) запросов | `3` |
| `OPENAI_EMBEDDING_DIMENSIONS` | Размерность эмбеддингов, запрашиваемая у API | *(авто)* |
| `VECTOR_INDEX_DIMENSION` | Ожидаемая размерность векторного индекса процедур; если не задана, читается из метаданных индекса | *(авто)* |
| `EMBEDDING_API_BASE` | URL API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Модель API эмбеддингов | `qwen/qwen3-embedding-8b` |
| `LOCAL_EMBEDDING_MODEL` | Резервная локальная CPU-модель. Совместимый алиас — `OFFLINE_EMBEDDING_MODEL` | `intfloat/multilingual-e5-small` |
| `ENABLE_CODE_SEARCH` | Поиск по BSL-коду | `true` |
| `ENABLE_BUSINESS_SEARCH` | Семантический поиск по бизнес-описаниям | `true` |
| `CALCULATE_BUSINESS_INFO` | Использовать бизнес-описания: генерировать LLM и подтягивать `business_info.html` рядом с объектом | `false` |
| `BUSINESS_INFO_UPDATE_POLICY` | Когда автоматический запуск вправе вызвать LLM: `never` (только объекты без описания), `threshold` (плюс объекты, изменившиеся сильнее порога), `manual` (только явный запуск) | `never` |
| `BUSINESS_INFO_CHANGE_THRESHOLD` | Порог доли структурных изменений объекта для `threshold`, от `0` до `1` | `0.2` |
| `ENABLE_METADATA_DESCRIPTION_EMBEDDING` | Эмбеддинги для описательных полей | `true` |
| `MCP_HOST` | Хост MCP-сервера | `0.0.0.0` |
| `MCP_PORT` | Порт MCP | `8006` |
| `MCP_PATH` | Совместимое поле конфигурации; текущий сервер его не применяет, MCP-эндпоинт фиксирован на `/mcp` | `/mcp` |
| `MCP_USE_SSE` | Использовать legacy SSE-транспорт вместо `streamable-http` | `false` |
| `FASTMCP_STATELESS_HTTP` | Не хранить серверную HTTP-сессию для `streamable-http` | `true` |
| `FASTMCP_JSON_RESPONSE` | Отвечать на POST обычным JSON вместо SSE-потока; используется вместе со stateless-режимом | `true` |
| `MCP_TOOL_PROFILE` | Профиль публикуемых инструментов: `admin` или `read-only` | `admin` |
| `MCP_NAMESPACE` | Namespace регистрации графовых проектов | `default` |
| `GRAPH_SCOPE_ENFORCED` | Записан ли scope в графе: при `true` и закрытом миграционном окне `project_id` обязателен; при `false` отсутствующий id подставляется из единственного/legacy-проекта и ответ помечается `deprecated` | `false` |
| `GRAPH_SCOPE_MIGRATION_WINDOW` | При scoped-графе разрешить временную подстановку единственного проекта. При `GRAPH_SCOPE_ENFORCED=false` legacy-вызовы обслуживаются независимо от окна и помечаются `deprecated` | `false` |
| `INGESTION_COORDINATOR_ENABLED` | Включить координатор загрузки с фазами, лизом и чекпоинтами | `false` |
| `INGESTION_LEASE_TTL_SECONDS` | Время жизни лиза загрузчика; heartbeat продлевает его каждые TTL/3 | `120` |
| `INGESTION_CHECKPOINT_BATCH_SIZE` | Размер пакета между чекпоинтами | `500` |
| `INGESTION_CHECKPOINT_INTERVAL_SECONDS` | Интервал записи чекпоинтов | `30` |
| `EMBEDDING_CARRY_BATCH_MODULES` | Изменённых модулей в одной пачке переноса эмбеддингов процедур при инкрементальном обновлении; ограничивает пиковую память | `100` |
| `INGESTION_TRACKER_BACKEND` | Хранилище состояния загрузки: `json` или `neo4j` | `json` |
| `INGESTION_STATE_DIRECTORY` | Каталог состояния для backend `json` | — |
| `INDEXING_STATE_PATH` | Путь к JSON-состоянию фоновых задач старта | `<app>/data/.indexing_state.json` |
| `GRAPH_MAX_ITEMS` | Жёсткий предел элементов в ответе | `200` |
| `GRAPH_TOOL_TIMEOUT_SECONDS` | Бюджет времени одного вызова инструмента, в секундах. По истечении сервер отвечает типизированной ошибкой `timeout`; начатая работа при этом не прерывается. Значение по умолчанию оставляет запас до типичного 30-секундного лимита MCP-клиента. Действует на все инструменты, включая административные. Долгие административные операции — `refresh_graph_project` и (образы от 04.10.2026) `refresh_extension_layers` — сразу отвечают `accepted` и выполняются на сервере, поднимать бюджет ради них не нужно. `0` — без ограничения | `25` |
| `EMBEDDING_ALLOW_OFFLINE_FALLBACK` | Автопереход на локальную модель | `true` |
| `TEMPLATE_MODE_ENABLED` | Шаблонный режим (JSON-запросы без LLM) | `true` |
| `TEMPLATE_MODE_ONLY` | Только шаблоны, без LLM | `false` |
| `CODE_SEARCH_MAX_FILE_SIZE` | Объявленный порог размера BSL-файла (байт). Индексация его не применяет: ни отбора, ни обрезки нет, файл читается целиком. Поднимать значение ради полноты индекса бессмысленно. Подробности — в конфигурации GraphMetadataSearch | `50000` |
| `CODE_EXPORT_PATH` | Путь к XML-выгрузке в файлы | — |
| `LOAD_BSL_SIGNATURES` | Загружать BSL-граф (Module/Routine/CALLS) | `true` |
| `ENABLE_ROUTINE_EMBEDDINGS` | Эмбеддинги для процедур/функций | `true` |
| `LOAD_FORMS_FROM_XML` | Загружать структуру управляемых форм из XML | `false` |
| `LOAD_ORDINARY_FORMS` | Загружать структуру обычных форм | `true` |
| `LOAD_EVENT_SUBSCRIPTIONS` | Загружать подписки на события | `false` |
| `LOAD_PREDEFINED_VALUES` | Загружать предопределённые элементы | `false` |
| `LOAD_ROLE_RIGHTS` | Загружать права ролей | `false` |
| `LOAD_HELP_FROM_HTML` | Загружать справку из HTML | `false` |
| `LOAD_DCS_TEMPLATES` | Загружать схемы компоновки данных (СКД) из макетов отчётов и обработок: узлы `DcsDataSet`, `DcsField`, `DcsParameter`, `DcsGrouping`, `DcsFilter`, `DcsTemplateArea` под макетом `Layout` — данные `get_report_dcs_lineage`. Состав узлов и связей — на странице конфигурации Graph | `false` |
| `EXTENSION_NAME` | Имя расширения | — |
| `EXTENSION_BASE_PROJECT` | Имя базового проекта для расширения | — |
| `EXTENSION_BASE_PROJECT_ID` | `PROJECT_ID` базовой конфигурации, если он отличается от её отображаемого имени | — |
| `EXTENSION_APPLY_ORDER` | Порядок применения слоя расширения | `1` |
| `EXTENSION_CATALOG_ENABLED` | Загружать все расширения из каталога массовой выгрузки за один прогон | `false` |
| `EXTENSIONS_PATH` | Каталог с выгрузками расширений внутри контейнера; пусто — подкаталоги каталога выгрузки. В поставляемых compose-профилях — `/app/extensions` | *(пусто)* |
| `EXTENSIONS_HOST_PATH` | Переменная compose-файла, а не сервера: каталог расширений на хосте, монтируемый только на чтение в `/app/extensions` | *(зависит от профиля)* |
| `EXTENSION_ORDER_MANIFEST` | JSON с порядком применения расширений внутри одного назначения | *(пусто)* |
| `EXTENSION_CATALOG_SYNC` | Синхронизировать слои с каталогом (удалять исчезнувшие расширения) | `true` |
| `GRAPH_PLUGINS_ENABLED` | Загружать плагины Graph Metadata Search | `true` |
| `GRAPH_PLUGINS_DIRECTORY` | Каталог плагинов | `plugins` |
| `GRAPH_PLUGIN_STRICT_BUILD` | Останавливать построение поколения при ошибке derived-state hook | `false` |
| `GRAPH_PLUGIN_HOOK_TIMEOUT_SECONDS` | Бюджет времени одного plugin hook; `0` отключает контроль | `5.0` |

{% hint style="warning" %}
`GRAPH_FORM_XML_BATCH_SIZE` и `GRAPH_FORM_XML_BATCH_MAX_ROWS` описывают текущие исходники. Наличие в опубликованном образе проверяйте по release notes.
{% endhint %}

Полный список переменных Graph Metadata Search, включая лимиты графовых ответов, поколения и загрузку данных: [Конфигурация Graph Metadata Search](../servery/graph-metadata-search/konfiguraciya.md).

### 1CCodeChecker (порт 8007)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `ONEC_AI_TOKEN` | Токен 1С:Напарник | Обязательно |
| `MCP_TOOL_CALL_MODE` | Режим вызова upstream: `direct` (прямые вызовы) или `standard` (промпты) | `direct` |
| `ONEC_AI_BASE_URL` | Базовый URL API 1С.ai | `https://code.1c.ai` |
| `ONEC_AI_TIMEOUT` | Таймаут отдельного запроса к API (секунды): connect, write, получение соединения из пула и чтение любого непотокового запроса. SSE-поток им **не** обрывается — его границей чтения служит бюджет операции | `30` |
| `ONEC_AI_OPERATION_TIMEOUT` | Бюджет времени на всю операцию, включая чтение SSE и повторные попытки (секунды). Должен помещаться в таймаут инструмента вашего MCP-клиента | `300` |
| `ONEC_AI_TRANSPORT_RETRIES` | Сколько дополнительных попыток делать при транспортном сбое (сетевая ошибка, таймаут одного запроса, HTTP 5xx/429), каждая — на свежей дискуссии и в пределах бюджета операции. `0` — одна попытка | `2` |
| `ONEC_AI_SKILL_NAME` | Режим сессии: `custom` (с инструментами) или `raw` | `custom` |
| `ONEC_AI_INPUT_MAX_LENGTH` | Максимальная длина каждого входного поля | `100000` |
| `ONEC_AI_WORKSPACE_ROOTS` | Абсолютные корни, внутри которых сервер может читать пути из `files`; пустое значение запрещает чтение | `/workspace` в Docker |
| `ONEC_AI_WORKSPACE_MAX_FILE_BYTES` | Максимальный размер одного файла для `files` | `2000000` |
| `ONEC_AI_WORKSPACE_PATH_MAP` | Пары `префикс_на_машине_клиента=корень_в_контейнере`, разделённые `;` или переводами строк. Каждый корень обязан входить в `ONEC_AI_WORKSPACE_ROOTS` | *(пусто)* |
| `ONEC_AI_DOC_VERSION` | Версия документации платформы по умолчанию (конкретная версия, не `latest`) | `v8.5.1` |
| `ONEC_AI_UI_LANGUAGE` | Язык интерфейса | `russian` |
| `ONEC_AI_PROGRAMMING_LANGUAGE` | Язык программирования | *(пусто)* |
| `ONEC_AI_SCRIPT_LANGUAGE` | Скриптовый язык (`ru` / `en`) | `ru` |
| `ONEC_CONFIG_NAME` | Конфигурация 1С по умолчанию для `config_help` | *(пусто)* |
| `MAX_ACTIVE_SESSIONS` | Макс. активных upstream-дискуссий | `10` |
| `SESSION_TTL` | Время жизни upstream-дискуссии (секунды) | `3600` |
| `ONEC_AI_CONVERSATION_BUSY_TIMEOUT` | Ожидание освобождения занятой дискуссии (секунды) | `60` |
| `MCP_TRANSPORT_SESSION_IDLE_TIMEOUT` | Простой транспортной сессии MCP до закрытия (секунды) | `900` |
| `MCP_TRANSPORT_SESSION_MAX_LIFETIME` | Абсолютное время жизни транспортной сессии MCP (секунды) | `28800` |
| `MCP_MAX_TRANSPORT_SESSIONS` | Максимум одновременных транспортных сессий MCP | `100` |
| `MCP_TRANSPORT_SESSION_SWEEP_INTERVAL` | Период уборки истёкших сессий (секунды) | `30` |
| `MCP_TRANSPORT_SESSION_SWEEP_BATCH` | Максимум сессий, закрываемых за один проход уборки | `100` |
| `HTTP_PORT` | Порт HTTP-сервера | `8007` |
| `CHECKER_IMAGE_DIGEST` | Переданный оператором registry digest вида `sha256:<64 lowercase hex>` для `/release`; образ должен запускаться по тому же digest | *(не задано)* |
| `PLUGIN_DIR` | Каталог Python-плагинов | `/app/plugins` |

{% hint style="warning" %}
`CHECKER_IMAGE_DIGEST` относится к текущим исходникам и ещё не подтверждён в опубликованных образах. Без переменной `/release` возвращает `image_digest_available=false`; два пустых digest не доказывают идентичность образов.
{% endhint %}

### SyntaxCheckServer (порт 8002)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `USESSE` | SSE транспорт | `false` |
| `FILES_DIR` | Каталог с файлами BSL внутри контейнера. Если каталог задан и существует, сервер регистрирует инструмент `syntaxcheck_file` | *(пусто)* |
| `FULLINDEX` | `true`/`1`/`yes`/`on` включает режим полного индекса: `FILES_DIR` индексируется при старте, и `UnresolvedMethodCall`, `UnresolvedField`, `QueryToMissingMetadata` отвечают из индекса | *(пусто)* |
| `INDEX_DIR` | Каталог индекса конфигурации; образ объявляет его томом | `/index` |
| `FULLINDEX_REINDEX_INTERVAL_SEC` | Только для режима полного индекса: как долго готовый индекс не переспрашивают, то есть как быстро подхватывается изменение исходников после первой сборки. Отключить переиндексацию нельзя: `0`, отрицательное значение и не-число читаются как значение по умолчанию, положительное ограничивается диапазоном 60…86400 секунд | `3600` |
| `PLUGINS_DIR` | Каталог Python-плагинов; пустое значение использует `/app/plugins` | *(пусто)* |
| `LOG_LEVEL` | Уровень журналирования | `INFO` |
| `MCP_HTTP_PATH` | Endpoint `streamable-http` | `/mcp` |
| `MCP_SSE_PATH` | Endpoint потока событий при `USESSE=true` | `/sse` |
| `MCP_MESSAGE_PATH` | Endpoint сообщений при `USESSE=true` | `/messages/` |
| `RESPONSE_MAX_CHARS` | Предел размера ответа `syntaxcheck` / `syntaxcheck_file` в байтах: текст и структурированная часть вместе. Если находки не помещаются, остаются самые серьёзные, `summary.truncated` равен `true`. `0` снимает предел, значение меньше 2048 поднимается до 2048 | `32768` |
| `BSL_ANALYZER_TIMEOUT_SECONDS` | Таймаут анализатора | `30` |
| `BSL_ANALYZER_STDOUT_LIMIT_BYTES` | Лимит JSONL-отчёта анализатора | `16777216` |
| `BSL_ANALYZER_STDERR_LIMIT_BYTES` | Лимит диагностического вывода анализатора | `4194304` |
| `BSL_ANALYZER_KILL_GRACE_SECONDS` | Ожидание перед принудительной остановкой | `2` |
| `BSL_SOURCE_ENCODING` | Явная кодировка файлов; если не задана, пробуются UTF-8 с BOM и CP1251 | *(пусто)* |
| `MCP_SESSION_IDLE_TTL_SECONDS` | Простой Streamable HTTP-сессии до уборки, секунды | `1800` |
| `MCP_SESSION_MAX_LIFETIME_SECONDS` | Абсолютное время жизни сессии, секунды | `28800` |
| `MCP_SESSION_MAX_CONCURRENT` | Максимум одновременных Streamable HTTP-сессий | `64` |
| `MCP_SESSION_REAP_INTERVAL_SECONDS` | Интервал уборки просроченных сессий, секунды | `30` |

### TemplatesSearchServer (порт 8004)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `RESET_DATABASE` | Построить новое поколение индекса шаблонов | `false` |
| `RESET_CACHE` | Удалить кэш весов embedding-модели и скачать заново | `false` |
| `HTTP_PORT` | Порт HTTP-сервера | `8004` |
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API эмбеддингов | — |
| `EMBEDDING_API_KEY` | Ключ API эмбеддингов | — |
| `EMBEDDING_MODEL` | Имя модели для API эмбеддингов | `qwen/qwen3-embedding-8b` |
| `LOCAL_EMBEDDING_MODEL` | Резервная локальная CPU-модель (Hugging Face repo id). Совместимый алиас — `OFFLINE_EMBEDDING_MODEL` | `intfloat/multilingual-e5-small` |
| `EMBEDDING_DIMENSIONS` | Размерность эмбеддингов | *(авто)* |
| `EMBEDDING_API_TIMEOUT` | Предел одного запроса к API эмбеддингов, секунды (на попытку). Ограничивает ожидание семантической полосы, за которой стоит полнотекстовая | `60` |
| `EMBEDDING_MAX_ATTEMPTS` | Сколько раз отправляется один запрос к API эмбеддингов: первая попытка и повторы клиента после таймаута, обрыва соединения, `429` или `5xx`. `1` — без повторов | `3` |
| `EMBEDDING_ALLOW_OFFLINE_FALLBACK` | Разрешить переход на `LOCAL_EMBEDDING_MODEL`, если API эмбеддингов не ответил при старте. При запрете старт повторяется (5, 15, 30 с), затем процесс завершается с кодом 70 до следующего запуска контейнера; индекс не трогается. Даже при `true` локальная модель не заменяет непустой индекс, построенный через API: старт отклоняется, индекс сохраняется (с 27.09.2026) | `false`, если задан `EMBEDDING_API_BASE`; иначе `true` |
| `TEMPLATES_DB_PATH` | Путь к SQLite-базе шаблонов и заметок | `/app/chroma_db/templates.db` |
| `ZVEC_DB_PATH` | Каталог векторного индекса zvec | `/app/chroma_db/zvec_db` |
| `RECALL_RELEVANCE_THRESHOLD` | Максимальная cosine-distance для `recall` | `1.0` |
| `TEMPLATE_RELEVANCE_THRESHOLD` | Максимальная cosine-distance для `templatesearch` | `1.0` |
| `FUSION_RANK_CONSTANT` | Константа reciprocal rank fusion | `60` |
| `FTS_DEFAULT_OPERATOR` | Объединение соседних терминов в полнотекстовом маршруте: `OR` или `AND` | `OR` |
| `GROUP_RESULT_CAP` | Максимум документов одного шаблона в ответе | `1` |
| `GROUP_CANDIDATE_BUDGET` | Сколько различных шаблонов запрашивается в выборке кандидатов | `11` |
| `INDEX_GENERATION_RETENTION` | Сколько поколений каждой коллекции хранится на диске | `2` |
| `PLUGIN_DIR` | Каталог Python-плагинов | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Останавливать индексацию при ошибке derived-state hook | `false` |
| `MCP_ENABLE_WRITE_TOOLS` | Регистрировать изменяющие инструменты `add_template` и `plugin_reload`. `remember` регистрируется всегда и от этой переменной не зависит | *(не задано — инструменты не публикуются)* |
| `MCP_OPERATOR_TOKEN` | Операторский токен для `add_template` / `plugin_reload`; передаётся в заголовке `Authorization`. Обязателен вместе с `MCP_ENABLE_WRITE_TOOLS`, иначе сервер не стартует. Для `remember` не нужен | *(не задано)* |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | Учётная запись для изменяющих web-операций | *(не заданы)* |
| `ADMIN_USERS` | Несколько web-операторов: `имя:пароль:разрешения;...` | *(пусто)* |
| `ADMIN_PERMISSIONS` | Разрешения единственного оператора: `create,edit,delete` | все три |
| `ADMIN_SESSION_SECRET` | Подпись web-сессий; без значения сессии не переживают рестарт | генерируется при старте |
| `ADMIN_ALLOW_UNAUTHENTICATED` | Небезопасно разрешить web-изменения без входа | `false` |
| `MAX_DESCRIPTION_BYTES` | Максимум байт в описании шаблона | `20000` |
| `MAX_CODE_BYTES` | Максимум байт в коде шаблона | `200000` |
| `MAX_MEMORY_BYTES` | Максимум байт в заметке | `20000` |
| `MAX_REQUEST_BODY_BYTES` | Максимум байт в теле web-запроса | `1000000` |
| `MCP_MAX_CONCURRENT_MUTATIONS` | Одновременные MCP-мутации | `2` |
| `MCP_MUTATION_RATE_PER_MINUTE` | Скорость мутаций в минуту | `30` |
| `MCP_MUTATION_BURST` | Разрешённый всплеск мутаций | `10` |
| `MCP_MUTATION_DAILY_QUOTA` | Суточная квота мутаций (UTC) | `500` |
| `MCP_MAX_MUTATIONS_PER_REQUEST` | Изменяющих calls в одном JSON-RPC запросе | `8` |
| `MCP_SESSION_IDLE_TIMEOUT` | Простой Streamable HTTP-сессии, секунды | `1800` |
| `MCP_SESSION_MAX_LIFETIME` | Абсолютное время жизни сессии, секунды | `28800` |
| `MCP_MAX_SESSIONS` | Максимум одновременных сессий | `200` |
| `MCP_SESSION_CLEANUP_INTERVAL` | Интервал уборки сессий, секунды | `60` |
| `MCP_SESSION_CLEANUP_BATCH` | Сессий за проход уборки | `50` |
| `INDEX_RECOVERY_INTERVAL` | Период проверки outbox, секунды | `60` |
| `INDEX_RECOVERY_BACKOFF` | Начальная пауза retry индексации, секунды | `5` |
| `INDEX_RECOVERY_BACKOFF_CAP` | Максимальная пауза retry, секунды | `300` |
| `INDEX_RECOVERY_MAX_ATTEMPTS` | Попыток до stuck state | `6` |
| `INDEX_RECOVERY_BATCH` | Маркеров outbox за проход | `100` |
| `EMBEDDING_TRUST_REMOTE_CODE` | Opt-in для custom model code; требует allowlist и immutable revision | `false` |
| `EMBEDDING_TRUST_REMOTE_CODE_MODELS` | Allowlist model ID для custom code | *(пусто)* |
| `EMBEDDING_MODEL_REVISION` | Полный commit SHA или content digest модели | *(пусто)* |

{% hint style="info" %}
Переменные `PLUGIN_DIR`, `PLUGINS_DIR`, `PLUGIN_STRICT_DERIVED_STATE`, `PLUGIN_HOOK_TIMEOUT_SECONDS` и `GRAPH_PLUGIN*` относятся к системе плагинов — общему механизму доработки серверов. Что они включают и чем это оплачивается: [Доработка MCP: система плагинов](../sistema-pluginov/).
{% endhint %}

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

## MCP QA (0.7.12)

- `LICENSE_KEY` — ключ QA; в MCP_Distr хранится как `LICENSE_KEY_QA`.
- `LICENSE_KEY_FILE` — UTF-8 файл с ключом, имеет приоритет над переменной.
- `MCP_QA_HTTP_TOKEN` — необязательный Bearer-токен HTTP, отдельный от лицензии.
- `MCP_QA_HOST` / `MCP_QA_HTTP_PORT` — адрес и порт слушателя (Docker: 0.0.0.0:8020).
- `MCP_QA_BACKEND` — `native` (по умолчанию вне Windows и в образе: сервер сам
  подключается к тест-клиенту) либо `manager` (Windows, сервер запускает 1С сам).
- `MCP_QA_EXECUTOR` — `native` или `platform`; в образе `native`.
- `MCP_QA_TESTCLIENT` — адрес тест-клиента `хост:порт`, по умолчанию
  `host.docker.internal:1538`.
- `MCP_QA_TESTCLIENT_USER` / `MCP_QA_TESTCLIENT_PASSWORD` / `MCP_QA_TESTCLIENT_DOMAIN` —
  необязательная учётная запись Windows для проверки подлинности канала
  тестирования (NTLM через `pyspnego`). Тест-клиент 8.3.27 принимает любую
  учётную запись, поэтому без них контейнер входит от имени-заглушки. Секрет.
- `MCP_QA_TESTCLIENT_ID` — идентификатор клиента для веб-клиента,
  запущенного с `TestClientID=<ид>`; для тонкого клиента пусто. Порт
  веб-клиента слушает веб-сервер публикации (по умолчанию 1538), его и
  указывают в `MCP_QA_TESTCLIENT`.
- `MCP_QA_CLIENT_BUS_URL` — адрес этого сервера, каким его видит машина
  тест-клиента, для сетевого канала расширения `MCPQAClient`
  (`<адрес>/client-bus/v1`). По умолчанию `http://127.0.0.1:<MCP_QA_HTTP_PORT>`;
  задайте, если порт опубликован иначе, например `http://127.0.0.1:8030`.
- `MCP_QA_TRANSPORT` — `stdio` либо `http`/`streamable-http`.
- `MCP_QA_COMMAND_TIMEOUT` — срок ожидания ответа тест-клиента на один запрос,
  когда инструмент не передал свой таймаут; по умолчанию `120` с, верхняя
  граница таймаута инструмента — 3600 с (0.7.5).
- В поставке: `QA_IMAGE`, `QA_HTTP_PORT`, `QA_HTTP_TOKEN` — настройки только QA.

Переменные прежних образов `MCP_QA_UPSTREAM_URL`/`MCP_QA_UPSTREAM_TOKEN` (0.5.0) и
`MCP_QA_TESTPILOT_TIMEOUT`/`TC1C_LOGGING` (0.4.4) образ 0.7.12 в нативном режиме не использует.

[Установка и первый сеанс QA](../servery/qa/README.md).

## Конвертация данных 2.0 (KD20)

- `LICENSE_KEY` — ключ KD20; в MCP_Distr хранится как `LICENSE_KEY_KD20`.
- `LICENSE_KEY_FILE` — UTF-8 файл с ключом, имеет приоритет над переменной.
- `KD20_HOST` / `KD20_HTTP_PORT` — адрес и порт слушателя (в образе `0.0.0.0:8009`).
- `MCP_TRANSPORT` — `http` (в образе) или `stdio`.
- `KD20_PATH_MAP` — пары `путь_машины=путь_контейнера` через `;`: ИИ передаёт пути своей
  машины, сервер переводит их в пути контейнера, а в ответах возвращает обратно.
  Пример: `E:\Dumps=/workspace;E:\bases\mcp\kd20=/data`.
- `KD20_ALLOWED_ROOTS` — каталоги, где разрешено читать и писать, через `;`
  (в образе `/data;/workspace`).
- `KD20_DATA` — рабочие данные: разобранные конфигурации, наборы правил, правила
  регистрации (в образе `/data`).
- `KD20_OUTPUT` — сформированные файлы правил (в образе `/data/output`).
- `KD20_SANDBOX_DATA_URL` — сервер данных копии базы 1С (1c-data-mcp, `…/hs/mcp`) для
  `check_rules_in_1c`; из контейнера база на этой машине —
  `http://host.docker.internal/<публикация>/hs/mcp`. Рабочую базу не указывать.
- `KD20_DB` — поисковый индекс справочников (в образе `/data/kd20.sqlite3`); пустой
  индекс наполняется при первом запуске.
- В поставке: `KD20_WORKSPACE` (папка с выгрузками и макетами правил, монтируется
  только на чтение в `/workspace`), `KD20_HTTP_PORT` и `KD20_SANDBOX_DATA_URL`; рабочие
  данные — `PATH_BASES\kd20`.
