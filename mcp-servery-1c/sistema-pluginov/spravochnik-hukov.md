# Справочник хуков по серверам

Набор хуков объявляет каждый сервер сам. Ниже — что предлагает каждый из семи прикладных серверов с плагинами: имена, классификация, момент срабатывания, форма payload и какие поля применяются из возвращённого значения.

{% hint style="info" %}
Эта страница — карта. Полный справочник лежит внутри образа: обычно `/app/plugin_api.py`, у CodeMetadataSearchServer — `/app/src/plugin_api.py`, у 1CCodeChecker — `/app/MCP_1copilot/plugin_api.py`. В нём для каждого хука и таблицы есть форма payload и рабочий пример.
{% endhint %}

## Сводная таблица хуков

| Хук | Help | SSL | Syntax | Templates | Graph | Code | Checker |
|-----|------|-----|--------|-----------|-------|------|---------|
| `on_startup` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `on_request` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `on_result` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `on_query` | ✅ | — | — | — | — | — | — |
| `on_upstream_call` | — | — | — | — | — | — | ✅ |
| `on_diagnostics` | — | — | ✅ | — | — | — | — |
| `on_rank` | — | — | — | ✅ | — | — | — |
| `on_search_candidates` | — | — | — | — | ✅ | ✅ | — |
| `on_document` *(derived)* | ✅ | — | — | — | — | — | — |
| `on_entry` *(derived)* | — | ✅ | — | — | — | — | — |
| `on_template`, `on_memory` *(derived)* | — | — | — | ✅ | — | — | — |
| `on_source_unit`, `on_metadata_object`, `on_routine`, `on_embedding_document` *(derived)* | — | — | — | — | ✅ | — | — |
| `on_source_file`, `on_chunk`, `on_metadata_object` *(derived)* | — | — | — | — | — | ✅ | — |
| Таблицы | `ALIASES`, `TOOL_PRESETS` | `QUERY_ALIASES` | `SUPPRESSED_DIAGNOSTICS` | `QUERY_ALIASES` | `ALIASES`, `TOOL_PRESETS`, `CYPHER_TEMPLATES` | `QUERY_ALIASES`, `TOOL_PRESETS` | `TOOL_PRESETS` |

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
#                         "max_chars", "max_items", "detail_level", "cursor", "diagnostics"}}
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



---

