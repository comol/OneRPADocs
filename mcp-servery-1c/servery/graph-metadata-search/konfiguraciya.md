# Конфигурация Graph Metadata Search

## Переменные окружения MCP сервера

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `NEO4J_URI` | URI подключения к Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь Neo4j | `neo4j` |
| `NEO4J_USER` | Устаревший алиас `NEO4J_USERNAME`. Если заданы обе переменные, приоритет у `NEO4J_USERNAME`, а выбор пишется в лог | — |
| `NEO4J_PASSWORD` | Пароль Neo4j. Значения по умолчанию нет: в образе пароль не зашит, и запуск без этой переменной завершается ошибкой аутентификации Neo4j | Обязательно |

{% hint style="info" %}
Отдельный текстовый отчёт больше не обязателен. При `METADATA_SOURCE=auto` сервер использует его, если он есть, а иначе синтезирует совместимый отчёт из Designer XML-выгрузки в `CODE_EXPORT_PATH`.
{% endhint %}

### Источники метаданных и кода

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `METADATA_DIRECTORY` | Каталог с текстовым отчётом Конфигуратора. Может быть пустым при работе по Designer XML | `/app/metadata` |
| `METADATA_SOURCE` | `auto` — готовый `*.txt`, иначе синтез из Designer XML; `report` — требовать отчёт; `xml` — игнорировать отчёт и синтезировать из XML | `auto` |
| `GENERATED_REPORT_DIRECTORY` | Кэш синтезированного отчёта; не должен находиться внутри `CODE_EXPORT_PATH` | `INGESTION_STATE_DIRECTORY/generated-report` |
| `CODE_EXPORT_PATH` | Корень Designer XML-выгрузки или проекта 1C:EDT. Для Designer XML служит источником и BSL/форм, и синтеза отчёта | — |
| `METADATA_FILES` | Каталог дополнительных детальных файлов метаданных | — |

Синтез отчёта поддерживает Designer XML (каталог с `Configuration.xml`) и выполняется в фоновой стадии `metadata_ingest`. Для проекта 1C:EDT нужен готовый текстовый отчёт; сам формат EDT для остальных контуров включается через `SOURCE_FORMAT_ADAPTERS_ENABLED=true` и `SOURCE_FORMAT=edt`.

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `ASYNC_VECTOR_INDEXING` | Индексация векторов в фоне (неблокирующая) | `true` |
| `BACKGROUND_POST_INDEXING` | Пост-обработка индексов в фоне после старта | `true` |
| `INDEX_BATCH_SIZE` | Количество объектов, обрабатываемых за один пакет при создании векторного индекса | `512` |
| `GRAPH_FORM_XML_BATCH_SIZE` | Сколько управляемых форм лента `form_xml` обрабатывает одной пачкой. Одна пачка — это один запрос на поиск владельца, один запрос на проверку resume и одна транзакция записи вместо примерно 15 обращений к Neo4j на каждую форму. Значение `1` возвращает прежний путь «по одной форме». Допустимый диапазон `1..500`, значение вне диапазона отклоняется при старте | `50` |
| `GRAPH_FORM_XML_BATCH_MAX_ROWS` | Сколько строк накапливается до сброса пачки — вторая граница бюджета памяти, чтобы одна аномально большая форма не сделала транзакцию безразмерной. Минимум `1`, значение ниже отклоняется при старте | `20000` |
| `MAX_TOKENS_PER_BATCH` | Максимальное количество токенов в одном пакете запроса к API эмбеддингов | `28000` |
| `EMBEDDING_REQUEST_CONCURRENCY` | Количество параллельных запросов к API эмбеддингов | `6` |
| `EMBEDDING_MAX_TOKENS` | Максимальное количество токенов на один текст при генерации эмбеддингов. Определяется автоматически по модели, но можно переопределить | *(авто)* |
| `EMBEDDING_CHUNK_TARGET_TOKENS` | Целевой размер чанка при разбивке длинных текстов | *(авто)* |
| `EMBEDDING_CHUNK_OVERLAP_TOKENS` | Перекрытие чанков при разбивке длинных текстов | *(авто)* |

