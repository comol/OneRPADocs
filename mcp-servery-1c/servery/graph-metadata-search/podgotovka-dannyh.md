# Подготовка данных для Graph Metadata Search

Для работы Graph Metadata Search нужен хотя бы один источник метаданных. Рекомендуемый вариант — Designer XML-выгрузка из Конфигуратора: по ней сервер строит BSL-граф и доменные связи, а при отсутствии текстового отчёта автоматически синтезирует совместимый отчёт для базового контура метаданных.

## Необходимые выгрузки

| Выгрузка | Назначение | Обязательность |
|----------|------------|----------------|
| Designer XML-выгрузка в файлы | Метаданные, BSL-код, структура форм, подписки, роли, предопределённые, справка; из неё может быть синтезирован текстовый отчёт | **Рекомендуется**, достаточна сама по себе |
| Отчёт из конфигурации | Готовый источник базовых метаданных; сохраняется для совместимости и обязателен только при `METADATA_SOURCE=report` или для проекта 1C:EDT | Опционально |
| Проект 1C:EDT | Альтернативный источник кода и структуры при `SOURCE_FORMAT_ADAPTERS_ENABLED=true`; автоматический синтез текстового отчёта из EDT не поддерживается | Опционально, нужен отдельный отчёт |

## Шаг 1: Создание папок

```powershell
New-Item -ItemType Directory -Force -Path @(
    "E:\1C_Export\Files",
    "E:\bases\graph_generated_report"
)
```

## Шаг 2: Выгрузка в файлы

Это основной источник для BSL-графа, структуры форм XML, подписок на события, предопределённых элементов, прав ролей и справки. При `METADATA_SOURCE=auto` он также становится источником базовых метаданных, если готового отчёта нет.

1. Меню **Конфигурация** → **Выгрузить конфигурацию в файлы**
2. Укажите папку `E:\1C_Export\Files`

## Шаг 3: Необязательный готовый отчёт

Если политика установки требует использовать именно готовый отчёт, выполните **Конфигурация → Отчёт из конфигурации**, включите все разделы и сохраните `*.txt` в отдельный каталог. Задайте `METADATA_SOURCE=report`. При `auto` готовый отчёт имеет приоритет над синтезом; при `xml` он намеренно игнорируется.

{% hint style="info" %}
Синтез выполняется в фоновой стадии `metadata_ingest` и кэшируется по отпечатку Designer XML-выгрузки. Каталог `GENERATED_REPORT_DIRECTORY` должен быть отдельным и не находиться внутри `CODE_EXPORT_PATH`.
{% endhint %}

## Граф метаданных

После индексации в Neo4j создаётся граф с разветвлённой структурой.

### Типы узлов

#### Базовые узлы (из готового или синтезированного отчёта)

| Узел | Описание |
|------|----------|
| **Project** | Корневой узел, идентификация проекта |
| **Configuration** | Конфигурация |
| **MetadataCategory** | Категория метаданных (Справочники, Документы, ...) |
| **MetadataObject** | Объект метаданных |
| **Attribute** | Реквизит объекта |
| **TabularPart** | Табличная часть объекта |
| **Form** | Форма объекта |
| **EnumValue** | Значение перечисления |
| **Command** | Команда объекта |
| **Layout** | Макет объекта |

#### Специализированные узлы (из готового или синтезированного отчёта)

| Узел | Описание |
|------|----------|
| **UrlTemplate** | Шаблон URL HTTP-сервиса |
| **UrlMethod** | Метод URL-шаблона (GET, POST, ...) |
| **Characteristic** | Характеристика плана видов характеристик |
| **JournalGraph** | Граф журнала документов |
| **AccountingFlag** | Признак учёта плана счетов |
| **DimensionAccountingFlag** | Признак учёта субконто |

#### BSL-граф (из выгрузки в файлы, `LOAD_BSL_SIGNATURES=true`)

| Узел | Описание |
|------|----------|
| **Module** | Модуль объекта (ObjectModule, ManagerModule, FormModule, CommonModule, ...) |
| **Routine** | Процедура или функция BSL-кода |

#### XML-данные (из выгрузки в файлы)

| Узел | Описание | Переменная |
|------|----------|------------|
| **EventSubscription** | Подписка на событие | `LOAD_EVENT_SUBSCRIPTIONS=true` |
| **PredefinedItem** | Предопределённый элемент | `LOAD_PREDEFINED_VALUES=true` |
| **FormControl** | Элемент управления формы | `LOAD_FORMS_FROM_XML=true` |
| **FormEvent** | Событие формы или элемента | `LOAD_FORMS_FROM_XML=true` |
| **FormAttribute** | Реквизит формы | `LOAD_FORMS_FROM_XML=true` |

#### Системные узлы

| Узел | Описание |
|------|----------|
| **SystemMeta** | Системная информация (embedding_model_id, timestamps) |

### Связи

#### Иерархия

