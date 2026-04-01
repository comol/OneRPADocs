# MCP серверы

Обзор всех доступных MCP-серверов для разработки на 1С.

## Список серверов

| Сервер | Порт | Назначение | Сложность |
|--------|------|------------|-----------|
| [HelpSearchServer](help-search-server/) | 8003 | Справка платформы 1С | Средняя |
| [CodeMetadataSearchServer](code-metadata-search/) | 8000 | Метаданные и код конфигурации | Средняя |
| [CloudEmbeddingsServer](cloud-embeddings-server/) | 8000 | Облачная индексация с параллельными эмбеддингами | Средняя |
| [Graph Metadata Search](graph-metadata-search/) | 8006 | Графовый поиск связей | Высокая |
| [SSLSearchServer](ssl-search-server/) | 8008 | Библиотека стандартных подсистем | Низкая |
| [SyntaxCheckServer](syntax-check-server/) | 8002 | Проверка синтаксиса BSL | Низкая |
| [TemplatesSearchServer](templates-search-server/) | 8004 | Шаблоны кода 1С и проектная память | Низкая |
| [FormsServer](forms-server/) | 8011 | Схемы форм 1С | Низкая |
| [1CCodeChecker](code-checker/) | 8007 | Проверка через 1С:Напарник | Низкая |

## Классификация по сложности

### Простые (не требуют данных)

Можно запустить сразу после установки Docker:

1. **SyntaxCheckServer** — проверка синтаксиса
2. **FormsServer** — схемы форм
3. **TemplatesSearchServer** — шаблоны кода
4. **SSLSearchServer** — справка БСП

### Требующие данных

Нужна подготовка данных из конфигурации:

5. **HelpSearchServer** — нужна папка bin платформы 1С
6. **CodeMetadataSearchServer** — нужна выгрузка конфигурации
7. **CloudEmbeddingsServer** — нужна выгрузка конфигурации + API-ключ облачного провайдера
8. **Graph Metadata Search** — нужна выгрузка + Neo4j

### Требующие внешних ресурсов

8. **1CCodeChecker** — нужен токен 1С:Напарник

## Приоритет установки

### Этап 1: Быстрый старт

```powershell
# SyntaxCheckServer
docker run -d -p 8002:8002 --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest

# FormsServer
docker run -d -p 8011:8011 --name 1c_forms_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_forms:latest
```

### Этап 2: Если используете БСП

```powershell
# SSLSearchServer
docker run -d -p 8008:8008 --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -v "E:/bases/mcp_ssl:/app/chroma_db" `
  comol/mcp_ssl_server:latest
```

### Этап 3: Справка платформы

```powershell
# HelpSearchServer
docker run -d -p 8003:8003 --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

### Этап 4: Метаданные конфигурации

Требует выгрузки из Конфигуратора. См. [CodeMetadataSearchServer](code-metadata-search/).

## Общие переменные окружения

Переменные, которые используются во всех серверах:

| Переменная | Описание | Обязательная |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да |
| `RESET_DATABASE` | Переиндексировать данные | Нет (default: true) |
| `RESET_CACHE` | Перезагрузить модель | Нет (default: true) |
| `USESSE` | SSE транспорт | Нет (default: false) |

### Embedding модели

| Переменная | Описание |
|------------|----------|
| `OPENAI_API_BASE` | URL API (LM Studio: `http://host.docker.internal:1234/v1`) |
| `OPENAI_API_KEY` | Ключ API (любой для LM Studio) |
| `OPENAI_MODEL` | Название модели |
| `EMBEDDING_MODEL` | Встроенная CPU модель |

## Мониторинг контейнеров

### Просмотр запущенных контейнеров

```powershell
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### Просмотр логов

```powershell
# Все логи
docker logs 1c_help_mcp

# В реальном времени
docker logs -f 1c_help_mcp

# Последние 100 строк
docker logs --tail 100 1c_help_mcp
```

### Перезапуск контейнера

```powershell
docker restart 1c_help_mcp
```

### Остановка и удаление

```powershell
docker stop 1c_help_mcp
docker rm 1c_help_mcp
```

## Общие рекомендации

### Хранение данных

Всегда монтируйте внешние тома для:
- Векторных БД (`/app/chroma_db`)
- Кэша моделей (`/app/model_cache`)

### Первый запуск

При первом запуске происходит индексация данных:
- Может занять от минут до часов
- Следите за логами: `docker logs -f <container>`
- Используйте `RESET_DATABASE=false` для последующих запусков

### Embedding модели

Для лучшего качества используйте LM Studio или Ollama с Qwen моделями. Подробнее: [Embedding модели](../embedding-modeli/).
