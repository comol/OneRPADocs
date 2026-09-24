# Выбор модели

Детальное сравнение вариантов embedding моделей для MCP-серверов.

## Сводная таблица

| Модель                             | Размерность | Качество | Скорость | Требования | Офлайн |
| ---------------------------------- | ----------- | -------- | -------- | ---------- | ------ |
| **Qwen3-Embedding-4B** (LM Studio) | 2560        | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐⭐    | GPU 4 ГБ   | ✅      |
| **Qwen3-Embedding-8B** (LM Studio) | \~4096      | ⭐⭐⭐⭐⭐    | ⭐⭐⭐⭐     | GPU 8 ГБ   | ✅      |
| **multilingual-e5-large** (CPU)    | 1024        | ⭐⭐⭐⭐⭐    | ⭐⭐       | 4 ГБ RAM   | ✅      |
| **multilingual-e5-base** (CPU)     | 768         | ⭐⭐⭐⭐     | ⭐⭐⭐      | 2 ГБ RAM   | ✅      |
| **multilingual-e5-small** (CPU)    | 384         | ⭐⭐⭐      | ⭐⭐⭐⭐⭐    | 1 ГБ RAM   | ✅      |

{% hint style="info" %}
Для GPU-ускорения рекомендуется использовать **LM Studio**. Это самый простой способ получить высокую производительность без сложной настройки CUDA в Docker-контейнерах.
{% endhint %}

## Рекомендации по сценариям

### Сценарий 1: Есть NVIDIA GPU (4+ ГБ)

**Рекомендация: LM Studio + Qwen3-Embedding-4B**

```env
EMBEDDING_API_BASE=http://host.docker.internal:1234/v1
EMBEDDING_API_KEY=lm-studio
EMBEDDING_MODEL=Qwen3-Embedding-4B
```

Плюсы:

* Лучшее качество поиска
* Быстрая индексация (минуты)
* Отличная поддержка русского языка

### Сценарий 2: Есть NVIDIA GPU (8+ ГБ)

**Рекомендация: LM Studio + Qwen3-Embedding-8B**

```env
EMBEDDING_API_BASE=http://host.docker.internal:1234/v1
EMBEDDING_API_KEY=lm-studio
EMBEDDING_MODEL=Qwen3-Embedding-8B
```

Плюсы:

* Максимальное качество
* Ещё лучше понимает контекст

### Сценарий 3: Нет GPU, но есть время

**Рекомендация: CPU + multilingual-e5-base**

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-base
```

Плюсы:

* Не требует GPU
* Хороший баланс качество/скорость
* Полностью офлайн

### Сценарий 4: Нет GPU, нужна скорость

**Рекомендация: CPU + multilingual-e5-small**

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-small
```

Плюсы:

* Самая быстрая на CPU
* Минимальные требования
* Приемлемое качество

## Качество поиска

### Что влияет на качество

1. **Размерность вектора** — больше = точнее, но медленнее
2. **Обучение модели** — Qwen обучен на русском языке
3. **Квантизация** — Q8 лучше Q4

### Инструкция к запросу у Qwen3

Qwen3-Embedding рассчитан на то, что перед поисковым запросом стоит инструкция задачи: `Instruct: <задача>`, перевод строки, `Query:` и сам запрос. Серверы добавляют её сами, если имя модели содержит `qwen` и `embed` (`qwen/qwen3-embedding-8b`, `Qwen3-Embedding-4B`, `text-embedding-qwen3-embedding-0.6b` в LM Studio, `qwen3-embedding:8b` в Ollama). Инструкция добавляется только к запросу: документы индексируются как раньше, поэтому переиндексация не нужна. Отдельной переменной для неё нет.

| Сервер | Задача в инструкции |
|--------|---------------------|
| HelpSearchServer | `Given a 1C developer question, retrieve the matching 1C:Enterprise documentation page (built-in language or query language reference, platform object, file format specification, or development standard) that answers it.` |
| SSLSearchServer | `Given a 1C developer question, retrieve the matching 1C Standard Subsystems Library (SSL/BSP) API procedure or function that answers it.` |
| Graph Metadata Search | `Given a 1C developer question, retrieve the matching 1C configuration metadata object or code fragment (BSL, query language, or module) that answers it.` |
| TemplatesSearchServer, `templatesearch` | `Given a 1C developer question, retrieve the matching 1C code template (BSL or query language) that answers it.` |
| TemplatesSearchServer, `recall` | `Given a 1C developer question, retrieve the matching project memory note (solution, observation, or fact) that answers it.` |
| CodeMetadataSearchServer | `Given a web search query, retrieve relevant passages that answer the query` — текст из карточки модели |

Замеры на `qwen/qwen3-embedding-8b` (23.09.2026):

