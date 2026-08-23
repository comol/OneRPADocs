# Конфигурация HelpSearchServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ. Без него сервер не стартует | `YOUR_LICENSE_KEY` |

### Embedding

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `EMBEDDING_API_BASE` | URL OpenAI-совместимого API, включая `/v1`. Пробуется раньше локальной модели | `http://host.docker.internal:1234/v1` |
| `EMBEDDING_API_KEY` | Ключ API (LM Studio значение не проверяет) | `lm-studio` |
| `EMBEDDING_MODEL` | Имя модели — и для API, и для локального режима | `intfloat/multilingual-e5-small` |
| `HF_HOME` | Каталог кэша модели; в полный образ модель уложена сюда при сборке | `/app/model_cache` |
| `HF_HUB_OFFLINE` | Запрет загрузок при старте; `0` разрешает докачать модель | `1` |
| `RESET_CACHE` | Очистить кэш моделей при старте (удаляется только содержимое `HF_HOME`, сам том сохраняется) | `false` |

{% hint style="danger" %}
`RESET_CACHE=true` вместе с `HF_HUB_OFFLINE=1` (значение по умолчанию) удаляет уложенную в образ модель без возможности скачать её обратно. Если кэш нужно чистить, модель должна приходить извне — смонтированным каталогом в `HF_HOME`.
{% endhint %}

{% hint style="info" %}
`OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` поддерживаются как устаревшие алиасы. В новых конфигурациях используйте `EMBEDDING_*`.
{% endhint %}

### Пути и данные

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `1C_BIN_PATH` | Смонтированный каталог с `shcntx_ru.hbk` (папка `bin` платформы, сама папка с архивом или путь к файлу). Не задан — берётся архив, поставляемый с образом | *(не задано)* |
| `PLUGIN_DIR` | Каталог плагинов; смонтируйте свой, чтобы не пересобирать образ | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | `true` — упавший hook `on_document` роняет сборку индекса; `false` — документ берётся как прочитан, а ошибка считается | `false` |

### Индексация

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Разрешить разрушающие операции и очистить индекс | `false` |
| `INDEX_RETAIN_GENERATIONS` | Сколько прошлых поколений индекса хранить для отката | `1` |
| `INDEXING_WORKERS` | Количество потоков индексации | `5` |
| `INDEXING_BATCH_SIZE` | Строк в одной записи при индексации; именно это ограничивает память сборки — уменьшите на машине, которой её не хватает | `100` |

{% hint style="info" %}
Пересборка индекса не разрушает работающий: новое поколение собирается в staging и начинает обслуживать только после проверки. Упавшая сборка уходит в `index/quarantine` и никогда не обслуживает запросы, а отвечает по-прежнему предыдущее поколение.
{% endhint %}

### Транспорт и HTTP-сессии

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `USESSE` | Использовать SSE-транспорт (для legacy-клиентов) | `false` |
| `MCP_SESSION_IDLE_TTL_SECONDS` | Время простоя сессии Streamable HTTP до уборки | `1800` |
| `MCP_SESSION_MAX_LIFETIME_SECONDS` | Максимальное время жизни сессии | `86400` |
| `MCP_SESSION_CAP` | Максимум одновременных stateful-сессий | `1000` |
| `MCP_SESSION_CLEANUP_INTERVAL_SECONDS` | Интервал сверки времени жизни сессий и метрик | `60` |

`/mcp` и `/mcp/` обслуживаются одним и тем же эндпоинтом напрямую — клиента никогда не просят повторить MCP-запрос через redirect. Счётчики активных сессий и уборки видны в ответе `/health`.

### Тонкая настройка поиска

Значения измерены на этом корпусе и на поставляемой модели; менять их нужно только осознанно.

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RELEVANCE_MAX_VECTOR_DISTANCE` | Максимальная cosine-distance для векторной дорожки; больше — хуже совпадение. `2.0` пропускает всё (выключатель порога) | `0.13` |
| `RELEVANCE_MIN_LEXICAL_SCORE` | Минимальная лексическая релевантность полнотекстового попадания | `0.0` |
| `CHUNK_MAX_TOKENS` | Бюджет токенов одного чанка. Задавайте только для API-модели, не сообщающей свой лимит. **Изменение вызывает переиндексацию** | окно модели |
| `CHUNK_OVERLAP_TOKENS` | Сколько предыдущего чанка повторяет следующий. **Изменение вызывает переиндексацию** | `48` |
| `CHUNK_MAX_PER_PARENT` | Максимум чанков одного документа в одном ответе | `3` |
| `CHUNK_DUPLICATE_THRESHOLD` | Порог схожести, выше которого два чанка одного документа считаются одним ответом | `0.85` |
| `FILTERED_FETCH_FACTOR` | Во сколько раз шире опрашиваются дорожки при заданном `doc_type` | `4` |
| `DOCINFO_FALLBACK_CANDIDATES` | Предел числа кандидатов, когда имя не совпало точно | `5` |
| `DOCINFO_AMBIGUITY_CANDIDATES` | Предел числа кандидатов в одном ответе для неоднозначного имени | `20` |
| `LEXICAL_PROFILE` | Токенизация лексической дорожки: `stemmed`, `plain`, `folded`, `verbatim`. **Изменение вызывает переиндексацию** | `stemmed` |
| `RESPONSE_MAX_TOKENS` | Дополнительно ограничивать ответ в токенах, если модель отдаёт токенизатор. `max_chars` действует всегда | *(не задано)* |
| `RESPONSE_CURSOR_SECRET` | Ключ подписи курсоров постраничного вывода | *(не задано)* |

### Журналирование

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LOG_DIR` | Каталог файлов журнала | `logs` |
| `LOG_FILE` | Имя текущего файла журнала | `app.log` |
| `LOG_MAX_BYTES` | Размер, при котором журнал ротируется | `10485760` (10 МиБ) |
| `LOG_BACKUP_COUNT` | Сколько ротированных файлов хранится | `5` |
| `LOG_LEVEL` | Уровень записи в файл. `DEBUG` добавляет записи по документам и **текст запросов** | `INFO` |
| `LOG_CONSOLE_LEVEL` | Уровень вывода в stdout, если он должен отличаться | значение `LOG_LEVEL` |
| `LOG_PROGRESS_EVERY` | Через сколько пакетов писать прогресс индексации | `10` |

