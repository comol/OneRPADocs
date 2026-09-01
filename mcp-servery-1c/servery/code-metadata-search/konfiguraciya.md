# Конфигурация CodeMetadataSearchServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `LICENSE_KEY_FILE` | Предпочтительный способ передачи ключа: путь к смонтированному файлу, который содержит только лицензионный ключ. Значение ключа не попадает в окружение процесса. Файл должен быть обычным, непустым, небольшим и на POSIX недоступным для чтения группе и остальным | `/run/secrets/license_key` |
| `LICENSE_KEY_FILE_CONSUME` | Удалить файл ключа сразу после чтения | `false` |
| `CODE_PATH` | Корень Designer XML-выгрузки или проекта 1C:EDT | `/app/code` |

`METADATA_PATH` больше не обязателен. Он нужен только для совместимого режима `METADATA_SOURCE=report` с готовым текстовым отчётом.

### Настройки сервера

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_HOST` | Хост для привязки сервера | `0.0.0.0` |
| `MCP_PORT` | Порт сервера | `8000` |
| `MCP_PATH` | Путь MCP-эндпоинта | `/mcp` |
| `USESSE` | Включить SSE-транспорт для legacy-клиентов. По умолчанию используется `streamable-http` | `false` |
| `FASTMCP_STATELESS_HTTP` | Не хранить состояние между HTTP-вызовами; уменьшает число долгоживущих сессий | `true` |
| `MCP_SESSION_IDLE_TTL_SEC` | Время простоя сессии до освобождения; `0` отключает ограничение | `1800` |
| `MCP_SESSION_MAX_LIFETIME_SEC` | Максимальное время жизни сессии; `0` отключает ограничение | `86400` |
| `MCP_SESSION_MAX_CONCURRENT` | Максимум одновременных MCP-сессий | `64` |
| `MCP_SESSION_CLEANUP_INTERVAL_SEC` | Интервал очистки завершённых и просроченных сессий | `60` |
| `MCP_SESSION_BOUNDS_MODE` | `enforce` применяет лимиты, `report` только считает нарушения | `enforce` |
| `MCP_IMAGE_REF` | Неизменяемая ссылка на образ с digest для проверки release identity в `stats` | *(не задано)* |
| `METADATA_SOURCE` | Источник метаданных: `xml` — читать их из `CODE_PATH`; `report` — требовать готовый отчёт в `METADATA_PATH` | `xml` |
| `SOURCE_FORMAT` | Формат `CODE_PATH`: `auto`, `designer_xml` или `edt` | `auto` |
| `PROJECT_ID` | Явно закрепить идентификатор проекта индекса. Если не задан — выводится из каталогов `CODE_PATH` и `METADATA_PATH`; закрепление переживает перенос точки монтирования | *(выводится)* |
| `GENERATION_RETENTION_COUNT` | Сколько поколений индекса хранить на диске; значение меньше 1 повышается до 1 | `2` |
| `GENERATION_LEASE_RENEW_SEC` | Интервал продления аренды поколения | `30` |
| `GENERATION_LEASE_TIMEOUT_SEC` | Таймаут аренды поколения | `300` |
| `FILE_TRACKER_DB_PATH` | Совместимый путь к устаревшей базе трекера файлов | *(внутри `VECTOR_DB_PATH`)* |
| `VECTOR_DB_PATH` | Путь к общей директории векторного хранилища и служебных индексов | `/app/chroma_db` |
| `CHROMA_DB_PATH` | Совместимый старый алиас для `VECTOR_DB_PATH`; внутри каталога теперь используются zvec-коллекции и SQLite-индексы | `/app/chroma_db` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `BACKGROUND_INDEXING` | Выполнять стартовую индексацию в фоне, не блокируя запуск MCP | `true` |
| `INCREMENTAL_INDEXING` | Сверять SHA-256 файлов и обновлять только изменившиеся данные | `true` |
| `INCREMENTAL_HASH_CHUNK_BYTES` | Размер блока чтения при расчёте SHA-256 | `65536` |
| `REINDEX_INTERVAL_SEC` | Интервал периодической инкрементальной индексации в секундах. `0` — явное отключение (остаётся только ручной `reindex()`). Тик без изменений в исходниках стоит только расчёта SHA-256 и сверки инвентаря: поколение не создаётся и полосы индексации не запускаются | `3600` |
| `REINDEX_INTERVAL_HOURS` | Пользовательский алиас интервала в часах; явный `REINDEX_INTERVAL_SEC` имеет приоритет | *(не задано)* |
| `PARSE_WORKERS` | Число параллельных обработчиков файлов | от 1 до 8, по числу CPU |
| `EMBEDDING_CONCURRENCY` | Максимум параллельных запросов эмбеддингов | `6` |
| `SUB_INDEX_WORKERS` | Число параллельно строящихся вспомогательных индексов | `4` |
| `SUB_INDEX_PROGRESS_WARN_SEC` | Через сколько секунд непрерывной работы над одним файлом или стадией вывести предупреждение с относительным путём, стадией, временем, прогрессом и числом активных воркеров. Это только диагностика: она не прерывает и не отменяет работу. `0` полностью отключает монитор | `300` |
| `SUB_INDEX_PROGRESS_HEARTBEAT_SEC` | Минимальный интервал повторных предупреждений, пока та же работа остаётся активной. Значения меньше 5 секунд поднимаются до 5, чтобы исключить лавину сообщений | `300` |
| `SUB_INDEX_BULK_LOAD` | Загружать вспомогательные индексы пакетно | `true` |
| `SHUTDOWN_GRACE_SEC` | Время ожидания активных HTTP-запросов при остановке | `5` |
| `INDEX_METADATA` | Индексировать метаданные | `true` |
| `INDEX_CODE` | Индексировать BSL-код | `true` |
| `INDEX_STRUCTURAL` | Строить структурный индекс символов. `false` полностью исключает запись и финализацию этой дорожки, переводит readiness-дорожку `symbols` в `disabled`; `search_function` использует ограниченный grep-fallback | `true` |
| `INDEX_DEPENDENCY_GRAPH` | Строить граф зависимостей. `false` полностью исключает его запись и финализацию, а также запись объектов расширений из XML-прохода графа | `true` |
| `INDEX_FORM_INDEX` | Строить индекс форм. `false` полностью исключает запись и финализацию этой дорожки и переводит readiness-дорожку `forms` в `disabled` | `true` |
| `INDEX_HELP` | Индексировать HTML-справку | `true` |
| `INDEX_PHASE_ORDER` | Порядок фаз; метаданные всегда выполняются первыми | `metadata,code,help` |
| `INDEX_NESTED_CONFIGURATIONS` | Индексировать вложенные конфигурации поставщика отдельными источниками | `false` |
| `NESTED_CONFIGURATION_PATHS` | Список относительных каталогов-контейнеров вложенных конфигураций через запятую | `Ext/ParentConfigurations` |
| `SUB_INDEX_INTEGRITY_GATE` | Режим проверки целостности вспомогательных индексов: `auto`, `blocking`, `report_only` или `off` | `auto` |

`INDEX_CODE` остаётся общим переключателем векторной индексации BSL-кода. Три вспомогательные дорожки управляются независимо: выключенная дорожка не создаёт и не финализирует свои артефакты и не блокирует `/ready`. Если выключить все три, общий проход вспомогательных индексов пропускается целиком, но векторная индексация кода продолжается.

### Индексация расширений конфигурации

Расширения индексируются вместе с основной конфигурацией: явно указанные корни берутся первыми, а остальное сервер находит сам рядом с `CODE_PATH`. Происхождение каждого результата возвращается в блоке `origin` — см. [провенанс расширений](README.md#провенанс-расширений).

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `INDEX_EXTENSIONS` | Индексировать найденные расширения. `false` отключает индексацию, но не отключает их обнаружение: состав расширений по-прежнему виден в `stats` | `true` |
| `EXTENSION_PATHS` | Явные корни расширений через запятую; путь абсолютный или относительный к `CODE_PATH`. Несуществующий путь пропускается с предупреждением, а не останавливает индексацию | *(не задано)* |
| `EXTENSION_DISCOVERY_DEPTH` | На сколько уровней выше `CODE_PATH` подниматься при поиске соседних выгрузок расширений. `0` отключает поиск по соседям — тогда работают только `EXTENSION_PATHS` | `1` |
| `EXTENSION_EXCLUDE` | Идентификаторы или объявленные имена расширений через запятую, которые не индексируются. Обнаружение по-прежнему сообщает о них | *(не задано)* |

Набор расширений — свойство прогона, построившего индекс, а не запроса: изменение этих переменных не меняет происхождение уже сохранённых строк, оно применяется при следующей индексации.

### Бюджет ответа по умолчанию

Параметры вызова `max_chars`, `max_items` и `detail_level` со значением `0` / `""` берут значение из этих переменных. Явный параметр запроса всегда имеет приоритет.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESPONSE_MAX_CHARS` | Предел размера ответа в символах для всей установки. Значение ограничивается сервером диапазоном 2048–262144; выход за него отмечается полем `clamped` | `0` *(без предела)* |
| `RESPONSE_MAX_ITEMS` | Максимум элементов на странице для всей установки; серверный потолок — `500` | `0` *(без предела)* |
| `RESPONSE_DETAIL_LEVEL` | Уровень детализации по умолчанию: `outline` или `full` | `full` |

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
| `LIVE_XML_FALLBACK` | Дочитывать XML-выгрузку напрямую, когда индекс не содержит нужного факта | `true` |
| `LIVE_XML_CACHE_SIZE` | Размер кэша разобранных XML-файлов | `2000` |
| `LIVE_XML_MAX_FILES` | Максимум файлов, просматриваемых при live-чтении XML | `50000` |
| `GREP_FILE_CACHE_SIZE` | Размер кэша файлов текстового поиска по коду | `500` |
| `GREP_BROAD_DIR_THRESHOLD` | Порог числа файлов, после которого каталог считается слишком широким для текстового поиска | `10000` |
| `GREP_MAX_RESULTS` | Предел числа результатов слоя точного текстового поиска (grep) | `50` |
| `GREP_DEADLINE_SEC` | Общий бюджет одного fallback-сканирования в секундах; `0` отключает предел | `10` |
| `GREP_MAX_CACHED_FILE_MB` | Файлы крупнее этого размера читаются, но не остаются в LRU-кэше | `8` |
| `MCP_TOOL_WORKERS` | Число потоков для тел MCP-инструментов; долгий синхронный поиск не блокирует `/live` и другие вызовы | `4` |
| `INJECT_GRAPH_DEPENDENCIES` | Добавлять в индексируемый текст связи графа зависимостей объекта | `false` |

