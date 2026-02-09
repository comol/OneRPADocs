# Конфигурация Graph Metadata Search

## Переменные окружения MCP сервера

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |
| `NEO4J_URI` | URI подключения к Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь Neo4j | `neo4j` |
| `NEO4J_PASSWORD` | Пароль Neo4j | `password123` |
| `METADATA_DIRECTORY` | Путь к метаданным | `/app/metadata` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать при запуске | `true` |
| `AUTO_UPDATE_ON_STARTUP` | Авто-обновление индексов при запуске | `true` |
| `ASYNC_VECTOR_INDEXING` | Индексация векторов в фоне (неблокирующая) | `true` |
| `INDEX_BATCH_SIZE` | Количество объектов, обрабатываемых за один пакет при создании векторного индекса | `50` |
| `MAX_TOKENS_PER_BATCH` | Максимальное количество токенов в одном пакете запроса к API эмбеддингов. Защита от превышения лимита 8192 токена | `7500` |
| `EMBEDDING_MAX_TOKENS` | Максимальное количество токенов на один текст при генерации эмбеддингов. Определяется автоматически по модели, но можно переопределить | *(авто)* |

### Embedding модели

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API для генерации и LLM | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API для генерации и LLM | `lm-studio` |
| `OPENAI_MODEL` | LLM модель для генерации описаний | `gpt-4o` |
| `OPENAI_TEMPERATURE` | Температура генерации (0–1). Ниже — более детерминированный результат | `0.1` |
| `OPENAI_MAX_COMPLETION_TOKENS` | Максимальное количество токенов в ответе LLM | `2000` |
| `OPENAI_MODEL_IS_REASONING` | Принудительное указание, является ли модель «рассуждающей». Если не указано — определяется автоматически по имени модели | *(авто)* |
| `OPENAI_EMBEDDING_API_BASE` | Отдельный URL для API эмбеддингов (если отличается от основного) | — |
| `OPENAI_EMBEDDING_API_KEY` | Отдельный ключ для API эмбеддингов | — |
| `OPENAI_EMBEDDING_MODEL` | Модель для эмбеддингов | `Qwen3-Embedding-4B` |
| `OPENAI_EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов. Для моделей с переменной размерностью (Qwen3: 32–4096, text-embedding-3: 256–3072) | *(авто)* |
| `EMBEDDING_MODEL` | Локальная CPU модель (sentence-transformers) | `intfloat/multilingual-e5-small` |

### Поиск по коду

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ENABLE_CODE_SEARCH` | Включить поиск по BSL-файлам | `true` |
| `CODE_SEARCH_MAX_FILE_SIZE` | Максимальный размер BSL-файла (байт) для индексации | `50000` |
| `ENABLE_BUSINESS_SEARCH` | Включить семантический поиск по бизнес-описаниям | `true` |

### Дополнительные

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_HOST` | Хост MCP-сервера | `0.0.0.0` |
| `MCP_PORT` | Порт MCP | `8006` |
| `MCP_PATH` | URL-путь для MCP эндпоинта | `/mcp` |
| `MCP_USE_SSE` | SSE транспорт (для legacy клиентов) | `false` |
| `CALCULATE_BUSINESS_INFO` | Генерировать AI бизнес-описания для объектов метаданных перед индексацией | `false` |
| `PROJECT_NAME` | Название проекта (для логов и интерфейса) | `1C Metadata Project` |
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
| `E:/1C_Export/Report` | `/app/metadata` | Отчет по метаданным |
| `E:/1C_Export/Files` | `/app/metadata_files` | Файлы кода |

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
      - LICENSE_KEY=YOUR_LICENSE_KEY
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=gpt-4o
      - OPENAI_EMBEDDING_MODEL=Qwen3-Embedding-4B
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
