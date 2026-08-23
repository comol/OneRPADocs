# MCP серверы

Обзор всех доступных MCP-серверов для разработки на 1С.

## Список серверов

| Сервер | Порт | Назначение | Сложность |
|--------|------|------------|-----------|
| [HelpSearchServer](help-search-server/) | 8003 | Справка платформы 1С, руководства, спецификации форматов, стандарты | Средняя |
| [CodeMetadataSearchServer](code-metadata-search/) | 8000 | Метаданные и код конфигурации | Средняя |
| [CloudEmbeddingsServer](cloud-embeddings-server/) | 8000* | Метаданные, код и справка через cloud embeddings | Средняя |
| [Graph Metadata Search](graph-metadata-search/) | 8006 | Графовый поиск связей | Высокая |
| [SSLSearchServer](ssl-search-server/) | 8008 | Библиотека стандартных подсистем | Низкая |
| [SyntaxCheckServer](syntax-check-server/) | 8002 | Проверка синтаксиса BSL | Низкая |
| [TemplatesSearchServer](templates-search-server/) | 8004 | Шаблоны кода 1С и проектная память | Низкая |
| [1CCodeChecker](code-checker/) | 8007 | Проверка через 1С:Напарник | Низкая |

{% hint style="info" %}
Актуальные версии серверов по умолчанию используют транспорт `streamable-http` на endpoint `/mcp`. Для старых клиентов можно включить SSE через `USESSE=true`; URL при этом остаётся `/mcp`, если в документации конкретного сервера не указано иное.
{% endhint %}

`*` CloudEmbeddingsServer имеет настраиваемый `MCP_PORT` и по умолчанию тоже использует 8000. При совместном запуске с CodeMetadataSearchServer вынесите его на свободный внешний порт, например 8001.

## Классификация по сложности

### Простые (не требуют данных)

Можно запустить сразу после установки Docker:

1. **SyntaxCheckServer** — проверка синтаксиса строки BSL; при подключении каталога `FILES_DIR` также проверяет файлы
2. **TemplatesSearchServer** — шаблоны кода
3. **SSLSearchServer** — справка БСП

### Требующие данных

Нужна подготовка данных из конфигурации:

5. **HelpSearchServer** — работает сразу (корпуса в образе); папка bin платформы нужна только для справки своей версии
6. **CodeMetadataSearchServer** — нужна выгрузка конфигурации
7. **Graph Metadata Search** — нужна выгрузка + Neo4j
8. **CloudEmbeddingsServer** — нужна выгрузка и внешний embedding API

### Требующие внешних ресурсов

9. **1CCodeChecker** — нужен токен 1С:Напарник

## Приоритет установки

### Этап 1: Быстрый старт

```powershell
# SyntaxCheckServer
docker run -d -p 8002:8002 --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest

```

### Этап 2: Если используете БСП

```powershell
# SSLSearchServer
docker run -d -p 8008:8008 --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -v "E:/bases/mcp_ssl:/app/zvec_db" `
  comol/mcp_ssl_server:latest
```

### Этап 3: Справка платформы

```powershell
# HelpSearchServer
docker run -d -p 8003:8003 --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:latest
```

Справка платформы поставляется внутри образа. Чтобы индексировать справку своей версии, добавьте `-v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs"` и `-e 1C_BIN_PATH=/1c_docs`.

### Этап 4: Метаданные конфигурации

Требует выгрузки из Конфигуратора. См. [CodeMetadataSearchServer](code-metadata-search/).

## Общие переменные окружения

Переменные, которые используются во всех серверах:

| Переменная | Описание | Обязательная |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да |
| `RESET_DATABASE` | Переиндексировать данные | Нет (default: false) |
| `RESET_CACHE` | Очистить кэш моделей | Нет (default: false) |
| `USESSE` | SSE транспорт | Нет (default: false) |

### Embedding модели

| Переменная | Описание |
|------------|----------|
| `EMBEDDING_API_BASE` | URL API (LM Studio: `http://host.docker.internal:1234/v1`) |
| `EMBEDDING_API_KEY` | Ключ API (любой для LM Studio) |
| `EMBEDDING_MODEL` | Модель API или встроенная CPU-модель |

Старые имена `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` остаются совместимыми алиасами.

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
- Векторных БД (`/app/chroma_db`, `/app/data` или путь, указанный в странице конкретного сервера)
- Кэша моделей (`/app/model_cache`)

### Первый запуск

При первом запуске происходит индексация данных:
- Может занять от минут до часов
- Следите за логами: `docker logs -f <container>`
- Используйте `RESET_DATABASE=false` для последующих запусков

### Embedding модели

Для лучшего качества используйте LM Studio или Ollama с Qwen моделями. Подробнее: [Embedding модели](../embedding-modeli/).
