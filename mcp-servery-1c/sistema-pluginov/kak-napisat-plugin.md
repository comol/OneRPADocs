# Как написать плагин

Пошаговый сценарий доработки любого MCP-сервера, поддерживающего плагины. Команды даны для PowerShell на Windows.

## Шаг 1. Прочитать контракт из образа

Единственный источник истины — читаемые файлы внутри образа. Их три:

| Файл в образе | Что это |
|---------------|---------|
| `/app/plugin_api.py` | Полный справочник: каждый хук и таблица продукта, форма payload, какие поля применяются, рабочий пример |
| `/app/plugins/AGENTS.md` | Короткая версия для агента: модель, правила, чем платят за правку |
| `/app/plugins/example.py` | Пример, объявляющий все хуки и таблицы продукта; безвреден, копируется целиком |

Исключения по пути справочника: CodeMetadataSearchServer — `/app/src/plugin_api.py`, 1CCodeChecker — `/app/MCP_1copilot/plugin_api.py`.

Достать их можно, не запуская сервер:

```powershell
# Из работающего контейнера
docker cp 1c_help_mcp:/app/plugin_api.py .
docker cp 1c_help_mcp:/app/plugins/AGENTS.md .
docker cp 1c_help_mcp:/app/plugins/example.py .

# Или из образа, без контейнера
docker run --rm comol/1c_help_mcp:latest cat /app/plugin_api.py > plugin_api.py
```

{% hint style="info" %}
`plugin_api.py` не импортирует ничего вне стандартной библиотеки — его можно открыть и прочитать любым интерпретатором и любым редактором. Это сделано специально: он рассчитан на чтение ИИ-агентом.
{% endhint %}

## Шаг 2. Написать один файл

```powershell
New-Item -ItemType Directory -Force -Path "E:\plugins\mcp_docs"
```

Файл `E:\plugins\mcp_docs\10-terminy.py`:

```python
"""Термины нашей компании в поиске по справке."""

REQUIRES = {"host": 1, "product": "1c-help", "hooks": 1}

TERMS = {}


def on_startup(config):
    """Один раз при старте. Возвращаемое значение игнорируется."""
    global TERMS
    TERMS = {"ТЗ": "ТаблицаЗначений", "РС": "РегистрСведений"}


def on_request(request):
    """Каждый вызов инструмента, после валидации аргументов."""
    if request["tool"] != "docsearch":
        return
    text = request["arguments"]["query"]
    for short, full in TERMS.items():
        text = text.replace(short, full)
    request["arguments"]["query"] = text
    return request
```

Три вещи, которые чаще всего делают неправильно:

1. **Забывают `return`.** Хук получает копию payload. Изменить её и не вернуть — это не частичный эффект, это отсутствие эффекта.
2. **Пишут хук, которого у продукта нет.** Имя вне объявленного набора отклоняет весь файл. Набор — в `plugin_api.py` этого сервера, а не другого.
3. **Правят derived-state хук ради call-scoped задачи.** Это стоит пересборки индекса. Сначала проверьте, нет ли подходящей [декларативной таблицы](README.md).

**Имя файла задаёт порядок.** Файлы выполняются в отсортированном порядке имён, каждый хук — конвейер: что вернул `10-terminy.py`, то получит `20-filtr.py`. Называйте файлы так, чтобы читаемый порядок совпадал с фактическим.

## Шаг 3. Прогнать dry-run

Dry-run запускает объявленные в файле хуки на фикстурах, зашитых в образ, и печатает payload до и после. Ему не нужны ни смонтированные данные, ни построенный индекс, ни запущенный сервер, ни лицензионный ключ. Ошибку загрузки он сообщает теми же словами, что и сервер.

| Сервер | Команда внутри контейнера |
|--------|---------------------------|
| HelpSearchServer | `python3 plugin_host.py --dry-run /tmp/my_plugin.py` |
| SSLSearchServer | `python launcher.py --dry-run /tmp/my_plugin.py` |
| SyntaxCheckServer | `python mcp_server.py --dry-run /tmp/my_plugin.py` |
| TemplatesSearchServer | `python main.py --dry-run /tmp/my_plugin.py` |
| Graph Metadata Search | `python run.py plugin-dry-run /tmp/my_plugin.py` |
| CodeMetadataSearchServer | `python src/plugin_dry_run.py /tmp/my_plugin.py` |

Полностью, одной строкой и без запущенного сервера:

