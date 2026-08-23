# Справочник хуков по серверам

Набор хуков объявляет каждый сервер сам. Ниже — что предлагает каждый из пяти серверов с плагинами: имена, классификация, момент срабатывания, форма payload и какие поля применяются из возвращённого значения.

{% hint style="info" %}
Эта страница — карта. Единственный полный и всегда актуальный справочник — `/app/plugin_api.py` внутри образа: в нём для каждого хука и таблицы есть форма payload и рабочий пример. Достать: `docker run --rm <образ> cat /app/plugin_api.py`.
{% endhint %}

## Сводная таблица хуков

| Хук | Help | SSL | Syntax | Templates | Graph |
|-----|------|-----|--------|-----------|-------|
| `on_startup` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `on_request` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `on_result` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `on_query` | ✅ | — | — | — | — |
| `on_diagnostics` | — | — | ✅ | — | — |
| `on_rank` | — | — | — | ✅ | — |
| `on_search_candidates` | — | — | — | — | ✅ |
| `on_document` *(derived)* | ✅ | — | — | — | — |
| `on_entry` *(derived)* | — | ✅ | — | — | — |
| `on_template`, `on_memory` *(derived)* | — | — | — | ✅ | — |
| `on_source_unit`, `on_metadata_object`, `on_routine`, `on_embedding_document` *(derived)* | — | — | — | — | ✅ |
| Таблицы | `ALIASES`, `TOOL_PRESETS` | `QUERY_ALIASES` | `SUPPRESSED_DIAGNOSTICS` | `QUERY_ALIASES` | `ALIASES`, `TOOL_PRESETS`, `CYPHER_TEMPLATES` |

---

## HelpSearchServer

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/1c_help_mcp` / 8003 |
| `PRODUCT` | `1c-help` |
| Версии | `HOST_CONTRACT_VERSION = 1`, `HOOKS_VERSION = 1` |
| Каталог | `/app/plugins` (`PLUGIN_DIR`) |
| Интроспекция | `GET /plugins`, перезагрузка `POST /plugins/reload` |
| Dry-run | `python3 plugin_host.py --dry-run <файл>` |
| Цена derived-state правки | Пересборка индекса справки в staging-поколение; текущее продолжает отвечать |

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | Один раз, до того как сервер что-либо строит или открывает | Ничего: возврат игнорируется |
| `on_request(request)` | call-scoped | Каждый вызов инструмента, после валидации аргументов | `arguments`; результат валидируется заново, `tool` только для чтения |
| `on_query(query, request)` | call-scoped | После нормализации текста поиска, до обращения к индексу | Только `normalized`, непустая строка |
| `on_result(result, request)` | call-scoped | После работы, до применения контракта ответа | Только `items`: переупорядочить или сократить |
| `on_document(doc)` | **derived-state** | При сборке, после чтения документа, до нарезки на чанки | `markdown`, `metadata`, `skip` |

```python
# config: словарь всех настроек; секреты приходят как "<redacted>"
# request: {"tool": "docsearch" | "docinfo" | "formatspec" | "standards",
#           "arguments": {"query"/"name", "top_k", "doc_type", "scope",
#                         "max_chars", "max_items", "detail_level", "cursor"}}
# query:   {"tool": ..., "text": ..., "normalized": ...}
# result:  {"tool", "outcome", "query", "generation", "total",
#           "items": [{"doc_id", "citation", "snippets", "snippet_count",
#                      "truncated", "score", "lanes"}]}
# doc:     {"corpus", "source", "doc_id", "markdown", "metadata", "skip"}
```

`on_result` может только переупорядочивать и сокращать: элемент с `doc_id`, которого этот вызов не производил, или дубликат отбрасывает всё возвращённое значение. После хука пересчитываются `max_items`, `max_chars`, счётчики и курсор. В `on_document` поля `corpus`, `source`, `doc_id` только для чтения — их изменение отбрасывает возврат целиком.

**Таблицы**

| Таблица | Форма | Действие |
|---------|-------|----------|
| `ALIASES` | `{алиас: каноническое имя}` | Дополнительные имена, которые разрешает `docinfo`. Не требует пересборки. Конфликт с проверенным алиасом или с алиасом более раннего плагина отклоняет загрузку с указанием обеих сторон |
| `TOOL_PRESETS` | список `{"name", "description", "tool", "bind"}` | Новый MCP-инструмент как пресет `docsearch` или `docinfo` с зафиксированными аргументами. Пресет появляется в `tools/list` под своим именем и без связанных параметров в схеме |

---

## SSLSearchServer

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/mcp_ssl_server` / 8008 |
| `PRODUCT` | `ssl-search` |
| Версии | `HOST_CONTRACT_VERSION = 1`, `HOOKS_VERSION = 1` |
| Каталог | `/app/plugins` (`PLUGIN_DIR`) |
| Интроспекция | MCP-инструменты `plugin_state`, `plugin_reload` |
| Dry-run | `python launcher.py --dry-run <файл>` |
| Цена derived-state правки | Полное переэмбеддирование выбранной базы `bases/<версия>.db` |

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | Один раз, до инициализации embedding-модели | Ничего |
| `on_request(request)` | call-scoped | Каждый вызов `ssl_search`, после валидации аргументов и до нормализации запроса | Только `arguments["query"]` |
| `on_result(result, request)` | call-scoped | После извлечения, до применения контракта ответа | `entries` |
| `on_entry(entry)` | **derived-state** | Сборка: один раз на запись БСП, перед эмбеддингом | `description`, `full_content`, `skip` |

