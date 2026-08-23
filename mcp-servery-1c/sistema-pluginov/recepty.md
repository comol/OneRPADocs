# Рецепты

Готовые примеры доработки. Каждый — один файл, который кладётся в каталог плагинов соответствующего сервера. Проверяйте [dry-run](kak-napisat-plugin.md) перед развёртыванием.

## Чем решать задачу

| Задача | Чем решать | Цена |
|--------|-----------|------|
| Свои сокращения и термины в запросах | Таблица `ALIASES` / `QUERY_ALIASES` | Перезагрузка |
| Готовый инструмент с зафиксированными аргументами | Таблица `TOOL_PRESETS` | Перезагрузка |
| Не показывать часть диагностик анализатора | Таблица `SUPPRESSED_DIAGNOSTICS` | Перезагрузка |
| Переписать запрос перед поиском | `on_request` (call-scoped) | Перезагрузка |
| Переупорядочить или отфильтровать выдачу | `on_result` / `on_rank` / `on_search_candidates` | Перезагрузка |
| Изменить то, что попадает в индекс | derived-state хук | **Пересборка индекса** |
| Выбросить документы или строки из индекса | derived-state хук, `skip = True` | **Пересборка индекса** |

Правило простое: если ответ не зависит от вызова — таблица; если зависит — call-scoped хук; и только если менять нужно само сохранённое состояние — derived-state.

## HelpSearchServer: термины компании и пресет инструмента

Таблицы и call-scoped хук — пересборки не требуют.

```python
"""Термины компании и готовый поиск по стандартам. /app/plugins/10-terms.py"""

REQUIRES = {"host": 1, "product": "1c-help", "hooks": 1}

#: Дополнительные имена, которые разрешает docinfo.
ALIASES = {
    "ТЗ": "ТаблицаЗначений",
    "РС": "РегистрСведений",
}

#: Отдельный инструмент — пресет docsearch с зафиксированным doc_type.
TOOL_PRESETS = [
    {
        "name": "search_structures",
        "description": "Поиск по страницам структур справки платформы.",
        "tool": "docsearch",
        "bind": {"doc_type": "structure"},
    },
]


def on_result(result, request):
    """Свои стандарты — первыми в выдаче."""
    result["items"].sort(
        key=lambda item: not item["citation"].get("source", "").startswith("rules/"),
    )
    return result
```

## HelpSearchServer: разметить и выбросить документы при сборке

Derived-state: правка этого файла делает сохранённый индекс устаревшим, и следующий старт пересобирает его в staging-поколение, пока текущее продолжает отвечать.

```python
"""Разметка собственных стандартов. /app/plugins/20-standards.py"""


def on_document(doc):
    if doc["corpus"] != "project-standards":
        return
    if doc["source"].startswith("rules/draft-"):
        doc["skip"] = True          # черновики в индекс не попадают
        return doc
    doc["metadata"]["doc_type"] = "structure"
    return doc
```

## SSLSearchServer: синонимы и вытеснение устаревшего

```python
"""Наш словарь и наша библиотека вместо устаревших процедур БСП."""

REQUIRES = {"host": 1, "product": "ssl-search", "hooks": 1}

QUERY_ALIASES = {
    "ЗУП": "Зарплата и управление персоналом",
    "РС": "РегистрСведений",
}


def on_result(result, request):
    """Убрать из выдачи то, что заменено собственной библиотекой."""
    result["entries"] = [
        entry for entry in result["entries"]
        if "УстаревшиеПроцедуры" not in entry["full_content"]
    ]
    return result
```

Выбросить записи ещё на этапе индексации (derived-state, стоит переэмбеддирования выбранной базы БСП):

```python
def on_entry(entry):
    if entry["description"].startswith("Устарела."):
        entry["skip"] = True
        return entry
    if "ЗарплатаКадры" in entry["description"]:
        entry["description"] += " ЗУП кадровый учёт"   # ищется по нашему названию
        return entry
```

## SyntaxCheckServer: корпоративный профиль проверок

Таблица подавления окончательна и не выполняет кода; `on_diagnostics` добавляет то, что зависит от вызова.