### Embedding модели

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API для генерации и LLM | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API для генерации и LLM | `lm-studio` |
| `OPENAI_MODEL` | LLM модель для генерации описаний | `gpt-5` |
| `OPENAI_TEMPERATURE` | Температура генерации (0–1). Для reasoning-моделей (o1, o3, gpt-5) игнорируется | `0.1` |
| `OPENAI_MAX_COMPLETION_TOKENS` | Максимальное количество токенов в ответе LLM | `2000` |
| `OPENAI_MODEL_IS_REASONING` | Принудительное указание, является ли модель «рассуждающей». Если не указано — определяется автоматически по имени модели (o1*, o3*, gpt-5*) | *(авто)* |
| `EMBEDDING_API_BASE` | Отдельный URL для API эмбеддингов (если отличается от LLM API) | — |
| `EMBEDDING_API_KEY` | Отдельный ключ для API эмбеддингов | — |
| `EMBEDDING_MODEL` | Модель для API эмбеддингов | `qwen/qwen3-embedding-8b` |
| `OPENAI_EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов | *(авто)* |
| `LOCAL_EMBEDDING_MODEL` | Локальная CPU модель (sentence-transformers). Совместимый алиас — `OFFLINE_EMBEDDING_MODEL` | `intfloat/multilingual-e5-small` |
| `EMBEDDING_ALLOW_OFFLINE_FALLBACK` | Разрешить автопереход на локальную модель при недоступности API | `true` |

Старые имена `OPENAI_EMBEDDING_API_BASE`, `OPENAI_EMBEDDING_API_KEY` и `OPENAI_EMBEDDING_MODEL` остаются совместимыми алиасами.

### Шаблонный режим

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `TEMPLATE_MODE_ENABLED` | Включить шаблонный режим — JSON-запросы с мгновенными ответами без LLM | `true` |
| `TEMPLATE_MODE_ONLY` | Только шаблонные запросы, LLM не используется. Требует `TEMPLATE_MODE_ENABLED=true` | `false` |

### Поиск по коду

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ENABLE_CODE_SEARCH` | Включить поиск по BSL-файлам | `true` |
| `CODE_SEARCH_MAX_FILE_SIZE` | Максимальный размер BSL-файла (байт) для индексации | `50000` |

### Семантический поиск

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ENABLE_BUSINESS_SEARCH` | Включить семантический поиск по бизнес-описаниям | `true` |
| `CALCULATE_BUSINESS_INFO` | Генерировать AI бизнес-описания для объектов метаданных | `false` |
| `BUSINESS_INFO_MAX_TOKENS` | Максимум токенов контекста для бизнес-описаний | `4000` |
| `BUSINESS_INFO_RETRY_COUNT` | Количество повторных попыток при ошибках API | `3` |
| `BUSINESS_INFO_THREADS` | Количество параллельных воркеров генерации бизнес-описаний | `10` |
| `ENABLE_METADATA_DESCRIPTION_EMBEDDING` | Генерировать эмбеддинги для описательных полей метаданных (Синоним, Комментарий, Описание, справка) | `true` |

{% hint style="warning" %}
**Каталог исходников монтируется на запись — без `:ro`.** При `CALCULATE_BUSINESS_INFO=true` описание пишется в `business_info.html` рядом с объектом метаданных, и в Neo4j оно попадает только после успешной записи файла. Если `/app/code` смонтирован read-only, каждый объект даёт `[Errno 30] Read-only file system: /app/code/.../business_info.html`: описание не сохраняется ни на диск, ни в граф, а токены LLM тратятся впустую. Контейнер при этом остаётся `healthy`, структурный граф и векторный индекс описаний строятся как обычно — ошибку легко не заметить.

Проверить правом на запись, а не глазами: `docker exec 1c_graph_metadata sh -c "touch /app/code/.rwcheck && rm /app/code/.rwcheck && echo OK"` — на корректном монтировании команда печатает `OK`, на read-only падает с `Read-only file system`. Остальные серверы (CodeMetadataSearch, SyntaxCheck, 1CCodeChecker) читают тот же каталог и монтируют его `:ro` — это нормально, требование к записи есть только у GraphMetadataSearch.
{% endhint %}


### BSL-граф (Module / Routine / CALLS)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `CODE_EXPORT_PATH` | Путь к Designer XML-выгрузке или проекту 1C:EDT (тот же источник, что описан выше) | — |
| `LOAD_BSL_SIGNATURES` | Загружать сигнатуры BSL-кода в граф — создавать ноды Module и Routine с графом вызовов CALLS | `true` |
| `ENABLE_ROUTINE_EMBEDDINGS` | Генерировать эмбеддинги для процедур/функций. Требует `LOAD_BSL_SIGNATURES=true`. Индексация в фоновом потоке | `true` |
| `BSL_READ_PREFIX_BYTES` | Сколько байт файла читается для быстрой классификации BSL | `4096` |

### Дополнительные данные из XML-выгрузки

Все опции ниже требуют заполнения `CODE_EXPORT_PATH`.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LOAD_FORMS_FROM_XML` | Загружать структуру управляемых форм из `Ext/Form.xml` (FormControl, FormEvent, FormAttribute) | `false` |
| `LOAD_ORDINARY_FORMS` | Загружать структуру обычных форм (отдельная модель узлов от управляемых) | `true` |
| `LOAD_EVENT_SUBSCRIPTIONS` | Загружать подписки на события из `EventSubscriptions/*.xml` | `false` |
| `LOAD_PREDEFINED_VALUES` | Загружать предопределённые элементы из `*/Predefined.xml` | `false` |
| `LOAD_ROLE_RIGHTS` | Загружать права ролей из `Roles/*/Ext/Rights.xml` | `false` |
| `LOAD_HELP_FROM_HTML` | Загружать справку объектов из `*/Help/ru.html` | `false` |
| `LOAD_DCS_TEMPLATES` | Загружать схемы компоновки данных (для `get_report_dcs_lineage`) | `false` |