{% hint style="warning" %}
`INDEX_STRUCTURAL`, `INDEX_DEPENDENCY_GRAPH`, `INDEX_FORM_INDEX`, `SUB_INDEX_PROGRESS_WARN_SEC`, `SUB_INDEX_PROGRESS_HEARTBEAT_SEC`, `GREP_DEADLINE_SEC`, `GREP_MAX_CACHED_FILE_MB` и `MCP_TOOL_WORKERS` описывают текущую beta-кандидатную реализацию исходников. Перед применением к опубликованному образу проверьте наличие переменных в его release notes или `/release`; stable и ранее опубликованные beta-теги могут их не содержать.
{% endhint %}

Если grep-fallback упирается в время или ширину дерева, он возвращает уже найденное и помечает ответ `partial`: `reason=scan_deadline_reached` или `reason=scan_scope_capped`. Бюджет общий для всех проходов одного вызова, поэтому синонимы не умножают допустимое время. Файлы крупнее `GREP_MAX_CACHED_FILE_MB` по-прежнему читаются — ограничивается только кэширование.

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
| `VECTOR_OPTIMIZE_ENABLED` | Разрешить периодическую и финальную оптимизацию zvec | `true` |
| `VECTOR_OPTIMIZE_EVERY` | Число новых документов между оптимизациями; `0` — только финальная | `100000` |
| `VECTOR_OPTIMIZE_DEADLINE_SEC` | Максимальное ожидание оптимизации в фоновой индексации | `1800` |
| `VECTOR_OPTIMIZE_CANCEL_DEADLINE_SEC` | Максимальное ожидание оптимизации из запроса или принудительной переиндексации | `5` |
| `VECTOR_WRITE_FAILURE_THRESHOLD` | Последовательные ошибки записи до перевода коллекции в терминальное состояние | `5` |