```python
# request: {"tool": "ssl_search",
#           "arguments": {"query": str, "limit": 1..20,
#                         "min_score": float | None, "database": str | None}}
# result:  {"entries": [{"database", "version", "doc_id", "source",
#                        "full_content", "score"}]}   # score: больше — ближе
# entry:   {"database", "id", "description", "other", "full_content", "skip"}
```

Порядок в `on_result` не окончателен: дедупликация, порог релевантности, ограничение количества и публикуемый порядок (по score, затем по идентификатору документа) применяются **после** хука. Хук не срабатывает, когда извлекать было нечего: пустой запрос, пустая коллекция и поиск без совпадений отвечают собственным ответом сервера.

**Таблица `QUERY_ALIASES`** — `{термин: замена}`. Применяется к нормализованному запросу до эмбеддинга, по целым словам и без учёта регистра: `ТЗ` не сработает внутри `ТЗначение`. Кода плагина на этом пути нет.

---

## SyntaxCheckServer

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/1c_syntaxcheck_mcp` / 8002 |
| `PRODUCT` | `bsl-syntax-check` |
| Версии | `HOST_CONTRACT_VERSION = 1`, **`HOOKS_VERSION = 2`** |
| Каталог | `/app/plugins` (`PLUGINS_DIR`; пустое значение = `/app/plugins`) |
| Интроспекция | MCP-инструменты `plugin_state`, `plugin_reload` |
| Dry-run | `python mcp_server.py --dry-run <файл>` |
| Производное состояние | Отсутствует: каждый вызов самодостаточен |

{% hint style="info" %}
Все хуки этого сервера — call-scoped, потому что между вызовами он ничего не строит и не хранит: код пишется во временный каталог, анализатор отрабатывает, каталог удаляется. Поэтому здесь нет отпечатков годности, пересборок и курсоров продолжения. Имя derived-state хука другого продукта отклонит загрузку файла.
{% endhint %}

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | Один раз, после проверки лицензии, до обслуживания | Ничего |
| `on_request(request)` | call-scoped | Каждый вызов, после валидации аргументов, до запуска `bsl-analyzer` | Только `arguments`; `tool` на возврате игнорируется |
| `on_diagnostics(diagnostics, request)` | call-scoped | Разобранные диагностики события `file`, до подавления и фильтра строк | Список диагностик |
| `on_result(result, request)` | call-scoped | Собранный поток событий, до применения контракта ответа | `events` |

```python
# request: {"tool": "syntaxcheck",
#           "arguments": {"code": str, "file_name": str}}   # file_name = "" если не задан
#          {"tool": "syntaxcheck_file",                      # только при смонтированном FILES_DIR
#           "arguments": {"file_path": str, "lines": str}}
# result:  {"events": [{"type": "start", ...},
#                      {"type": "file", "path", "diagnostics": [...], "metrics": {...}},
#                      {"type": "done", "total_files", "total_diagnostics", ...}]}
```

{% hint style="warning" %}
**Номера строк в payload нулевые.** `bsl-analyzer` выдаёт `start_line`/`end_line` с нуля, а публикуемая база применяется уже после хуков. Сортировка, сравнение и фильтрация внутри хука работают в тех числах, которые хук получил.

**`file_name` присутствует в каждом запросе `syntaxcheck`** и равен пустой строке, если вызывающая сторона его не задала. Хук, который пересобирает словарь аргументов, обязан вернуть оба имени: результат, чьи имена аргументов отличаются от полученных, отбрасывается. Именно из-за появления этого ключа `HOOKS_VERSION` поднят с 1 до 2 — плагин, закрепивший старую версию, не загрузится с указанием обеих версий.
{% endhint %}

**Таблица `SUPPRESSED_DIAGNOSTICS`** — `{код диагностики: причина подавления}`. Применяется после `on_diagnostics`, до фильтра строк и пересчёта `done`, поэтому подавление окончательно: хук не может вернуть то, что убрала таблица. Причина показывается в `plugin_state` — исчезнувшая из отчёта диагностика остаётся видимым решением. Таблица формирует отчёт, но **не перенастраивает анализатор**: `bsl-analyzer` запускается с той же зашитой конфигурацией. Код, выключенный в поставляемом `bsl-analyzer.toml`, отклоняется при загрузке — анализатор такую диагностику всё равно не выдаст.

---

## TemplatesSearchServer

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/template-search-mcp` / 8004 |
| `PRODUCT` | `template-search` |
| Версии | `HOST_CONTRACT_VERSION = 1`, `HOOKS_VERSION = 1` |
| Каталог | `/app/plugins` (`PLUGIN_DIR`) |
| Интроспекция | `plugin_state`; перезагрузка `plugin_reload` — изменяющий инструмент |
| Dry-run | `python main.py --dry-run <файл>` (прогоняет оба набора фикстур и проверки таблиц продукта) |
| Цена derived-state правки | Пересборка векторных коллекций шаблонов и заметок; отпечаток хранится в `index_meta.json` |

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | Один раз, после проверки лицензии, до инициализации базы | Ничего |
| `on_request(request)` | call-scoped | Каждый вызов MCP-инструмента, после валидации аргументов | `arguments` |
| `on_rank(results, request)` | call-scoped | `templatesearch` — после объединения и дедупликации маршрутов; `recall` — после порога релевантности | Порядок и состав записей |
| `on_result(result, request)` | call-scoped | Каждый вызов, на структурном значении до рендеринга | Структурное значение |
| `on_template(template)` | **derived-state** | Один раз на шаблон перед эмбеддингом: и при полной сборке, и на write-through пути `add_template` | `description`, `document`, `skip` |
| `on_memory(memory)` | **derived-state** | Один раз на заметку перед эмбеддингом: и при полной сборке, и на write-through пути `remember` | `content`, `document`, `skip` |