| Связь | Описание |
|-------|----------|
| `HAS_CONFIGURATION` | Project → Configuration |
| `HAS_CATEGORY` | Configuration → MetadataCategory |
| `HAS_OBJECT` | MetadataCategory → MetadataObject |
| `HAS_ATTRIBUTE` | MetadataObject (или TabularPart) → Attribute |
| `HAS_TABULAR_PART` | MetadataObject → TabularPart |
| `HAS_FORM` | MetadataObject → Form |
| `HAS_VALUE` | MetadataObject (Перечисление) → EnumValue |
| `HAS_COMMAND` | MetadataObject (или Form) → Command |
| `HAS_LAYOUT` | MetadataObject → Layout |
| `HAS_URL_TEMPLATE` | MetadataObject (HTTP-сервис) → UrlTemplate |
| `HAS_URL_METHOD` | UrlTemplate → UrlMethod |
| `HAS_CHARACTERISTIC` | MetadataObject → Characteristic |
| `HAS_GRAPH` | MetadataObject → JournalGraph |
| `HAS_ACCOUNTING_FLAG` | MetadataObject → AccountingFlag |
| `HAS_DIMENSION_ACCOUNTING_FLAG` | MetadataObject → DimensionAccountingFlag |

#### Перекрёстные связи объектов

| Связь | Описание |
|-------|----------|
| `USED_IN` | Объект используется как тип реквизита/ресурса/измерения в другом объекте |
| `DO_MOVEMENTS_IN` | Документ делает движения в регистр |
| `CONTAINS_OBJECT` | Подсистема содержит объект |

#### BSL-граф

| Связь | Описание |
|-------|----------|
| `HAS_MODULE` | MetadataObject → Module |
| `DECLARES` | Module → Routine |
| `CALLS` | Routine → Routine (вызов процедуры) |

#### XML-данные

| Связь | Описание |
|-------|----------|
| `HAS_PREDEFINED` | MetadataObject → PredefinedItem |
| `HAS_CHILD` | PredefinedItem → PredefinedItem (иерархия) |
| `HAS_HANDLER` | EventSubscription (или FormEvent) → Routine |
| `HAS_CONTROL` | Form → FormControl |
| `HAS_CHILD` | FormControl → FormControl (иерархия элементов) |
| `HAS_EVENT` | Form (или FormControl) → FormEvent |
| `HAS_FORM_ATTRIBUTE` | Form → FormAttribute |
| `BINDS_TO` | FormControl → цель привязки данных |
| `LINKS_TO_COMMAND` | FormControl → Command |
| `GRANTS_ACCESS_TO` | Role → MetadataObject (с деталями прав) |

#### Расширения

| Связь | Описание |
|-------|----------|
| `EXTENDS` | MetadataObject расширения → MetadataObject базы |
| `OVERRIDES` | Элемент расширения (Form, Module, Routine) → элемент базы |

### Визуализация графа

```
Project (корневой узел)
  └─> Configuration
        └─> MetadataCategory (Справочники, Документы, ...)
              └─> MetadataObject
                    ├─> Attribute (реквизиты)
                    ├─> TabularPart → Attribute
                    ├─> Form
                    │     ├─> FormControl → FormControl (иерархия)
                    │     ├─> FormEvent → Routine (обработчик)
                    │     ├─> FormAttribute
                    │     └─> Command
                    ├─> Command
                    ├─> Layout
                    ├─> EnumValue
                    ├─> Module → Routine → Routine (CALLS)
                    ├─> PredefinedItem → PredefinedItem (иерархия)
                    ├─> EventSubscription → Routine (обработчик)
                    ├── USED_IN ──> другой MetadataObject
                    ├── DO_MOVEMENTS_IN ──> регистр
                    └── EXTENDS ──> базовый MetadataObject
```

## Просмотр графа

После индексации откройте Neo4j Browser (`http://localhost:7474`) и выполните запрос:

```cypher
MATCH (n) RETURN n LIMIT 100
```

Полезные запросы:

```cypher
// Все категории метаданных
MATCH (c:MetadataCategory) RETURN c.name

// Связи конкретного объекта
MATCH (o:MetadataObject {name: "Контрагенты"})-[r]->(n) RETURN o, r, n

// Граф вызовов процедуры
MATCH path = (r:Routine {name: "ПередЗаписью"})-[:CALLS*1..3]->(callee)
RETURN path

// Объекты расширения
MATCH (o:MetadataObject {origin: "extension"}) RETURN o.name, o.extension_name
```

## Обновление данных

При изменении конфигурации:

1. Повторите выгрузку
2. Установите `RESET_DATABASE=true` в docker-compose.yml
3. Перезапустите сервисы

```powershell
docker-compose down
docker-compose up -d
```

{% hint style="warning" %}
При смене embedding-модели также необходимо установить `RESET_DATABASE=true`. Система обнаружит несоответствие модели и не запустится без сброса.
{% endhint %}

## Мультипроект

Система поддерживает несколько конфигураций в одной базе Neo4j. Каждая конфигурация идентифицируется через `PROJECT_NAME`. При поиске результаты автоматически фильтруются по текущему проекту.

При `FULL_METADATA_RELOAD=true` удаляются только данные текущего проекта, данные других проектов не затрагиваются.