```python
"""Профиль проверок компании. /app/plugins/10-profile.py"""

REQUIRES = {"host": 1, "product": "bsl-syntax-check", "hooks": 2}

#: Код диагностики -> почему она не попадает в отчёт. Причина видна в plugin_state.
SUPPRESSED_DIAGNOSTICS = {
    "CanonicalSpellingKeywords": "регистр ключевых слов у нас не контролируется",
    "LineLength": "",
}


def on_diagnostics(diagnostics, request):
    """В общих модулях показываем только ошибки, замечания опускаем."""
    path = request["arguments"].get("file_path", "")
    if not path.startswith("CommonModules/"):
        return
    return [entry for entry in diagnostics if entry["severity"] == "Error"]
```

{% hint style="info" %}
Номера строк в payload нулевые: публикуемая база применяется после хуков. Сравнивайте и сортируйте в тех числах, которые получили.
{% endhint %}

## TemplatesSearchServer: свой префикс запроса и метка отдела

```python
"""Шаблоны: чистим запрос и метим шаблоны отдела."""

REQUIRES = {"host": 1, "product": "template-search", "hooks": 1}

QUERY_ALIASES = {"ТЗ": "ТаблицаЗначений"}


def on_request(request):
    """Наши агенты добавляют префикс «1С:» — он мешает поиску."""
    if request["tool"] == "templatesearch":
        query = request["arguments"]["query"]
        if query.startswith("1С:"):
            request["arguments"]["query"] = query[3:].strip()
            return request
```

Derived-state вариант — эмбеддить шаблоны вместе с названием подсистемы (стоит пересборки коллекций):

```python
def on_template(template):
    if "РегистрыСведений." in template["code"]:
        template["description"] += " регистры сведений"
        return template
```

## Graph Metadata Search: свои факты об объектах и свой Cypher

Не забудьте `GRAPH_PLUGINS_ENABLED=true` — иначе каталог не читается.

```python
"""Разметка объектов и шаблон запроса. /app/plugins/10-facts.py"""

REQUIRES = {"host": 1, "product": "graph-metadata-search", "hooks": 1}

ALIASES = {"Контрагент": "Справочник.Контрагенты"}

CYPHER_TEMPLATES = [
    {
        "id": "objects_by_owner",
        "description": "Объекты в зоне ответственности отдела продаж.",
        "cypher": (
            "MATCH (m:MetadataObject) "
            "WHERE m.`ЗонаОтветственности` = 'ОтделПродаж' "
            "RETURN m.name AS name, m.category_name AS category ORDER BY name"
        ),
    },
]


def on_metadata_object(obj):
    """Пометить объекты зоны ответственности отдела — свойство уедет в граф."""
    if obj["category"] == "Документы" and obj["name"].startswith("Заказ"):
        obj["properties"]["ЗонаОтветственности"] = "ОтделПродаж"
        return obj


def on_search_candidates(candidates, query, tool):
    """Свои объекты — выше, до отсечения top_k."""
    return sorted(
        candidates,
        key=lambda item: not str(item.get("name", "")).startswith("Наш"),
    )
```

Хук проставляет свойство при сборке графа, шаблон его читает — вместе это даёт инструменту `run_graph_cypher_template` запрос в терминах вашей организации.

{% hint style="warning" %}
`on_metadata_object` — derived-state: значения `properties` должны быть примитивами или списками примитивов (Neo4j другого не хранит), а правка файла вызывает пересборку проекта. `ALIASES`, `CYPHER_TEMPLATES` и `on_search_candidates` — call-scoped, им достаточно `reload_plugins`.
{% endhint %}

## Композиция нескольких файлов

Файлы выполняются в отсортированном порядке имён, и каждый хук — конвейер: что вернул `10-terms.py`, то получит `20-filter.py`. Отсюда практика именования:

```
/app/plugins/
    10-terms.py       # словари и переписывание запроса
    20-index.py       # derived-state правки
    90-ranking.py     # финальная сортировка выдачи
```

Действующий порядок всегда виден в состоянии подсистемы (`plugin_state`, `list_plugins`, `GET /plugins`) — вместе с эпохой, ошибками и накопленным временем каждого хука.
