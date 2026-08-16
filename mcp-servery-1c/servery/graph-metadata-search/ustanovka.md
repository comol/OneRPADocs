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
    "E:\1C_Export\Report",
    "E:\1C_Export\Files"
)
```

## Создание docker-compose.yml

Создайте файл `docker-compose.yml`:

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
    volumes:
      - E:/bases/mcp_graph/neo4j:/data
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
      - LICENSE_KEY=YOUR_LICENSE_KEY
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_EMBEDDING_MODEL=Qwen3-Embedding-4B
      - TEMPLATE_MODE_ENABLED=true
      - CODE_EXPORT_PATH=/app/metadata_files
      - LOAD_BSL_SIGNATURES=true
    volumes:
      - E:/1C_Export/Report:/app/metadata
      - E:/1C_Export/Files:/app/metadata_files
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

### MCP сервер — health

```powershell
curl http://localhost:8006/healthz
```

Дешёвая liveness-проба: отвечает сразу, как только процесс жив и создан драйвер Neo4j, без запросов к графу. Именно её использует healthcheck в `docker-compose.yml`. Готовность проверяется отдельно:

```powershell
curl http://localhost:8006/readyz
```

### Статус фоновых задач

```powershell
curl http://localhost:8006/status
```

Возвращает состояние фоновых процессов: business info generation, vector indexing, routine embeddings. Каждая задача может быть в состоянии `pending`, `running`, `completed` или `failed`.

### Прогресс индексации

```powershell
curl http://localhost:8006/search/index-status
```

### OpenAPI документация

Откройте в браузере: `http://localhost:8006/docs`

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
