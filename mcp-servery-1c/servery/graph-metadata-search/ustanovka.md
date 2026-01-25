# Установка Graph Metadata Search

Graph Metadata Search требует два сервиса: Neo4j и MCP сервер. Рекомендуется использовать docker-compose.

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен (рекомендуется)
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
      - OPENAI_MODEL=Qwen3-Embedding-4B
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

### MCP сервер

```powershell
curl http://localhost:8006/health
```

## Первый запуск

При первом запуске происходит:

1. Запуск Neo4j (~30 сек)
2. Подключение MCP сервера к Neo4j
3. Парсинг метаданных
4. Построение графа связей
5. Создание векторного индекса

### Ожидаемое время индексации

| Размер конфигурации | Время индексации |
|---------------------|------------------|
| Небольшая | 2-6 часов |
| Средняя | 10-20 часов |
| Большая | 20-60 часов |

{% hint style="warning" %}
**Важно!** Обязательно используйте тома для сохранения данных Neo4j и индекса. Планируйте первый запуск на ночь или выходные.
{% endhint %}

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
