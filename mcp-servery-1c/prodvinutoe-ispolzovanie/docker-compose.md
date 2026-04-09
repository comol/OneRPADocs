# Docker Compose

Оркестрация всех MCP-серверов через docker-compose.

## Преимущества

- Один файл для всей конфигурации
- Запуск/остановка всех сервисов одной командой
- Управление зависимостями
- Health checks

## Базовый docker-compose.yml

```yaml
version: '3.8'

services:
  # ============================================
  # Простые серверы (без embedding)
  # ============================================
  
  syntax-check:
    image: comol/1c_syntaxcheck_mcp:latest
    container_name: 1c_syntaxcheck_mcp
    restart: unless-stopped
    ports:
      - "8002:8002"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}

  # ============================================
  # Серверы с embedding
  # ============================================

  help-search:
    image: comol/1c_help_mcp:latest
    container_name: 1c_help_mcp
    restart: unless-stopped
    ports:
      - "8003:8003"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - 1C_BIN_PATH=/1c_docs
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=Qwen3-Embedding-4B
    volumes:
      - ${1C_BIN_PATH}:/1c_docs:ro
      - E:/bases/mcp_docs:/app/chroma_db
      - E:/bases/mcp_model_cache:/app/model_cache

  ssl-search:
    image: comol/mcp_ssl_server:latest
    container_name: mcp_ssl_server
    restart: unless-stopped
    ports:
      - "8008:8008"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - SSL_VERSION=${SSL_VERSION:-3.1.11}
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=Qwen3-Embedding-4B
    volumes:
      - E:/bases/mcp_ssl:/app/chroma_db

  templates-search:
    image: comol/template-search-mcp:latest
    container_name: template_search_mcp
    restart: unless-stopped
    ports:
      - "8004:8004"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=Qwen3-Embedding-4B
    volumes:
      - E:/bases/mcp_templates:/app/data
```

## Файл .env

Создайте файл `.env` рядом с `docker-compose.yml`:

```env
LICENSE_KEY=YOUR_LICENSE_KEY
1C_BIN_PATH=C:/Program Files/1cv8/8.3.23.1997/bin
SSL_VERSION=3.1.11
```

## Использование

### Запуск всех сервисов

```powershell
docker-compose up -d
```

### Остановка

```powershell
docker-compose down
```

### Просмотр логов

```powershell
# Все сервисы
docker-compose logs -f

# Конкретный сервис
docker-compose logs -f help-search
```

### Перезапуск одного сервиса

```powershell
docker-compose restart ssl-search
```

### Статус

```powershell
docker-compose ps
```

## Расширенный docker-compose.yml

С Graph Metadata Search и CodeMetadataSearchServer:

```yaml
version: '3.8'

services:
  # ... (базовые сервисы сверху) ...

  # ============================================
  # CodeMetadataSearchServer
  # ============================================
  
  code-metadata:
    image: comol/1c_code_metadata_mcp:latest
    container_name: 1c_code_metadata_mcp
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - METADATA_PATH=/app/metadata
      - CODE_PATH=/app/code
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=Qwen3-Embedding-4B
    volumes:
      - E:/1C_Export/Report:/app/metadata:ro
      - E:/1C_Export/Files:/app/code:ro
      - E:/bases/mcp_codemetadata:/app/chroma_db

  # ============================================
  # Graph Metadata Search (Neo4j + MCP)
  # ============================================

  neo4j:
    image: neo4j:latest
    container_name: neo4j
    restart: unless-stopped
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      - NEO4J_AUTH=neo4j/password123
      - NEO4J_server_memory_heap_max__size=1g
    volumes:
      - E:/bases/mcp_graph/neo4j:/data
    healthcheck:
      test: ["CMD-SHELL", "wget --spider localhost:7474 || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  graph-metadata:
    image: comol/1c_graph_metadata:latest
    container_name: 1c_graph_metadata
    restart: unless-stopped
    ports:
      - "8006:8006"
    environment:
      - LICENSE_KEY=${LICENSE_KEY}
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USERNAME=neo4j
      - NEO4J_PASSWORD=password123
      - METADATA_DIRECTORY=/app/metadata
      - RESET_DATABASE=false
      - OPENAI_API_BASE=http://host.docker.internal:1234/v1
      - OPENAI_API_KEY=lm-studio
      - OPENAI_MODEL=Qwen3-Embedding-4B
    volumes:
      - E:/1C_Export/Report:/app/metadata:ro
    depends_on:
      neo4j:
        condition: service_healthy
```

## Health Checks

Добавьте проверки состояния:

```yaml
services:
  help-search:
    # ... остальные параметры ...
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8003/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Ограничение ресурсов

```yaml
services:
  help-search:
    # ... остальные параметры ...
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '2'
```