### Нейронный реранкер (cross-encoder)

Опциональный нейронный реранкер для повышения качества семантического поиска. При включении оценивает пары (запрос, документ) совместно, что значительно улучшает релевантность результатов.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ENABLE_RERANKER` | Включить нейронный реранкер. `true` — использовать cross-encoder, `false` — использовать BM25 | `false` |
| `RERANKER_MODEL` | Модель реранкера. Если не указано — автовыбор в зависимости от окружения | *(авто)* |
| `RERANKER_TOP_K` | Максимальное количество кандидатов, передаваемых реранкеру | `20` |
| `RERANKER_API_BASE` | URL API реранкера, если он отличается от `EMBEDDING_API_BASE` / `OPENAI_API_BASE` | *(не задано)* |
| `RERANKER_API_KEY` | Ключ API реранкера, если он отличается от ключа эмбеддингов | *(не задано)* |
| `RERANKER_REQUEST_TIMEOUT_S` | Таймаут одного запроса к реранкеру, секунды | `10` |
| `RERANKER_MAX_CONSECUTIVE_FAILURES` | Число подряд идущих ошибок до временного отключения реранкера | `3` |
| `RERANKER_COOLDOWN_SECONDS` | Пауза перед повторной попыткой после отключения реранкера | `30` |

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
| `EMBEDDING_PROVIDER` | Явно выбрать `remote` или `local`; если не задано, сервер выводит режим из варианта образа и настроек endpoint | *(авто)* |
| `EMBEDDING_PROVIDER_AMBIGUITY` | Поведение при неоднозначной legacy-конфигурации: ошибка или вывод режима | `error` |
| `EMBEDDING_MEMORY_BUDGET_MB` | Допустимая оценка памяти для локальной модели до загрузки весов | `4096` |
| `EMBEDDING_MEMORY_BUDGET_MODE` | `refuse`, `warn` или `off` при превышении бюджета | `refuse` |
| `EMBEDDING_MEMORY_SAMPLE_SEC` | Интервал измерения памяти локального embedding-процесса | `30` |
| `EMBEDDING_MEMORY_WARN_RATIO` | Доля бюджета, после которой состояние помечается предупреждением | `0.9` |
| `EMBEDDING_QUERY_PREFIX` | Переопределить префикс запроса для модели (профиль модели используется, если не задано) | *(из профиля модели)* |
| `EMBEDDING_DOCUMENT_PREFIX` | Переопределить префикс документа для модели | *(из профиля модели)* |

{% hint style="info" %}
**Поддержка OpenRouter:** Сервер автоматически определяет OpenRouter по URL и добавляет необходимые HTTP-заголовки (`HTTP-Referer`, `X-Title`).
{% endhint %}

### Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face; та же переменная выбирает модель и в API-режиме | `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` |

Старые имена `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` пока принимаются как совместимые алиасы, но новые конфигурации следует создавать с `EMBEDDING_*`.

### Плагины

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `PLUGIN_DIR` | Каталог, из которого читаются файлы плагинов | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | `true` — упавший derived-state хук (`on_source_file`, `on_chunk`, `on_metadata_object`) роняет сборку целиком; при `false` единица индексируется без изменений, а ошибка считается и попадает в сводку сборки | `false` |
| `PLUGIN_HOOK_TIMEOUT_SECONDS` | Бюджет времени одного вызова хука в секундах; превысивший его хук считается упавшим и call-scoped отключается до следующей перезагрузки каталога | `5.0` |

Каталог читается при каждом старте, флага включения нет: пустой или отсутствующий каталог ничего не загружает и ничего не меняет. Подробности: [Доработка MCP: система плагинов](../../sistema-pluginov/).

## Монтируемые тома

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/1C_Export/Report` | `/app/metadata` | Необязательный legacy-отчёт только для `METADATA_SOURCE=report` |
| `E:/1C_Export/Files` | `/app/code` | Выгрузка в файлы |
| `E:/bases/mcp_codemetadata` | `/app/chroma_db` | Векторная база данных |
| `./my-plugins` | `/app/plugins` | Свой каталог плагинов; монтирование поверх сохраняет плагины при `docker pull` |

