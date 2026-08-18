# Конфигурация Graph Metadata Search

## Переменные окружения MCP сервера

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `NEO4J_URI` | URI подключения к Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь Neo4j | `neo4j` |
| `NEO4J_PASSWORD` | Пароль Neo4j | `password123` |
| `METADATA_DIRECTORY` | Путь к метаданным (отчёт из конфигуратора) | `/app/metadata` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `false` |
| `ASYNC_VECTOR_INDEXING` | Индексация векторов в фоне (неблокирующая) | `true` |
| `BACKGROUND_POST_INDEXING` | Пост-обработка индексов в фоне после старта | `true` |
| `INDEX_BATCH_SIZE` | Количество объектов, обрабатываемых за один пакет при создании векторного индекса | `512` |
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
| `EMBEDDING_MODEL` | Модель для API эмбеддингов | `text-embedding-ada-002` |
| `OPENAI_EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов | *(авто)* |
| `LOCAL_EMBEDDING_MODEL` | Локальная CPU модель (sentence-transformers) | `intfloat/multilingual-e5-small` |
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

### BSL-граф (Module / Routine / CALLS)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `CODE_EXPORT_PATH` | Путь к XML-выгрузке конфигурации в файлы (Конфигуратор → Выгрузить конфигурацию в файлы) | — |
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
| `EXTENSION_APPLY_ORDER` | Порядок применения слоя расширения при резолве эффективной сущности | `1` |

### Профиль инструментов и scope

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_TOOL_PROFILE` | Какой набор инструментов публикуется: `admin` (всё) или `read-only` (без управления проектами) | `admin` |
| `MCP_NAMESPACE` | Namespace установки, в котором регистрируются графовые проекты | `default` |
| `ADMIN_TOKEN` | Токен для административных HTTP-эндпоинтов веб-интерфейса | — |
| `GRAPH_SCOPE_ENFORCED` | Требовать явного `project_id` во всех project-scoped вызовах | `false` |
| `GRAPH_SCOPE_MIGRATION_WINDOW` | Окно миграции: разрешить вызовы без `project_id`, когда зарегистрирован ровно один проект | `false` |
| `GRAPH_ONLY` | Режим только чтения графа — без загрузки и обогащения данных | `false` |

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

### Поколения графа и загрузка данных

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `INGESTION_COORDINATOR_ENABLED` | Координатор загрузки с фазами, лизом и чекпоинтами | `false` |
| `INGESTION_LEASE_TTL_SECONDS` | Время жизни лиза загрузчика | `900` |
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
| `NEO4J_PARALLEL_WRITE_WORKERS` | Количество параллельных воркеров записи в Neo4j | `4` |
| `NEO4J_WRITE_BATCH_SIZE` | Размер пакета записи в Neo4j | `1000` |
| `NEO4J_WRITE_RETRIES` | Количество повторов при ошибке записи | `5` |
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
    image: neo4j:latest
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
    image: comol/1c_graph_metadata:latest
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
    volumes:
      - E:/1C_Export/Report:/app/metadata
      - E:/1C_Export/Files:/app/metadata_files
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
    image: comol/1c_graph_metadata:latest
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
      - CODE_EXPORT_PATH=/app/metadata_files
      - LOAD_BSL_SIGNATURES=true
    volumes:
      - E:/1C_Export_Extension/Report:/app/metadata
      - E:/1C_Export_Extension/Files:/app/metadata_files
    depends_on:
      neo4j:
        condition: service_healthy
```

После загрузки расширения через основной MCP-сервер станут доступны инструменты `compare_base_and_extension` и шаблонные операции для работы с расширениями.

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