{% hint style="info" %}
На уровне по умолчанию журнал не содержит текста поисковых запросов: запрос записывается длиной и коротким хешем, а все записи одного вызова несут общий correlation id. Именно его называет клиенту ответ с ошибкой.
{% endhint %}

## Монтируемые тома

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `E:/bases/mcp_docs` | `/app/index` | Индекс (поколения zvec) — **обязательно** |
| `E:/bases/mcp_model_cache` | `/app/model_cache` | Кэш embedding-моделей |
| `C:/Program Files/1cv8/X.X.XX.XXXX/bin` | `/1c_docs` | Папка bin платформы, если нужна справка своей версии (только чтение) |
| `E:/plugins/mcp_docs` | `/app/plugins` | Свои плагины |

{% hint style="warning" %}
Раньше индекс лежал в `/app/chroma_db`. Сервер больше не использует ChromaDB; актуальный путь — `/app/index`. Перенос старого индекса — разовой командой миграции, см. [Установка](ustanovka.md).
{% endhint %}

## Примеры конфигураций

### Минимальная (встроенная CPU-модель)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:latest
```

### Рекомендуемая (LM Studio + своя версия платформы)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

### Light-образ (без PyTorch, embedding только через API)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:light
```

### CPU с выбором модели

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "E:/bases/mcp_docs:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

{% hint style="warning" %}
Смена embedding-модели делает сохранённый индекс несовместимым: при следующем старте он пересобирается в staging-поколение, а предыдущее продолжает отвечать до конца сборки.
{% endhint %}

## Плагины

Сервер поставляется скомпилированным, но расширяется плагинами. Плагин — **один Python-файл** в `/app/plugins` с функциями известных имён: ни базового класса, ни декоратора, ни регистрации, ни манифеста. Пустой файл — валидный плагин. Функции нет — hook не срабатывает. Упавший hook перехватывается, пишется в журнал с именем файла и трассировкой и отключается до следующей перезагрузки, но сервер не роняет.

Объявлены пять hooks — `on_startup`, `on_request`, `on_query`, `on_result` (в рамках вызова) и `on_document` (формирует индекс) — и две декларативные таблицы: `ALIASES` (дополнительные имена, которые разрешает `docinfo`) и `TOOL_PRESETS` (дополнительные MCP-инструменты как пресеты `docsearch` и `docinfo`).

| Что | Где |
|-----|-----|
| Полный справочник хуков и таблиц | `/app/plugin_api.py` в образе (читаемый исходник) |
| Короткая версия | `/app/plugins/AGENTS.md` |
| Пример со всеми хуками (закомментирован) | `/app/plugins/example.py` |
| Проверить плагин без данных и индекса | `docker run --rm -v "$PWD/my_plugin.py:/tmp/my_plugin.py" comol/1c_help_mcp:latest python3 plugin_host.py --dry-run /tmp/my_plugin.py` |
| Что загружено и что отключено | `GET /plugins` |
| Перечитать каталог без перезапуска | `POST /plugins/reload` |

Правка call-scoped хука действует со следующего вызова после перезагрузки и ничего не пересобирает. `on_document` формирует то, что пишется в индекс, поэтому исходник объявившего его файла входит в отпечаток годности индекса: правка делает сохранённый индекс устаревшим, и следующий старт пересобирает его в staging, пока обслуживающее поколение продолжает отвечать.

## Конфигурация Cursor

### mcp.json

```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    }
  }
}
```

### Путь к файлу

```
%APPDATA%\Cursor\User\globalStorage\mcp.json
```

## GPU ускорение

### Windows 11 с NVIDIA

```powershell
docker run -d -p 8003:8003 `
  --gpus all `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "E:/bases/mcp_docs:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

{% hint style="info" %}
GPU ускорение работает только с Windows 11 и актуальными драйверами NVIDIA. Альтернатива, которая обычно проще: считать эмбеддинги в LM Studio и указать серверу `EMBEDDING_API_BASE`.
{% endhint %}