```python
# инструменты: templatesearch, list_templates, get_template,
#              add_template, remember, recall
# template: {"id", "description", "code", "document", "skip"}
# memory:   {"id", "content", "document", "skip"}
```

Изменили `description` (или `content`) — документ пересчитывается из него, и вектор считается по вашему тексту. Изменили ещё и `document` — эмбеддится ровно он. `skip = True` оставляет строку в SQLite (`get_template` и `list_templates` её отдают), но вектора для неё не пишет.

Две вещи, которые нельзя вывести из кода:

* **Веб-интерфейс не вызывает request-хуки.** Шаблоны и заметки, созданные, изменённые или удалённые через веб-интерфейс `/extend`, не проходят через `on_request` и `on_result` — те срабатывают только на вызовах MCP-инструментов. При этом записи веб-интерфейса **доходят** до `on_template` и `on_memory`: путь индексации общий.
* **На write-through пути падение хука не теряет строку.** Заметка индексируется неизменённой, `remember` отвечает успехом, сбой считается в `plugin_state`.

**Таблица `QUERY_ALIASES`** — `{термин: замена}`, применяется по целым словам к нормализованному запросу `templatesearch` и `recall` до запуска обоих маршрутов поиска. Ключи и значения — непустые строки. Два файла могут объявлять таблицу (записи применяются в порядке файлов), но разные значения для одного ключа отклоняются с указанием обоих файлов.

---