### Поддержка расширений

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `EXTENSION_NAME` | Имя расширения. Если задано, все загружаемые объекты получают `origin="extension"` | — |
| `EXTENSION_BASE_PROJECT` | Имя базового проекта (`PROJECT_NAME` базовой конфигурации) для построения связей EXTENDS/OVERRIDES | — |
| `EXTENSION_BASE_PROJECT_ID` | Зарегистрированный `PROJECT_ID` базовой конфигурации. Нужен, когда идентификатор проекта отличается от отображаемого имени в `EXTENSION_BASE_PROJECT`. Разрешается только в текущем `MCP_NAMESPACE`. Если не задан, идентификатором считается само значение `EXTENSION_BASE_PROJECT` | — |
| `EXTENSION_APPLY_ORDER` | Порядок применения слоя расширения при резолве эффективной сущности | `1` |

### Массовая загрузка расширений из каталога

Один прогон с `EXTENSION_NAME` загружает **одно** расширение. В 1С:ERP к конфигурации может быть применено до 200 расширений-патчей, и настраивать 200 прогонов вручную — не рабочий процесс. Поэтому прогон можно направить на каталог массовой выгрузки (`/DumpConfigToFiles … -AllExtensions`): каждая найденная в нём выгрузка расширения становится слоем одного и того же проекта.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `EXTENSION_CATALOG_ENABLED` | Распознавать выгрузки расширений внутри каталога и загружать их слоями за один прогон | `false` |
| `EXTENSIONS_PATH` | Где лежат выгрузки расширений. Пусто — подкаталоги каталога выгрузки (`CODE_EXPORT_PATH`, затем `METADATA_FILES`), куда их кладёт массовая выгрузка | *(пусто)* |
| `EXTENSION_ORDER_MANIFEST` | JSON со списком имён расширений в порядке применения (список или объект с полем `order`). Пусто — ищется `extensions_order.json` в каталоге | *(пусто)* |
| `EXTENSION_CATALOG_SYNC` | Синхронизировать набор слоёв с каталогом: исчезнувшая выгрузка — слой удаляется, новая — добавляется, изменённая — обновляется, неизменённая — не переингестируется. Базовый слой не удаляется никогда | `true` |

{% hint style="info" %}
**Порядок применения в пакетном режиме определяется в два уровня.** Между группами — по назначению расширения из его собственной выгрузки (Исправление → Адаптация → Дополнение, как применяет платформа); это не настройка оператора. Внутри группы — по манифесту, а без манифеста расширения одного назначения упорядочиваются по имени, и прогон явно сообщает, что внутригрупповой порядок **предположен**.
{% endhint %}

{% hint style="warning" %}
Режим выключен по умолчанию: распознавание читает каталоги, и проект, который загружает одно расширение объявленным способом, должен продолжать делать ровно это. Одиночный ручной режим (`EXTENSION_NAME` на прогон) сохраняется.
{% endhint %}

Расширение в 1С не применяется к другому расширению, поэтому слои по-прежнему прикрепляются только к базовому (`EXTENDS`/`OVERRIDES` ведут от расширения к базе), связей «расширение → расширение» нет.

