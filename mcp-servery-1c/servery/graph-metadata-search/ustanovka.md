# Установка Graph Metadata Search

Graph Metadata Search требует два сервиса: Neo4j и MCP сервер. Рекомендуется использовать docker-compose.

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен (рекомендуется) или OpenAI API доступен
3. [Подготовлены данные](podgotovka-dannyh.md) из Конфигуратора

## Создание структуры папок

```powershell
New-Item -ItemType Directory -Force -Path @(
    "E:\bases\mcp_graph",
    "E:\1C_Export\Files"
)
```

## Создание docker-compose.yml

Создайте файл `docker-compose.yml`:

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
    volumes:
      - E:/bases/mcp_graph/neo4j:/data
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
      - LICENSE_KEY=YOUR_LICENSE_KEY
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - METADATA_SOURCE=xml
      - CODE_EXPORT_PATH=/app/code
      - GENERATED_REPORT_DIRECTORY=/app/data/generated-report
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - EMBEDDING_MODEL=Qwen3-Embedding-4B
      - TEMPLATE_MODE_ENABLED=true
      - LOAD_BSL_SIGNATURES=true
    volumes:
      - E:/1C_Export/Files:/app/code:ro
      - E:/bases/mcp_graph/app:/app/data
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request,sys; sys.exit(0 if urllib.request.urlopen('http://localhost:8006/healthz', timeout=10).status == 200 else 1)"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 180s
    depends_on:
      neo4j:
        condition: service_healthy
```

## Запуск

```powershell
# Перейти в папку с docker-compose.yml
cd E:\mcp_setup

# Запустить сервисы
docker-compose up -d

# Просмотр логов
docker-compose logs -f
```

## Проверка работы

### Статус сервисов

```powershell
docker-compose ps
```

### Neo4j Browser

Откройте в браузере: `http://localhost:7474`

Логин: `neo4j`
Пароль: `password123`

### MCP сервер — health и readiness

```powershell
curl http://localhost:8006/healthz
```

Дешёвая liveness-проба: отвечает сразу, как только процесс жив и создан драйвер Neo4j, без запросов к графу. Именно её использует healthcheck в `docker-compose.yml`. Готовность проверяется отдельно:

```powershell
curl http://localhost:8006/readyz
```

В более новом исходном дереве есть алиасы `/health` и `/ready`, однако опубликованная beta подтверждённо предоставляет `/healthz` и `/readyz`, поэтому автоматические проверки используют именно их. Статус фоновых задач и поколений читайте MCP-инструментами `get_indexing_status` и `get_graph_project_status`: серверный режим не публикует старые веб-маршруты `/status`, `/search/*` и `/docs`.

## Конвейер запуска (Startup Pipeline)

При запуске система выполняет этапы в определённом порядке. MCP-сервер становится доступным сразу после загрузки базовых метаданных, не дожидаясь обогащения и векторной индексации.

### Этапы запуска

1. **Запуск Neo4j** (~30 сек)
2. **Подключение MCP сервера к Neo4j**
3. **Загрузка конфигураций** — парсинг метаданных, создание графа
4. **Загрузка BSL-графа** (если `LOAD_BSL_SIGNATURES=true`) — модули, процедуры, граф вызовов
5. **Загрузка XML-данных** (если включены) — подписки, предопределённые, роли, справка
6. **Загрузка форм XML** (если `LOAD_FORMS_FROM_XML=true`) — структура управляемых форм
7. **Запуск MCP-сервера** — принимает подключения
8. **Фоновая генерация бизнес-описаний** (если `CALCULATE_BUSINESS_INFO=true`)
9. **Фоновая векторная индексация** (после завершения бизнес-описаний)
10. **Фоновая индексация Routine-эмбеддингов** (если `ENABLE_ROUTINE_EMBEDDINGS=true`)

{% hint style="info" %}
Шаги 8–10 выполняются в фоне. MCP-инструменты доступны с шага 7. Семантический поиск будет полноценно работать после завершения векторной индексации. До этого момента инструменты вернут информативное сообщение о статусе.
{% endhint %}

### Устойчивость к ошибкам

Каждый этап индексации выполняется независимо. Ошибка в одном объекте или этапе не прерывает остальные:

- Ошибка в одном MetadataObject → остальные объекты загружаются
- Ошибка BSL-графа → XML-данные всё равно загружаются
- Ошибка подписок → роли и предопределённые загружаются
- Ошибка одной формы → остальные формы загружаются

По итогам индексации формируется структурированный отчёт (`IndexingReport`) с результатами каждого этапа.

### Ожидаемое время индексации

| Размер конфигурации | Время индексации (базовая) | С BSL-графом |
|---------------------|---------------------------|--------------|
| Небольшая | 2–6 часов | +1–2 часа |
| Средняя | 10–20 часов | +3–5 часов |
| Большая | 20–60 часов | +5–10 часов |

{% hint style="warning" %}
**Важно!** Обязательно используйте тома для сохранения данных Neo4j и индекса. Планируйте первый запуск на ночь или выходные.
{% endhint %}

## Защита embedding-модели

Система запоминает модель эмбеддингов после первой успешной индексации (в узле `SystemMeta`). При следующем запуске, если модель изменилась, сервер не запустится и выведет ошибку с указанием несоответствия.

Для смены модели:

1. Установите `RESET_DATABASE=true`
2. Перезапустите сервисы
3. Дождитесь переиндексации
4. Верните `RESET_DATABASE=false`

## Остановка

```powershell
docker-compose down
```

## Полный перезапуск

```powershell
# Остановить и удалить
docker-compose down -v

# Очистить данные Neo4j
Remove-Item -Recurse -Force "E:\bases\mcp_graph\neo4j\*"

# Запустить заново
docker-compose up -d
```

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