```powershell
# HelpSearchServer
docker run --rm -v "E:/plugins/mcp_docs/10-terminy.py:/tmp/my_plugin.py" `
  comol/1c_help_mcp:latest python3 plugin_host.py --dry-run /tmp/my_plugin.py

# SSLSearchServer
docker run --rm -v "E:/plugins/mcp_ssl/10-terminy.py:/tmp/my_plugin.py" `
  comol/mcp_ssl_server:latest python launcher.py --dry-run /tmp/my_plugin.py

# SyntaxCheckServer
docker run --rm -v "E:/plugins/mcp_syntax/10-podavlenie.py:/tmp/my_plugin.py" `
  comol/1c_syntaxcheck_mcp:latest python mcp_server.py --dry-run /tmp/my_plugin.py

# TemplatesSearchServer
docker run --rm -v "E:/plugins/mcp_templates/10-aliasy.py:/tmp/my_plugin.py" `
  comol/template-search-mcp:latest python main.py --dry-run /tmp/my_plugin.py

# Graph Metadata Search
docker run --rm -v "E:/plugins/mcp_graph/10-svojstva.py:/tmp/my_plugin.py" `
  comol/1c_graph_metadata:latest python run.py plugin-dry-run /tmp/my_plugin.py

# CodeMetadataSearchServer
docker run --rm -v "E:/plugins/mcp_code/10-terminy.py:/tmp/my_plugin.py" `
  comol/1c_code_metadata_mcp:latest-beta python src/plugin_dry_run.py /tmp/my_plugin.py

# 1CCodeChecker
docker run --rm -v "E:/plugins/checker/10-policy.py:/tmp/my_plugin.py" `
  comol/1c-code-checker:latest-beta `
  python -m MCP_1copilot --dry-run /tmp/my_plugin.py
```

{% hint style="info" %}
У TemplatesSearchServer предпочтительна именно `python main.py --dry-run`: она прогоняет оба набора фикстур и применяет проверки таблиц этого продукта. Общий `python plugin_host.py --dry-run` тоже работает, но проверяет только *форму* таблицы, поэтому пропустит запись, которую сервер отвергнет.
{% endhint %}

## Шаг 4. Смонтировать каталог

Каталог плагинов подключается томом. Один каталог — один сервер: плагин, написанный для другого продукта, отклоняется по `PRODUCT`, если объявил `REQUIRES`, и по неизвестным именам хуков, если не объявил. Именно поэтому `REQUIRES` стоит писать: ошибка получается внятной с первого раза.

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "E:/bases/mcp_docs:/app/index" `
  -v "E:/plugins/mcp_docs:/app/plugins" `
  comol/1c_help_mcp:latest
```

Для Graph Metadata Search каталог мало смонтировать — подсистему нужно включить:

```powershell
docker run -d -p 8006:8006 `
  --name 1c_graph_metadata `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e GRAPH_PLUGINS_ENABLED=true `
  -v "E:/plugins/mcp_graph:/app/plugins" `
  comol/1c_graph_metadata:latest