## Graph Metadata Search

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/1c_graph_metadata` / 8006 |
| `PRODUCT` | `graph-metadata-search` |
| Версии | `HOST_CONTRACT_VERSION = 1`, `HOOKS_VERSION = 1` |
| Каталог | `plugins` относительно `/app` (`GRAPH_PLUGINS_DIRECTORY`) |
| Включение | **`GRAPH_PLUGINS_ENABLED=true`** — иначе каталог не читается |
| Интроспекция | `list_plugins` (оба профиля), `reload_plugins(operation_id)` (только профиль `admin`) |
| Dry-run | `python run.py plugin-dry-run <файл>` — без Neo4j, метаданных, поколения и лицензии |
| Цена derived-state правки | Пересборка проекта: граф, полнотекстовый и векторный маршруты |

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | Один раз, когда прочитаны настройки | Ничего; секреты приходят булевыми «задано / не задано» |
| `on_request(request)` | call-scoped | Каждый вызов, после связывания и валидации аргументов, до тела инструмента | Только `arguments` |
| `on_result(result, request)` | call-scoped | После тела инструмента, до применения бюджета ответа | `items`, `nodes`, `edges`, `text`, `data`, `warnings`, `degraded`, `evidence` |
| `on_search_candidates(candidates, query, tool)` | call-scoped | В единственной точке схождения ранжирования: после извлечения, слияния, дедупликации и переранжирования, до отсечения `top_k` | Порядок и подмножество кандидатов |
| `on_source_unit(unit)` | **derived-state** | Один раз на файл инвентаря проекта, после классификации и хеширования | Только `skip` |
| `on_metadata_object(obj)` | **derived-state** | Один раз на разобранный объект метаданных, до записи в граф | Только `properties` (примитивы и списки примитивов) |
| `on_routine(routine)` | **derived-state** | Один раз на процедуру или функцию BSL, до попадания в граф кода | `description`, `body`, `is_export`, `is_function`, `directive`, `calls`, `extension_mode`, `extension_target` |
| `on_embedding_document(document)` | **derived-state** | Один раз на собранный текст эмбеддинга, до нарезки на чанки | Только `text` |

```python
# request:    {"tool", "arguments", "project_id", "mcp_namespace",
#              "generation", "correlation_id"}   # всё кроме arguments — контекст
# result:     {"items", "nodes", "edges", "graph", "text", "data",
#              "total", "warnings", "degraded", "evidence", "request"}
# unit:       {"source_unit_id", "root", "normalized_path", "artifact_kind",
#              "encoding", "size_bytes", "source_hash", "structural_hash",
#              "semantic_hash", "parser_fingerprint", "lanes", "skip"}
# obj:        {"name", "category", "properties", "attributes",
#              "tabular_parts", "forms"}
# routine:    {"name", "signature", "description", "body", "is_export",
#              "is_function", "directive", "line_number", "end_line",
#              "calls", "extension_mode", "extension_target", "module_path"}
# document:   {"kind": "business" | "description", "node_id", "name",
#              "category", "text"}
```

`on_search_candidates` — единственный способ переупорядочить ответ инструмента, который отвечает отрендеренным текстом, а не записями. Переписанный `body` в `on_routine` перечитывается на вызовы, если сам `calls` не менялся, — граф кода остаётся согласован с текстом, который он описывает. `total` в `on_result` пересчитывается из `items`, если плагин их менял.

**Таблицы**

| Таблица | Форма | Действие |
|---------|-------|----------|
| `ALIASES` | `{имя: существующая ссылка}` | Расширяет разрешение ссылок так же, как это делает свойство `Синоним`. Ссылка — FQN (`Справочник.Контрагенты`), квалифицированное имя, UUID или id узла. Алиас, который пока ни во что не разрешается, принимается и показывается как неразрешённый в `list_plugins` |
| `TOOL_PRESETS` | список `{"name", "tool", "arguments", "summary"}` | Новый публикуемый инструмент, фиксирующий аргументы уже публикуемого. Пресет наследует контракт связанного инструмента без изменений и не может назвать инструмент, которого активный профиль не публикует |
| `CYPHER_TEMPLATES` | список `{"id", "description", "cypher"}` | Дополнительные read-only Cypher-шаблоны для `run_graph_cypher_template`. Проходят те же проверки, что встроенные: без пишущих и административных конструкций, с возможностью ограничения областью. Область подставляет сервер, она не бывает аргументом шаблона |

{% hint style="warning" %}
У этого сервера есть бюджет времени на один hook: `GRAPH_PLUGIN_HOOK_TIMEOUT_SECONDS` (по умолчанию `5.0`, `0` отключает контроль).
{% endhint %}