## 1CCodeChecker

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/1c-code-checker:latest-beta` / 8007 |
| `PRODUCT` | `onec-code-checker` |
| Версии | `HOST_CONTRACT_VERSION = 1`, `HOOKS_VERSION = 1` |
| Каталог | `/app/plugins` (`PLUGIN_DIR`) |
| Справочник в образе | `/app/MCP_1copilot/plugin_api.py` |
| Интроспекция | `GET /plugins`, перезагрузка `POST /plugins/reload` |
| Dry-run | `python -m MCP_1copilot --dry-run <файл>` — без токена, лицензии и сети |
| Цена правки | Только перезагрузка: derived-state и индекса у сервера нет |

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | После проверки лицензии и конфигурации, до открытия порта | Ничего; секреты заменены маркерами |
| `on_request(request)` | call-scoped | Каждый вызов после собственных проверок сервера, до ограничения длины полей | `arguments` целиком, с теми же именами и типами |
| `on_upstream_call(call, request)` | call-scoped | Непосредственно перед уходом запроса в `code.1c.ai`; повторно перед fallback-промптом | Для prompt-пути — `prompt`; для direct-пути — `arguments`. Нельзя сменить `mode`, `capability` или `upstream_tool` |
| `on_result(result, request)` | call-scoped | После ответа upstream и очистки, до типизированного конверта | Только `answer`; доказательства усечения, diff и `safe_to_apply` вычисляются заново |

`on_upstream_call` — последняя точка перед внешней границей. Через неё проходят исходный код и параметры сессии, поэтому плагин может удалить корпоративные данные или добавить стандарт компании, но не должен записывать payload в собственный журнал. Перезапись, которая расширяет поле сверх `ONEC_AI_INPUT_MAX_LENGTH` или пытается перенаправить capability, отбрасывается целиком.

`ONEC_AI_TOKEN` и `LICENSE_KEY` не попадают ни в payload, ни в журнал, ни в `/plugins`. Если плагин изменил вызов, ответ сообщает имя файла и какие аргументы изменились; длинные значения показываются длиной и SHA-256-префиксом, а не исходным текстом.

**Таблица `TOOL_PRESETS`** — список `{"name", "description", "tool", "bind"}`. Каждый элемент публикует дополнительный read-only MCP-инструмент как пресет одного из 11 штатных с фиксированными аргументами. Связанный аргумент исчезает из схемы пресета и не может быть переопределён вызывающей стороной.

## CodeMetadataSearchServer

| Параметр | Значение |
|----------|----------|
| Образ / порт | `comol/1c_code_metadata_mcp` / 8000 |
| `PRODUCT` | `code-metadata-search` |
| Версии | `HOST_CONTRACT_VERSION = 1`, `HOOKS_VERSION = 1` |
| Каталог | `/app/plugins` (`PLUGIN_DIR`) |
| Справочник в образе | `/app/src/plugin_api.py` — **не** `/app/plugin_api.py`, как у остальных серверов |
| Интроспекция | MCP-инструменты `plugin_state`, `plugin_reload` |
| Dry-run | `python src/plugin_dry_run.py <файл>` — без смонтированных данных, построенного индекса, векторного хранилища, провайдера эмбеддингов и лицензионного ключа |
| Цена derived-state правки | Сборка нового поколения индекса; опубликованное продолжает отвечать. Принудительно: `reindex(force=true)` |

| Хук | Класс | Когда срабатывает | Что применяется |
|-----|-------|-------------------|-----------------|
| `on_startup(config)` | call-scoped | Один раз, до того как что-либо проиндексировано или обслужено | Ничего: возврат игнорируется. Лицензионный ключ и учётные данные провайдера эмбеддингов приходят строкой `<redacted>`, когда заданы, и `None`, когда нет |
| `on_request(request)` | call-scoped | Каждый вызов публикуемого инструмента, после валидации аргументов | Только `arguments`; результат валидируется заново |
| `on_search_candidates(candidates, query, lane)` | call-scoped | Ранжированные кандидаты одной дорожки, до дедупликации по источникам и до отсечения по `limit` | Порядок и подмножество кандидатов. Поднятый здесь элемент может попасть на страницу, с которой был бы срезан, — этого не даёт `on_result` |
| `on_result(result, request)` | call-scoped | Каждый результат инструмента, до применения бюджета ответа | Только `data` |
| `on_source_file(unit)` | **derived-state** | Сборка, один раз на файл, принятый обходом конфигурации, до чтения файла | Только `skip` |
| `on_chunk(chunk)` | **derived-state** | Сборка, один раз на чанк, до вывода его идентичности и эмбеддинга | `text`, `metadata`, `skip` |
| `on_metadata_object(obj)` | **derived-state** | Сборка, один раз на разобранный объект метаданных, до записи в хранилище | Только `properties` и `skip` |

```python
# config:     словарь настроек; секреты — "<redacted>" либо None
# request:    {"tool": <имя публикуемого инструмента>, "arguments": {...}}
# candidates: [{"document", "metadata", "distance", "combined_score"}]
#             lane: "metadata" | "code" | "help" | "forms"
# result:     {"tool", "data": {"results": [...], "search_layer", "origin_filter"}}
# unit:       {"path", "canonical_path", "kind", "origin_kind", "origin_id",
#              "owning_root", "skip"}
# chunk:      {"lane", "path", "canonical_path", "ordinal", "text",
#              "metadata", "skip"}
# obj:        {"full_path", "object_type", "name", "properties",
#              "origin_kind", "origin_id", "skip"}
```

`on_chunk` определяет идентичность чанка: стабильный id выводится из возвращённого текста, поэтому недетерминированное преобразование (отметка времени, случайное число, порядок обхода словаря) даёт новый id на каждом прогоне и переэмбеддирует всю конфигурацию заново. `on_result` не может расширить границу: он срабатывает до бюджета ответа, после которого `max_chars` измеряется на итоговой сериализованной полезной нагрузке, применяется `max_items`, а счётчики и курсор пересчитываются по тому, что ответ фактически несёт. В `on_source_file` `skip` — единственное принимаемое поле: остальные называют файл на диске. В `on_metadata_object` идентичность (`full_path`, `object_type`, `name`) только для чтения — её правка осиротила бы всё, что на неё ссылается.

Упавший derived-state хук по умолчанию не роняет сборку: единица индексируется без изменений, ошибка считается и попадает в сводку сборки. При `PLUGIN_STRICT_DERIVED_STATE=true` сборка падает целиком, а опубликованное поколение продолжает обслуживать запросы.

**Таблицы**

| Таблица | Форма | Действие |
|---------|-------|----------|
| `QUERY_ALIASES` | `{алиас: замена}` | Замены в аргументе `query` каждого поискового инструмента, после `on_request` и до извлечения, по целым словам и без учёта регистра — `ТЗ` не срабатывает внутри `ТЗначение`. Кода плагина на этом пути не выполняется. Два файла, отображающие один алиас в разные замены, отклоняют загрузку второго с указанием обоих файлов |
| `TOOL_PRESETS` | список `{"name", "tool", "description", "bind"}` | Новый MCP-инструмент как пресет уже публикуемого с зафиксированными аргументами. `tool` должен называть публикуемый инструмент, каждый ключ `bind` — его параметр, `name` не должен совпадать с публикуемым инструментом или другим пресетом. Связанный аргумент фиксирован: вызывающая сторона не может его переопределить. Пресеты переопубликовываются на `plugin_reload`, перезапуск не нужен |

{% hint style="warning" %}
У этого сервера есть бюджет времени на один hook: `PLUGIN_HOOK_TIMEOUT_SECONDS` (по умолчанию `5.0`). Превысивший его хук считается упавшим, call-scoped отключается до следующей перезагрузки каталога.
{% endhint %}