## Примеры конфигураций

### Минимальная (CPU)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -v "E:/1C_Export/Files:/app/code" `
  comol/1c_code_metadata_mcp:latest-beta
```

### Рекомендуемая (LM Studio)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
```

### С OpenRouter (облачные эмбеддинги)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=https://openrouter.ai/api `
  -e EMBEDDING_API_KEY=YOUR_OPENROUTER_KEY `
  -e EMBEDDING_MODEL=qwen/qwen3-embedding-8b `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
```

### Облегчённый образ (light) с LM Studio

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:light-beta
```

### С реранкером и тюнингом поиска

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e ENABLE_RERANKER=true `
  -e BM25_ALPHA=0.5 `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
```

### С настройкой индексации

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e INDEX_CODE=true `
  -e INDEX_METADATA=true `
  -e INDEX_HELP=true `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
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

## Доработка

Сервер поддерживает [систему плагинов](../../sistema-pluginov/). В отличие от остальных серверов справочник лежит не в корне образа, а в `/app/src/plugin_api.py` — команда `docker run --rm comol/1c_code_metadata_mcp:latest-beta ls /app/plugin_api.py` даст ложное «нет плагинов». Проверять нужно так:

```powershell
docker run --rm comol/1c_code_metadata_mcp:latest-beta ls /app/src/plugin_api.py /app/plugins
```

Без плагинов поведение сервера настраивается форматом источника (`METADATA_SOURCE`, `SOURCE_FORMAT`, `CODE_PATH`, вложенные конфигурации) и параметрами поиска: `BM25_ALPHA`, `MIN_SCORE_THRESHOLD`, `ENABLE_RERANKER`, `VECTOR_PROFILE` и множителями выборки.
