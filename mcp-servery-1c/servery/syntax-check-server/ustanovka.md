# Установка SyntaxCheckServer

## Предварительные требования

- Docker Desktop запущен

## Команда запуска

```powershell
docker run -d -p 8002:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest
```

Это всё! Сервер не требует дополнительной настройки.

### На arm64-хосте

Стабильный образ публикуется только для linux/amd64 — upstream не выпускает arm64-бинарник анализатора, а анализатор обязан быть нативным. На Apple Silicon и ARM-серверах есть два пути:

```powershell
# 1. Стабильный образ под эмуляцией
docker run -d -p 8002:8002 --platform linux/amd64 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest

# 2. Нативный beta-образ
docker run -d -p 8002:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:arm64-beta
```

{% hint style="info" %}
В `arm64-beta` анализатор собирается из исходников закреплённого релиза, а не берётся готовым от upstream. Сборка проверяет коммит, архитектуру ELF, версию анализатора и воспроизведение эталонного набора диагностик — но образ не воспроизводим побайтно и не проверялся на реальном arm64-железе, поэтому канал остаётся бетой.
{% endhint %}

## Переменные окружения

| Переменная | Описание | Обязательная |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да |
| `USESSE` | SSE транспорт для legacy клиентов | Нет |
| `FILES_DIR` | Каталог BSL-файлов; включает инструмент `syntaxcheck_file`, если каталог существует | Нет |
| `PLUGINS_DIR` | Каталог Python-плагинов; пустое значение использует `/app/plugins` | Нет |
| `LOG_LEVEL` | Уровень журналирования сервера и ошибок плагинов | Нет (`INFO`) |
| `MCP_HTTP_PATH` | Endpoint `streamable-http` | Нет (`/mcp`) |
| `MCP_SSE_PATH` | Endpoint потока событий в SSE-режиме | Нет (`/sse`) |
| `MCP_MESSAGE_PATH` | Endpoint отправки сообщений в SSE-режиме | Нет (`/messages/`) |
| `BSL_ANALYZER_TIMEOUT_SECONDS` | Таймаут запуска анализатора | Нет (`30`) |
| `BSL_ANALYZER_STDOUT_LIMIT_BYTES` | Максимальный размер сырого JSONL-отчёта анализатора до преобразования опубликованной текстовой части beta в TOON | Нет (`16777216`) |
| `BSL_ANALYZER_STDERR_LIMIT_BYTES` | Максимальный размер диагностического вывода анализатора | Нет (`4194304`) |
| `BSL_ANALYZER_KILL_GRACE_SECONDS` | Ожидание между terminate и принудительным kill | Нет (`2`) |
| `BSL_SOURCE_ENCODING` | Кодировка файлов из `FILES_DIR`; пустое значение пробует UTF-8 с BOM, затем CP1251 | Нет |

## Плагины

Свой каталог плагинов можно подключить томом в `/app/plugins`. Инструмент `plugin_state` показывает текущий набор, а `plugin_reload` перечитывает его без перезапуска контейнера.

Плагин — **один Python-файл**: ни базового класса, ни декоратора, ни регистрации, ни манифеста. Объявлены четыре hooks — `on_startup`, `on_request`, `on_diagnostics`, `on_result` — и таблица `SUPPRESSED_DIAGNOSTICS` (код диагностики → причина, по которой она не попадает в отчёт). **Все хуки call-scoped:** сервер ничего не хранит между вызовами, поэтому здесь нет ни пересборок, ни отпечатков годности, ни курсоров.

```powershell
# Проверить плагин без запущенного сервера и без анализатора
docker run --rm -v "E:/plugins/mcp_syntax/10-profile.py:/tmp/my_plugin.py" `
  comol/1c_syntaxcheck_mcp:latest-beta python mcp_server.py --dry-run /tmp/my_plugin.py
```

{% hint style="warning" %}
Номера строк в payload хуков **нулевые** — публикуемая база применяется уже после хуков. Аргумент `file_name` присутствует в каждом запросе `syntaxcheck` (пустая строка, если вызывающая сторона его не задала); из-за его появления версия набора хуков этого сервера — `HOOKS_VERSION = 2`.
{% endhint %}

Полный контракт — в образе (`/app/plugin_api.py`, `/app/plugins/AGENTS.md`, `/app/plugins/example.py`). Общие правила и рецепты: [Доработка MCP: система плагинов](../../sistema-pluginov/) и [справочник хуков SyntaxCheckServer](../../sistema-pluginov/spravochnik-hukov.md#syntaxcheckserver).

### SSE транспорт

При `USESSE=true` клиент подключается к `http://localhost:8002/sse`; `/messages/` — служебный endpoint, который сообщает сам SSE-поток, его не указывают как URL сервера в конфигурации клиента.

Если ваш клиент требует SSE:

```powershell
docker run -d -p 8002:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e USESSE=true `
  comol/1c_syntaxcheck_mcp:latest
```

## Проверка работы

### Статус контейнера

```powershell
docker ps --filter name=1c_syntaxcheck_mcp
```

### Просмотр логов

```powershell
docker logs 1c_syntaxcheck_mcp
```

## Конфигурация Cursor

### mcp.json для streamable-http

```json
{
  "mcpServers": {
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    }
  }
}
```

Для SSE укажите endpoint потока событий:

```json
{
  "mcpServers": {
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/sse",
      "transport": "sse"
    }
  }
}
```

## Управление контейнером

```powershell
# Остановить
docker stop 1c_syntaxcheck_mcp

# Запустить
docker start 1c_syntaxcheck_mcp

# Перезапустить
docker restart 1c_syntaxcheck_mcp

# Удалить
docker rm -f 1c_syntaxcheck_mcp
```

## Устранение проблем

### Контейнер не запускается

```powershell
docker logs 1c_syntaxcheck_mcp
```

### Порт занят

```powershell
netstat -ano | findstr :8002
```

Если порт занят, используйте другой порт:

```powershell
docker run -d -p 8102:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest
```

И измените порт в mcp.json: `http://localhost:8102/mcp`