```

Переменные, которые относятся к плагинам:

| Переменная | Серверы | Назначение | По умолчанию |
|------------|---------|------------|--------------|
| `PLUGIN_DIR` | Help, SSL, Templates, Code, 1CCodeChecker | Каталог плагинов внутри контейнера | `/app/plugins` |
| `PLUGINS_DIR` | Syntax | То же; пустое значение означает `/app/plugins` | *(пусто)* |
| `PLUGIN_STRICT_DERIVED_STATE` | Help, SSL, Templates, Code | `true` — упавший derived-state хук роняет сборку вместо пропуска единицы | `false` |
| `PLUGIN_HOOK_TIMEOUT_SECONDS` | Code | Бюджет времени одного хука; превысивший его хук считается упавшим | `5.0` |
| `GRAPH_PLUGINS_ENABLED` | Graph | Читать каталог плагинов вообще | `false` |
| `GRAPH_PLUGINS_DIRECTORY` | Graph | Каталог плагинов; относительный путь считается от `/app` | `plugins` |
| `GRAPH_PLUGIN_STRICT_BUILD` | Graph | Ронять построение поколения при ошибке derived-state хука | `false` |
| `GRAPH_PLUGIN_HOOK_TIMEOUT_SECONDS` | Graph | Бюджет времени одного хука; `0` отключает контроль | `5.0` |

## Шаг 5. Перезагрузить каталог без перезапуска

Перезагрузка перечитывает каждый файл каталога, атомарно заменяет действующий набор хуков и начинает новую **эпоху плагинов**. Если хотя бы один файл больше не загружается, не заменяется ничего: прежний набор остаётся в силе, а ответ называет проблемный файл и причину.

| Сервер | Что загружено | Перезагрузка |
|--------|---------------|--------------|
| HelpSearchServer | `GET /plugins` | `POST /plugins/reload` (HTTP 409, если перезагрузка отклонена) |
| SSLSearchServer | MCP-инструмент `plugin_state` | MCP-инструмент `plugin_reload` |
| SyntaxCheckServer | MCP-инструмент `plugin_state` | MCP-инструмент `plugin_reload` |
| TemplatesSearchServer | MCP-инструмент `plugin_state` | MCP-инструмент `plugin_reload` — только если включены изменяющие инструменты |
| Graph Metadata Search | MCP-инструмент `list_plugins` | MCP-инструмент `reload_plugins(operation_id)` — только профиль `admin` |
| CodeMetadataSearchServer | MCP-инструмент `plugin_state` | MCP-инструмент `plugin_reload` |
| 1CCodeChecker | `GET /plugins` | `POST /plugins/reload` (HTTP 409, если новый набор не загружается) |

```powershell
# HelpSearchServer — control plane, отвечает и во время сборки индекса
Invoke-RestMethod -Uri "http://localhost:8003/plugins"
Invoke-RestMethod -Method Post -Uri "http://localhost:8003/plugins/reload"
```

{% hint style="warning" %}
**TemplatesSearchServer.** `plugin_reload` относится к изменяющим инструментам (вместе с `add_template` и `remember`). По умолчанию они не регистрируются вообще: нужно задать `MCP_ENABLE_WRITE_TOOLS`, и вместе с ним — стойкий `MCP_OPERATOR_TOKEN`, иначе сервер откажется стартовать. Токен передаётся в заголовке `Authorization`, а не аргументом инструмента. Без них перечитать каталог можно перезапуском контейнера, а `plugin_state` доступен всегда.
{% endhint %}

**Что когда начинает действовать:** call-scoped хук и таблица — со следующего вызова после перезагрузки; derived-state хук — со следующего построения индекса, не задним числом.

## Шаг 6. Проверить состояние

Ответ `plugin_state` / `list_plugins` / `GET /plugins` показывает по каждому файлу: какие хуки и таблицы он объявил, классификацию каждого хука, включён он или отключён, ошибку, которая его отключила, счётчик сбоев, накопленное в хуке время, порядок выполнения файлов и текущую эпоху плагинов.

Три вопроса, на которые он отвечает быстрее, чем журнал:

* **Плагин загрузился?** Файла нет в списке — он не в каталоге или каталог не смонтирован (у Graph — ещё и `GRAPH_PLUGINS_ENABLED`).
* **Почему хук не срабатывает?** Отключён после исключения — рядом будет ошибка и счётчик сбоев.
* **Почему стало медленно?** У каждого хука показано накопленное время.

## Типичные ошибки загрузки

| Симптом в ответе или журнале | Причина | Что делать |
|------------------------------|---------|------------|
| Имя хука названо и перечислены разрешённые | Опечатка или хук другого продукта | Взять имя из `plugin_api.py` этого сервера |
| Назван неизвестный параметр и предложены доступные | Опечатка в имени аргумента хука | Хук получает аргументы **по именам**; берите только нужные |
| Названы продукт плагина и продукт сервера | `REQUIRES` из другого сервера семейства | Исправить `REQUIRES` или переложить файл в правильный каталог |
| Названы требуемая и предлагаемая версии | `HOOKS_VERSION`/`HOST_CONTRACT_VERSION` не совпали | Сверить payload с `plugin_api.py` новой версии |
| Названы файл и правило кодировки | Файл не в UTF-8 | Пересохранить в UTF-8 (BOM допустим) |
| Названы запись таблицы и что с ней не так | Некорректная декларативная таблица | Исправить запись; форму смотреть в `plugin_api.py` |
| Хук отмечен отключённым, рядом трассировка | Исключение в хуке | Исправить и перезагрузить: до перезагрузки хук не срабатывает |
| Результат хука отброшен, в журнале имя файла | Возврат нарушил контракт ответа | Вернуть ту же форму; не добавлять записей, которых сервер не производил |

Полный разбор правил, которые за этим стоят: [Правила хоста](pravila-hosta.md).