* **HelpSearchServer**, 219 страниц синтакс-помощника: нужная страница стала ближе к запросу. По короткому имени медианное расстояние 0,310 → 0,292, по фразе описания 0,247 → 0,214. Доля нужных страниц в пределах `RELEVANCE_MAX_VECTOR_DISTANCE` (0,35) выросла с 0,61 до 0,77 и с 0,79 до 0,85. Ближайший запрос не по теме остался на расстоянии 0,454, поэтому порог не менялся.
* **SSLSearchServer**, вся база 311 (1613 записей): на 22 вопросах разработчика MRR почти не изменился (0,716 без инструкции, 0,707 с ней). Зато порог `MIN_SCORE` стал отсекать часть запросов не по теме: без инструкции его проходили 10 из 10, с инструкцией — 7.
* **CodeMetadataSearchServer**: формулировки под 1С проверялись на корпусе бенчмарка и оказались хуже текста карточки (nDCG@10 0,836–0,844 против 0,866), поэтому там остался текст карточки. Подробно — в [конфигурации сервера](../servery/code-metadata-search/konfiguraciya.md).
* **Graph Metadata Search** и **TemplatesSearchServer** не измерялись.

Кроме этой инструкции серверы ничего к тексту не добавляют. В beta-сборках с 23.09.2026 (поздний вечер) убраны префиксы вида `query: ` / `passage: `, встроенные prompts моделей и переменные `EMBEDDING_QUERY_PREFIX`, `EMBEDDING_DOCUMENT_PREFIX` и `EMBEDDING_PASSAGE_PREFIX`: заданная переменная теперь только выводит предупреждение в журнал. Индекс, документы которого были записаны с префиксом, сервер один раз пересобирает сам:

* CodeMetadataSearchServer — модели e5, EmbeddingGemma, `qwen/qwen3-embedding-8b` (префикс `document: `) или заданный `EMBEDDING_DOCUMENT_PREFIX`; пересборка идёт в новом поколении, прежнее отвечает до её конца;
* SSLSearchServer — локальная модель e5 (модель полного образа по умолчанию), nomic или модель со встроенным prompt документа;
* TemplatesSearchServer — если был задан `EMBEDDING_QUERY_PREFIX` или `EMBEDDING_PASSAGE_PREFIX`.

Остальные индексы не меняются и не пересчитываются. У HelpSearchServer префиксы и раньше не применялись. У SSLSearchServer `EMBEDDING_INPUT_TYPE_ENABLED=false` выключает инструкцию вместе с параметром `input_type`.

## Смена модели

{% hint style="warning" %}
При смене embedding модели требуется полная переиндексация всех данных.
{% endhint %}

### Процесс смены модели

1. Остановите контейнер
2. Измените переменные окружения
3. Установите `RESET_DATABASE=true`
4. Запустите контейнер
5. Дождитесь переиндексации
6. Измените `RESET_DATABASE=false` для следующих запусков

{% hint style="info" %}
CodeMetadataSearchServer в beta-сборках с 23.09.2026 пересобирает индекс сам: смените модель и пересоздайте контейнер, `RESET_DATABASE` не нужен. Подробнее — в [конфигурации сервера](../servery/code-metadata-search/konfiguraciya.md).
{% endhint %}

### Пример

```powershell
# Остановить контейнер
docker stop 1c_help_mcp

# Удалить старый контейнер
docker rm 1c_help_mcp

# Запустить с новой моделью и переиндексацией
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=true `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-8B `
  -e 1C_BIN_PATH=/1c_docs `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:latest
```

## Автоматическое определение размерности

MCP-серверы автоматически:

1. Тестируют embedding модель при запуске
2. Определяют размерность вектора
3. Сравнивают с существующим индексом
4. Пересоздают индекс если размерность изменилась

### Логи при смене модели

```
INFO - Testing embedding API with model: Qwen3-Embedding-8B
INFO - ✓ Embedding dimension: 4096
INFO - Found existing vector index with dimension: 2560
WARNING - ⚠️ DIMENSION MISMATCH DETECTED!
INFO - 🔄 Rebuilding vector index with correct dimensions...
INFO - ✓ Vector index created successfully!
```

## Итоговые рекомендации

| Ваша ситуация                          | Рекомендуемая модель           |
| -------------------------------------- | ------------------------------ |
| GPU 4+ ГБ, хотите лучшее качество      | LM Studio + Qwen3-Embedding-4B |
| GPU 8+ ГБ, нужно максимальное качество | LM Studio + Qwen3-Embedding-8B |
| Нет GPU, важно качество                | CPU + multilingual-e5-base     |
| Нет GPU, важна скорость                | CPU + multilingual-e5-small    |

{% hint style="warning" %}
**Для пользователей из России:** CPU-модели скачиваются с huggingface.co, который может быть заблокирован. Используйте VPN для первоначального скачивания или выберите LM Studio, где модели скачиваются через встроенный интерфейс приложения.
{% endhint %}