### Профиль инструментов и scope

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_TOOL_PROFILE` | Какой набор инструментов публикуется: `admin` (всё) или `read-only` (без управления проектами) | `admin` |
| `MCP_NAMESPACE` | Namespace установки, в котором регистрируются графовые проекты | `default` |
| `ADMIN_TOKEN` | Токен для административных HTTP-эндпоинтов веб-интерфейса | — |
| `GRAPH_SCOPE_ENFORCED` | Записан ли scope в самой базе. Включать только на базе, пересобранной по новой схеме идентификаторов: данные, загруженные раньше, свойств scope не содержат. Вместе с миграционным окном задаёт строгость `project_id` — см. врезку ниже | `false` |
| `GRAPH_SCOPE_MIGRATION_WINDOW` | Окно миграции: вызов без `project_id` обслуживается единственным зарегистрированным проектом namespace, ответ помечается `deprecated` | `false` |
| `GRAPH_ONLY` | Режим только чтения графа — без загрузки и обогащения данных | `false` |

{% hint style="info" %}
**Строгость `project_id` определяется состоянием базы, а не только флагом контракта.** Пока `GRAPH_SCOPE_ENFORCED=false`, в графе нет ни одного свойства scope и ни один путь чтения им не ограничен: вызов с `project_id` и вызов без него читают одни и те же строки. Поэтому вызов без `project_id` не отклоняется, а обслуживается — единственным доступным проектом, а при пустом реестре legacy-scope из `PROJECT_NAME` (или `<unscoped>`, если и он не задан). Ответ помечается `deprecated` и несёт предупреждение, что результаты не ограничены проектом.

Отклоняется только вызов без `project_id` в namespace, где доступно несколько проектов: там выбор реальный, и его показывает `list_graph_projects`. При `GRAPH_SCOPE_ENFORCED=true` и закрытом `GRAPH_SCOPE_MIGRATION_WINDOW=false` `project_id` строго обязателен. При открытом окне единственный проект временно подставляется с `deprecated`; несколько проектов всегда требуют явного выбора.

Реестр графовых проектов живёт в памяти процесса и после рестарта пуст, поэтому свежий процесс на настройках по умолчанию читает граф без `project_id`, а не отказывает в доступе к данным, которые он всё равно не изолирует.
{% endhint %}

### Лимиты ответов графовых инструментов

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `GRAPH_MAX_ITEMS` | Жёсткий предел элементов в ответе (`max_items` не может его превысить) | `200` |
| `GRAPH_MAX_NODES` | Предел узлов в компактном графовом ответе | `500` |
| `GRAPH_MAX_EDGES` | Предел связей в компактном графовом ответе | `1000` |
| `GRAPH_MAX_CHARS` | Предел размера ответа в символах | `60000` |
| `GRAPH_TOOL_TIMEOUT_SECONDS` | Таймаут выполнения инструмента | `300` |
| `GRAPH_ENTITY_GROUP_LIMIT` | Предел размера группы связей в `explain_graph_entity` | `25` |
| `GRAPH_ENTITY_MAX_CANDIDATES` | Предел кандидатов при резолве сущности | `20` |
| `GRAPH_PATH_MAX_DEPTH` | Максимальная глубина поиска путей (`find_graph_path`) | `6` |
| `GRAPH_PATH_MAX_PATHS` | Количество возвращаемых кратчайших путей | `3` |
| `GRAPH_PATH_FANOUT_LIMIT` | Предел ветвления на шаг при поиске путей | `200` |
| `GRAPH_PATH_MAX_FRONTIER` | Предел фронта обхода при поиске путей | `2000` |
| `GRAPH_IMPACT_MAX_DEPTH` | Максимальная глубина `affected_subgraph` / `trace_impact` | `4` |
| `GRAPH_IMPACT_MAX_PATHS` | Предел путей-обоснований в анализе влияния | `500` |
| `GRAPH_IMPACT_FANOUT_LIMIT` | Предел ветвления на шаг в анализе влияния | `200` |
| `GRAPH_COMPARISON_MAX_ENTITIES` | Предел сущностей при сравнении scope (`compare_graph_scope`) | `10000` |
| `GRAPH_SYNOPSIS_MAX_ITEMS` | Предел элементов в кратком обзоре ответа | `5` |

### Тонкая настройка поиска и ранжирования

Значения подобраны под типовые конфигурации; менять их стоит только по результатам замеров.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `VECTOR_RETRIEVAL_STRATEGY` | Как фильтруется векторная выборка: `auto` — предфильтрация внутри индекса, если сервер и индекс это поддерживают (Neo4j 2026.01+), иначе ограниченный overfetch; `native_prefilter` — требовать предфильтрацию (при невозможности запрос обслуживается overfetch, а понижение пишется в журнал); `bounded_overfetch` — прежнее поведение (откат) | `auto` |
| `VECTOR_OVERFETCH_MULTIPLIER` | Множитель выборки векторного маршрута | `4` |
| `VECTOR_OVERFETCH_HARD_CAP` | Жёсткий предел векторной выборки | `1000` |
| `UNFILTERED_OVERSAMPLING_MULTIPLIER` | Множитель выборки без фильтра | `3` |
| `FILTERED_OVERSAMPLING_MULTIPLIER` | Множитель выборки при заданном фильтре | `10` |
| `FILTERED_OVERSAMPLING_RETRY_MULTIPLIER` | Во сколько раз расширяется выборка на повторе, если фильтр отсёк слишком много | `3` |
| `FILTERED_OVERSAMPLING_MAX_RETRIES` | Максимум таких повторов | `3` |
| `MAX_CANDIDATE_POOL` | Предел общего пула кандидатов до ранжирования | `200` |
| `RRF_K` | Константа reciprocal rank fusion | `60` |
| `CODE_SEARCH_RRF_W_VEC` / `CODE_SEARCH_RRF_W_FT` | Веса векторной и полнотекстовой дорожек при поиске по коду | `0.6` / `0.4` |
| `BUSINESS_SEARCH_RRF_W_VEC` / `BUSINESS_SEARCH_RRF_W_FT` | То же для поиска по бизнес-описаниям | `0.6` / `0.4` |
| `ATTRIBUTE_SEARCH_RRF_WEIGHT` | Вес маршрута поиска по реквизитам | `0.3` |
| `ENABLE_ATTRIBUTE_SEARCH` | Искать по реквизитам объектов | `true` |
| `CODE_SEARCH_MAX_PER_MODULE` | Максимум результатов из одного модуля | `1` |
| `CODE_SEARCH_DEDUP_OVERFETCH` | Запас выборки на дедупликацию по модулям | `10` |
| `CHANNEL_ADMISSION_MIN_SCORE` | Минимальная средняя оценка в топ-K, при которой маршрут допускается в общий пул. `0.0` отключает отсечение | `0.1` |
| `CHANNEL_ADMISSION_TOP_K_CHECK` | Сколько верхних результатов маршрута усредняется для этой проверки | `3` |
| `SEMANTIC_GATING_MIN_SCORE_VECTOR` | Порог семантического маршрута до реранка | `0.3` |
| `SEMANTIC_GATING_MIN_SCORE_RERANKED` | Порог после реранка | `0.5` |
| `SEMANTIC_FALLBACK_MIN_RESULTS` | Сколько результатов должно остаться, чтобы не включать резервный маршрут | `3` |
| `SEMANTIC_MERGE_W_DESCRIPTION` / `SEMANTIC_MERGE_W_BUSINESS` / `SEMANTIC_MERGE_W_TRANSLITERATED` | Веса описательного, бизнес- и транслитерированного представлений при слиянии | `1.0` / `1.0` / `0.7` |
| `MIXED_MERGE_W_SEMANTIC` / `MIXED_MERGE_W_LEXICAL` / `MIXED_MERGE_W_STRUCTURAL` | Веса семантического, лексического и структурного маршрутов | `1.0` / `0.4` / `1.0` |
| `DESCRIPTION_FT_BOOST_NAME` / `_SYNONYM` / `_COMMENT` / `_DESCRIPTION` / `_HELP` | Веса полей в полнотекстовом маршруте | `4.0` / `3.0` / `1.5` / `1.0` / `1.0` |
| `CHUNK_EXCERPTS_PER_OBJECT` | Сколько фрагментов одного объекта попадает в ответ | `3` |
| `CHUNK_OBJECT_DUAL_HIT_BOOST` | Прибавка к оценке, если объект найден и на уровне объекта, и на уровне фрагментов. `0.0` отключает | `0.05` |
| `CHUNK_EVIDENCE_WEIGHT` | Прибавка за каждый дополнительный совпавший фрагмент объекта | `0.02` |
| `MMR_LAMBDA` | Баланс релевантности и разнообразия (1.0 — только релевантность) | `0.7` |
| `MMR_USE_EMBEDDINGS` | Считать разнообразие по эмбеддингам | `true` |
| `POST_MERGE_RERANK_TOP_N` | Сколько верхних результатов уходит в реранк после слияния | `20` |
| `RERANK_MAX_CANDIDATES` | Максимум кандидатов в одном вызове реранкера; хвост сохраняет исходный порядок | `200` |
| `PROJECT_AFFINITY_BOOST` | Прибавка результатам «своего» проекта | `0.0` |
| `ENABLE_GRAPH_EXPANSION` | Расширять выдачу соседями по графу | `true` |
| `GRAPH_EXPANSION_TOP_N` | Сколько результатов расширяется соседями | `3` |
| `GRAPH_EXPANSION_DECAY` | Затухание оценки на шаг расширения | `0.5` |

### Ответы с участием LLM (`answer_metadata_question`)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `QA_MAX_OBJECTS` | Сколько объектов попадает в контекст ответа | `5` |
| `QA_MAX_TOKENS` | Предел токенов ответа | `4000` |
| `QA_TEMPERATURE` | Температура модели | `0.3` |
| `QA_INCLUDE_CODE` | Включать код в контекст | `true` |
| `SUMMARIZE` | Суммаризировать результаты поиска через LLM | `false` |

### Плагины

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `GRAPH_PLUGINS_ENABLED` | Загружать Python-плагины из каталога плагинов | `true` |
| `GRAPH_PLUGINS_DIRECTORY` | Каталог плагинов; относительный путь считается от рабочего каталога `/app` | `plugins` |
| `GRAPH_PLUGIN_STRICT_BUILD` | Останавливать построение нового поколения при ошибке derived-state hook вместо пропуска проблемной единицы | `false` |
| `GRAPH_PLUGIN_HOOK_TIMEOUT_SECONDS` | Бюджет времени одного hook; `0` отключает контроль | `5.0` |

Для контейнера обычно монтируют каталог хоста в `/app/plugins`. `list_plugins` доступен для диагностики в обоих профилях, а изменяющий состояние `reload_plugins(operation_id)` — только в профиле `admin`. После перезагрузки call-scoped hooks действуют со следующего вызова, а derived-state hooks — со следующего построения индекса.

Плагин — **один Python-файл** в каталоге плагинов. Объявлены восемь hooks: `on_startup`, `on_request`, `on_result`, `on_search_candidates` (в рамках вызова) и `on_source_unit`, `on_metadata_object`, `on_routine`, `on_embedding_document` (формируют граф, полнотекстовый и векторный маршруты) — плюс три таблицы: `ALIASES`, `TOOL_PRESETS` и `CYPHER_TEMPLATES` (дополнительные read-only Cypher-шаблоны для `run_graph_cypher_template`). Полный контракт лежит в образе: `/app/plugin_api.py`, `/app/plugins/AGENTS.md`, `/app/plugins/example.py`.

```powershell
# Проверить плагин без Neo4j, метаданных, поколения и лицензии
docker run --rm -v "E:/plugins/mcp_graph/10-facts.py:/tmp/my_plugin.py" `
  comol/1c_graph_metadata:latest-beta python run.py plugin-dry-run /tmp/my_plugin.py
```

