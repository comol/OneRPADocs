# Серверы без плагинов

Два сервера из списка не поставляют систему плагинов: **CloudEmbeddingsServer** и **1CCodeChecker**. Это не значит, что их нельзя адаптировать — но рычаги у них другие: переменные окружения, состав индексируемых данных, конфигурация вышестоящего сервиса и правила на стороне клиента.

CodeMetadataSearchServer систему плагинов **поддерживает** — см. [Справочник хуков](spravochnik-hukov.md#codemetadatasearchserver). Рычаги окружения ниже остаются актуальными для него как более дешёвая альтернатива derived-state хуку.

## Как проверить, поддерживает ли образ плагины

Не полагайтесь на память версий — спросите образ:

```powershell
# Есть плагины: команда напечатает путь
docker run --rm comol/1c_help_mcp:latest ls /app/plugin_api.py

# CodeMetadataSearchServer держит справочник глубже — проверяйте оба пути
docker run --rm comol/1c_code_metadata_mcp:latest ls /app/src/plugin_api.py

# Нет плагинов: команда завершится ошибкой "No such file or directory"
docker run --rm comol/1c-code-checker:latest ls /app/plugin_api.py /app/src/plugin_api.py
```

Если файл есть — читайте его: это полный контракт, и дальше действует [общая система плагинов](README.md). Если файла нет — смотрите разделы ниже.

{% hint style="warning" %}
Проверять только `/app/plugin_api.py` недостаточно: у CodeMetadataSearchServer справочник лежит в `/app/src/plugin_api.py`, и такая проверка даст для него ложное «нет плагинов».
{% endhint %}

## CodeMetadataSearchServer — рычаги окружения

Поиск по метаданным и коду конфигурации. Плагины сервер поддерживает, но и без них его поведение задаётся тем, **что индексируется** и **как ранжируется** — и это дешевле, чем derived-state хук.

| Рычаг | Переменные | Что даёт |
|-------|------------|----------|
| Состав данных | `METADATA_PATH`, `CODE_PATH`, `INDEX_NESTED_CONFIGURATIONS`, `NESTED_CONFIGURATION_PATHS` | Главный рычаг: сервер ищет ровно по тому, что вы смонтировали. Ограничьте выгрузку — сузите и ускорите поиск |
| Баланс поиска | `BM25_ALPHA` (вес семантики против полнотекста), `MIN_SCORE_THRESHOLD` | Смещение выдачи в сторону точных имён или смысла |
| Глубина выборки | `OVERFETCH_MULTIPLIER`, `SEMANTIC_OVERFETCH_MULTIPLIER` | Сколько кандидатов рассматривается до отсечения |
| Переранжирование | `ENABLE_RERANKER`, `RERANKER_MODEL`, `RERANKER_TOP_K` | Cross-encoder поверх выдачи, когда важнее качество, чем скорость |
| Профиль векторного хранилища | `VECTOR_PROFILE` (`fast_index`, `balanced`, `memory_saver`, `quality`) | Компромисс «скорость сборки / память / качество» |
| Обновление индекса | `INCREMENTAL_INDEXING`, `REINDEX_INTERVAL_SEC`, `BACKGROUND_INDEXING`, `GENERATION_RETENTION_COUNT` | Как часто и как дорого обновляется индекс |
| Эмбеддинги | `EMBEDDING_MODEL`, `EMBEDDING_API_BASE`, `EMBEDDING_DIMENSIONS`, `EMBEDDING_PROVIDER` | Качество семантического поиска |

Полный список: [Переменные окружения](../prilozhenia/peremennye-okruzheniya.md) и [Конфигурация CodeMetadataSearchServer](../servery/code-metadata-search/konfiguraciya.md).

{% hint style="info" %}
Смена модели эмбеддингов или размерности делает сохранённый индекс несовместимым и вызывает переиндексацию — по стоимости это сопоставимо с правкой derived-state хука у серверов с плагинами.
{% endhint %}

## CloudEmbeddingsServer

Индексация и поиск через внешний embedding-API. Плагинов нет; всё поведение задаётся окружением.

| Рычаг | Переменные | Что даёт |
|-------|------------|----------|
| Провайдер и модель | `EMBEDDING_PROVIDER`, `EMBEDDING_MODEL`, `OPENAI_API_BASE` и ключи провайдеров | Куда уходят тексты и каким качеством считаются векторы |
| Состав данных | `SOURCE_PATH`, `AUTO_INDEX` | Что именно индексируется при старте |
| Нарезка | `CHUNK_SIZE`, `CHUNK_OVERLAP` | Гранулярность найденного фрагмента — ближайший аналог «доработки индексации» |
| Пропускная способность | `EMBEDDING_CONCURRENCY`, `EMBEDDING_BATCH_SIZE`, `MAX_BATCH_SIZE` | Скорость индексации против нагрузки на API |
| Выдача | `DEFAULT_SEARCH_LIMIT` | Сколько результатов возвращается по умолчанию |

## 1CCodeChecker

Прокси к сервису 1С:Напарник: анализ выполняется на стороне 1С.ai, поэтому «доработка» здесь — это настройка того, **как формируется сессия** с вышестоящим сервисом.

| Рычаг | Переменные | Что даёт |
|-------|------------|----------|
| Режим вызова | `MCP_TOOL_CALL_MODE` (`direct` / `standard`), `ONEC_AI_SKILL_NAME` (`custom` / `raw`) | Прямые вызовы инструментов против промптов; наличие инструментов в сессии |
| Контекст платформы | `ONEC_AI_DOC_VERSION`, `ONEC_CONFIG_NAME`, `ONEC_AI_UI_LANGUAGE`, `ONEC_AI_SCRIPT_LANGUAGE` | Версия документации и конфигурация, в терминах которых отвечает сервис |
| Границы работы | `ONEC_AI_TIMEOUT`, `ONEC_AI_OPERATION_TIMEOUT`, `ONEC_AI_INPUT_MAX_LENGTH`, `MAX_ACTIVE_SESSIONS`, `SESSION_TTL` | Таймауты, размеры входа и количество одновременных дискуссий |

Подробности: [Конфигурация 1CCodeChecker](../servery/code-checker/konfiguraciya.md) и [Инструменты](../servery/code-checker/instrumenty.md).

## Если рычагов окружения не хватает

| Задача | Решение |
|--------|---------|
| Нужен свой инструмент с бизнес-логикой 1С | [Конструктор MCP серверов для 1С](../../konstruktor-mcp-serverov-1c/) — инструменты пишутся на встроенном языке и публикуются HTTP-сервисом 1С |
| Нужно поменять поведение ИИ, а не сервера | [Cursor Rules для 1С](../integraciya/cursor-rules.md) — правила задают, когда и как агент вызывает инструменты |
| Нужны свои пресеты инструментов и словари терминов | Разместите их на сервере, который поддерживает плагины: `TOOL_PRESETS` есть у [HelpSearchServer](spravochnik-hukov.md#helpsearchserver), [Graph Metadata Search](spravochnik-hukov.md#graph-metadata-search) и [CodeMetadataSearchServer](spravochnik-hukov.md#codemetadatasearchserver) |
| Нужен разбор связей объектов вместо поиска по коду | [Graph Metadata Search](../servery/graph-metadata-search/) работает по той же выгрузке и плагины поддерживает |

{% hint style="info" %}
Состав образов меняется — CodeMetadataSearchServer перешёл на общий контракт именно так. Прежде чем закладываться на отсутствие плагинов, проверьте текущий образ командой из первого раздела: появление `plugin_api.py` означает, что сервер перешёл на общий контракт, и вся [система плагинов](README.md) применима к нему без оговорок.
{% endhint %}