{% hint style="warning" %}
Без `GRAPH_PLUGINS_ENABLED=true` каталог не читается вообще, а `reload_plugins` отвечает, что подсистема отключена.
{% endhint %}

Правка любого derived-state hook делает сохранённое состояние проекта устаревшим и вызывает пересборку; call-scoped hooks и таблицы обходятся перезагрузкой. Подробно: [Доработка MCP: система плагинов](../../sistema-pluginov/) и [справочник хуков Graph Metadata Search](../../sistema-pluginov/spravochnik-hukov.md#graph-metadata-search).

### Поколения графа и загрузка данных

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `INGESTION_COORDINATOR_ENABLED` | Координатор загрузки с фазами, лизом и чекпоинтами | `false` |
| `INGESTION_LEASE_TTL_SECONDS` | Время жизни лиза загрузчика. Живой писатель продлевает лиз heartbeat'ом каждые TTL/3, штатная остановка (SIGTERM) освобождает лиз сразу, поэтому срок платит только аварийное завершение: при большом TTL контейнер, убитый по OOM, видел после перезапуска живой лиз мёртвого писателя и помечал задачи индексации как skipped | `120` |
| `INGESTION_CHECKPOINT_BATCH_SIZE` | Размер пакета между чекпоинтами | `500` |
| `INGESTION_CHECKPOINT_INTERVAL_SECONDS` | Интервал записи чекпоинтов | `30` |
| `INGESTION_TRACKER_BACKEND` | Где хранится состояние загрузки: `json` (без БД) или `neo4j` | `json` |
| `INGESTION_STATE_DIRECTORY` | Каталог для состояния при бэкенде `json` | — |
| `GRAPH_STAGING_VALIDATION_ENABLED` | Проверять инварианты staging-поколения перед promote | `false` |
| `GRAPH_STAGING_VALIDATION_MODE` | Что делает нарушение: `blocking` (отказ в promote) или `report` (promote с отчётом) | `blocking` |
| `GRAPH_STAGING_VALIDATION_SAMPLE_LIMIT` | Количество примеров нарушений в отчёте | `5` |
| `GRAPH_STAGING_COVERAGE_EXPECTATIONS` | Ожидания по покрытию для валидации staging | — |
| `GRAPH_DELETE_VISIBILITY_ENABLED` | Контроль видимости удалённых сущностей между поколениями | `true` |
| `GRAPH_DELETE_VISIBILITY_MODE` | Режим контроля видимости удалений | `blocking` |
| `GRAPH_GENERATION_FENCE_ENABLED` | Фенс поколений — защита от чтения из устаревшего поколения | `true` |
| `GRAPH_GENERATION_FENCE_ATTEMPTS` | Количество повторов при срабатывании фенса | `4` |
| `REFERENCE_EVIDENCE_ENABLED` | Сохранять evidence для `explain_graph_evidence` / `explain_path` | `false` |
| `REFERENCE_EVIDENCE_RETENTION_DAYS` | Срок хранения evidence в днях | `90` |
| `REFERENCE_EVIDENCE_RETENTION_GENERATIONS` | Сколько поколений evidence хранить | `3` |
| `SOURCE_UNIT_MANIFEST_ENABLED` | Манифест единиц исходников для инкрементальной пересборки | `false` |
| `SOURCE_UNIT_MANIFEST_DIRECTORY` | Каталог хранения манифеста | — |
| `SOURCE_UNIT_IGNORE_PATTERNS` | Шаблоны исключения файлов из манифеста | — |
| `SOURCE_UNIT_SCAN_PREFIX_BYTES` | Сколько байт файла читается при сканировании | `4096` |
| `SOURCE_FORMAT_ADAPTERS_ENABLED` | Адаптеры форматов выгрузки (Конфигуратор / EDT) | `false` |
| `SOURCE_FORMAT` | Явное указание формата выгрузки | *(авто)* |

### Дополнительные

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_HOST` | Хост MCP-сервера | `0.0.0.0` |
| `MCP_PORT` | Порт MCP | `8006` |
| `MCP_PATH` | URL-путь для MCP эндпоинта | `/mcp` |
| `MCP_USE_SSE` | SSE транспорт (для legacy клиентов) | `false` |
| `NEO4J_DATABASE` | Имя базы Neo4j | `neo4j` |
| `NEO4J_PARALLEL_WRITE_WORKERS` | Количество параллельных воркеров записи в Neo4j. Допустимый диапазон `1..16`, значение вне диапазона отклоняется при старте. Значения выше `1` приводили к взаимным блокировкам BSL-писателей на `NODE_RELATIONSHIP_GROUP_DELETE` при первой индексации — повышайте только по результатам замеров на своих данных | `1` |
| `NEO4J_WRITE_BATCH_SIZE` | Размер пакета записи в Neo4j | `1000` |
| `NEO4J_WRITE_RETRIES` | Количество повторов при ошибке записи | `5` |
| `NEO4J_WRITE_RETRY_DELAY_S` | Пауза перед первым повтором; далее удваивается | `1.0` |
| `NEO4J_WRITE_RETRY_MAX_DELAY_S` | Предел паузы между повторами | `30.0` |
| `NEO4J_EXPECTED_PAGECACHE_MB` | Ожидаемый размер page cache сервера Neo4j (МиБ). При старте сравнивается с фактическим и при меньшем значении пишется предупреждение. Никогда не применяется принудительно: если настройку не удалось прочитать, проверка молча считается успешной | — |
| `PROJECT_NAME` | Название проекта (для логов, интерфейса и мультипроектности) | `1C Metadata Project` |
| `DEBUG` | Режим отладки — дополнительные логи | `false` |

## Переменные Neo4j

| Переменная | Описание | Пример |
|------------|----------|--------|
| `NEO4J_AUTH` | Логин/пароль | `neo4j/password123` |
| `NEO4J_server_memory_heap_max__size` | Макс. память | `1g` |

## Монтируемые тома

### MCP сервер

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/1C_Export/Report` | `/app/metadata` | Отчёт по метаданным |
| `E:/1C_Export/Files` | `/app/metadata_files` | Файлы кода (BSL, XML, справка) |

### Neo4j

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/bases/mcp_graph/neo4j` | `/data` | Данные Neo4j |

## Полный docker-compose.yml

```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j@sha256:7d8d70f78c0c55830a162e6ae212799649b0cfd349dbfe413aab8124f7cabf1b
    container_name: neo4j
    restart: unless-stopped
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      - NEO4J_AUTH=neo4j/password123
      - NEO4J_server_memory_heap_initial__size=512m
      - NEO4J_server_memory_heap_max__size=1g
      - NEO4J_server_memory_pagecache_size=512m
    volumes:
      - E:/bases/mcp_graph/neo4j:/data
    deploy:
      resources:
        limits:
          memory: 2G
    healthcheck:
      test: ["CMD-SHELL", "wget --spider localhost:7474 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  mcp-app:
    image: comol/1c_graph_metadata:latest-beta
    container_name: 1c_graph_metadata
    restart: unless-stopped
    ports:
      - "8006:8006"
    environment:
      # === Обязательные ===
      - LICENSE_KEY=YOUR_LICENSE_KEY
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - RESET_DATABASE=false
      # === Embedding / LLM ===
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=gpt-5
      - EMBEDDING_MODEL=Qwen3-Embedding-4B
      # === Шаблонный режим ===
      - TEMPLATE_MODE_ENABLED=true
      # === BSL-граф ===
      - CODE_EXPORT_PATH=/app/metadata_files
      - LOAD_BSL_SIGNATURES=true
      - ENABLE_ROUTINE_EMBEDDINGS=true
      # === XML-данные (опционально) ===
      # - LOAD_FORMS_FROM_XML=true
      # - LOAD_EVENT_SUBSCRIPTIONS=true
      # - LOAD_PREDEFINED_VALUES=true
      # - LOAD_ROLE_RIGHTS=true
      # - LOAD_HELP_FROM_HTML=true
      # === Поиск ===
      - ENABLE_BUSINESS_SEARCH=true
      # - ENABLE_METADATA_DESCRIPTION_EMBEDDING=true
      # === Проект ===
      - MCP_PORT=8006
      - PROJECT_NAME=МояКонфигурация
      - METADATA_SOURCE=xml
      - CODE_EXPORT_PATH=/app/code
      - GENERATED_REPORT_DIRECTORY=/app/data/generated-report
    volumes:
      - E:/1C_Export/Files:/app/code
      - E:/bases/mcp_graph/app:/app/data
    deploy:
      resources:
        limits:
          memory: 1G
    depends_on:
      neo4j:
        condition: service_healthy
```

## Пример: загрузка расширения

Для загрузки расширения запустите отдельный экземпляр сервера с указанием `EXTENSION_NAME` и `EXTENSION_BASE_PROJECT`:

```yaml
  mcp-extension:
    image: comol/1c_graph_metadata:latest-beta
    container_name: 1c_graph_metadata_ext
    environment:
      - LICENSE_KEY=YOUR_LICENSE_KEY
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - RESET_DATABASE=true
      - EXTENSION_NAME=МоёРасширение
      - EXTENSION_BASE_PROJECT=МояКонфигурация
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - EMBEDDING_MODEL=Qwen3-Embedding-4B
      - METADATA_SOURCE=xml
      - CODE_EXPORT_PATH=/app/code
      - LOAD_BSL_SIGNATURES=true
    volumes:
      - E:/1C_Export_Extension/Files:/app/code
    depends_on:
      neo4j:
        condition: service_healthy
```

После загрузки расширения через основной MCP-сервер станут доступны инструменты `compare_base_and_extension` и шаблонные операции для работы с расширениями.

## Пример: массовая загрузка всех расширений

Если конфигурация выгружена вместе с расширениями (`/DumpConfigToFiles … -AllExtensions`), все они загружаются одним прогоном:

```yaml
  mcp-extensions:
    image: comol/1c_graph_metadata:latest-beta
    container_name: 1c_graph_metadata_ext
    environment:
      - LICENSE_KEY=YOUR_LICENSE_KEY
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - METADATA_SOURCE=xml
      - CODE_EXPORT_PATH=/app/code
      - EXTENSION_BASE_PROJECT=МояКонфигурация
      - EXTENSION_CATALOG_ENABLED=true
      - EXTENSION_CATALOG_SYNC=true
      - LOAD_BSL_SIGNATURES=true
    volumes:
      - E:/1C_Export/Files:/app/code
    depends_on:
      neo4j:
        condition: service_healthy
```

При `EXTENSION_CATALOG_SYNC=true` повторный прогон приводит граф в соответствие с каталогом: патч, вошедший в релиз конфигурации и исчезнувший из выгрузки, исчезает и из графа — вместо того чтобы отвечать за информационную базу, в которой его больше нет.

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-graph-metadata-mcp": {
      "url": "http://localhost:8006/mcp",
      "connection_id": "1c_graph_service_001"
    }
  }
}
```
