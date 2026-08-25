# Установка TemplatesSearchServer

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен (рекомендуется)

## Создание папки для данных

```powershell
New-Item -ItemType Directory -Force -Path "E:\bases\mcp_templates"
```

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 127.0.0.1:8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_templates:/app/chroma_db" `
  comol/template-search-mcp:latest-beta
```

### С CPU (без GPU)

```powershell
docker run -d -p 127.0.0.1:8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=ai-forever/FRIDA `
  -v "E:/bases/mcp_templates:/app/chroma_db" `
  comol/template-search-mcp:latest-beta
```

## Переменные окружения

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `RESET_DATABASE` | Построить новое поколение индекса шаблонов. Также срабатывает автоматически при несовпадении размерности эмбеддингов. Сохранённый индекс продолжает отвечать, пока замена не проверена и не переведена в обслуживание; `templates.db` не затрагивается | `false` |
| `RESET_CACHE` | Удалить скачанные веса embedding-модели и загрузить их заново | `false` |
| `USESSE` | SSE транспорт (для legacy клиентов) | `false` |
| `HTTP_PORT` | Порт HTTP-сервера | `8004` |
| `EMBEDDING_MODEL` | Модель для внешнего API или локальная CPU-модель с Hugging Face | `intfloat/multilingual-e5-small` |
| `EMBEDDING_API_BASE` | URL API сервера (LM Studio, Ollama, OpenRouter). Суффикс `/v1` добавляется автоматически | — |
| `EMBEDDING_API_KEY` | Ключ API | `lm-studio` |
| `EMBEDDING_DIMENSIONS` | Явное указание размерности эмбеддингов. Для моделей с переменной размерностью (Qwen3, text-embedding-3). Если не указано — определяется автоматически | *(авто)* |
| `TEMPLATES_DB_PATH` | Путь к SQLite-базе шаблонов и заметок | `/app/chroma_db/templates.db` |
| `ZVEC_DB_PATH` | Каталог коллекций zvec | `/app/chroma_db/zvec_db` |
| `RECALL_RELEVANCE_THRESHOLD` | Максимальная cosine-distance для результата `recall` | `1.0` |
| `TEMPLATE_RELEVANCE_THRESHOLD` | Максимальная cosine-distance для результата `templatesearch`; задаётся отдельно от `recall`, потому что описания кода и свободные заметки — разный текст | `1.0` |
| `FUSION_RANK_CONSTANT` | Константа reciprocal rank fusion при объединении семантического и полнотекстового маршрутов | `60` |
| `FTS_DEFAULT_OPERATOR` | Как объединяются соседние термины в полнотекстовом маршруте: `OR` или `AND` | `OR` |
| `GROUP_RESULT_CAP` | Максимум документов одного шаблона в ответе | `1` |
| `GROUP_CANDIDATE_BUDGET` | Сколько различных шаблонов запрашивается в выборке кандидатов | `11` |
| `INDEX_GENERATION_RETENTION` | Сколько поколений каждой коллекции хранится на диске; значение меньше 2 повышается до 2, потому что откатиться на удалённый индекс нельзя | `2` |
| `EMBEDDING_QUERY_PREFIX` | Префикс, добавляемый к тексту запроса перед эмбеддингом | *(пусто)* |
| `EMBEDDING_PASSAGE_PREFIX` | Префикс, добавляемый к индексируемому тексту перед эмбеддингом | *(пусто)* |
| `EMBEDDING_TRUST_REMOTE_CODE` | Разрешить выполнение пользовательского кода модели; требует allowlist и неизменяемую ревизию | `false` |
| `EMBEDDING_TRUST_REMOTE_CODE_MODELS` | Разрешённые model ID для remote code, через запятую | *(пусто)* |
| `EMBEDDING_MODEL_REVISION` | Полный commit SHA или `sha256:` digest модели | *(пусто)* |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | Учётная запись управления web UI | *(не заданы; изменения запрещены)* |
| `ADMIN_USERS` | Несколько операторов: `имя:пароль:разрешения;...` | *(пусто)* |
| `ADMIN_PERMISSIONS` | Разрешения единственного оператора: `create,edit,delete` | все три |
| `ADMIN_SESSION_SECRET` | Подпись web-сессий; без него сессии сбрасываются при рестарте | генерируется при старте |
| `ADMIN_ALLOW_UNAUTHENTICATED` | Небезопасный opt-in для изменяющих web-запросов без входа | `false` |
| `MAX_DESCRIPTION_BYTES` | Максимальный размер описания шаблона | `20000` |
| `MAX_CODE_BYTES` | Максимальный размер кода шаблона | `200000` |
| `MAX_MEMORY_BYTES` | Максимальный размер заметки | `20000` |
| `MAX_REQUEST_BODY_BYTES` | Максимальный размер тела web-запроса | `1000000` |
| `MCP_ENABLE_WRITE_TOOLS` | Регистрировать `add_template` и `plugin_reload`. На `remember` не влияет | `false` |
| `MCP_OPERATOR_TOKEN` | Bearer-токен для `add_template` / `plugin_reload`; нужен вместе с `MCP_ENABLE_WRITE_TOOLS=true` | *(пусто)* |
| `MCP_MAX_CONCURRENT_MUTATIONS` | Одновременные MCP-мутации | `2` |
| `MCP_MUTATION_RATE_PER_MINUTE` | Скорость пополнения лимита мутаций | `30` |
| `MCP_MUTATION_BURST` | Максимальный всплеск мутаций | `10` |
| `MCP_MUTATION_DAILY_QUOTA` | Суточная квота MCP-мутаций (UTC) | `500` |
| `MCP_MAX_MUTATIONS_PER_REQUEST` | Максимум изменяющих вызовов в одном JSON-RPC запросе | `8` |
| `MCP_SESSION_IDLE_TIMEOUT` | Простой Streamable HTTP-сессии до освобождения, секунды | `1800` |
| `MCP_SESSION_MAX_LIFETIME` | Максимальное время жизни сессии, секунды | `28800` |
| `MCP_MAX_SESSIONS` | Максимум одновременных сессий | `200` |
| `MCP_SESSION_CLEANUP_INTERVAL` | Интервал уборки сессий, секунды | `60` |
| `MCP_SESSION_CLEANUP_BATCH` | Сессий за один проход уборки | `50` |
| `INDEX_RECOVERY_INTERVAL` | Период фоновой проверки outbox, секунды | `60` |
| `INDEX_RECOVERY_BACKOFF` | Начальная пауза повторной индексации, секунды | `5` |
| `INDEX_RECOVERY_BACKOFF_CAP` | Максимальная пауза повторной индексации, секунды | `300` |
| `INDEX_RECOVERY_MAX_ATTEMPTS` | Число попыток до состояния stuck | `6` |
| `INDEX_RECOVERY_BATCH` | Маркеров outbox за проход | `100` |
| `PLUGIN_DIR` | Каталог Python-плагинов внутри контейнера | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Останавливать индексацию при ошибке derived-state hook | `false` |

Старые имена `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` остаются совместимыми алиасами.

{% hint style="warning" %}
Web UI закрыт для изменений по умолчанию. Не включайте `ADMIN_ALLOW_UNAUTHENTICATED` на опубликованном порту. Для `add_template` и `plugin_reload` нужны одновременно `MCP_ENABLE_WRITE_TOOLS=true` и стойкий `MCP_OPERATOR_TOKEN`; клиент формирует из него bearer-заголовок `Authorization`, а значение не записывается в документацию или журнал. Проектная память (`remember`) под этот гейт не попадает и токена не требует — поэтому порт публикуйте на `127.0.0.1`: заметку запишет любой, кто до него дотянется.
{% endhint %}

Для моделей с пользовательским кодом все три параметра supply chain обязательны: opt-in, allowlist model ID и неизменяемая ревизия. Неполная комбинация отклоняется до загрузки модели.

## Плагины

Свой каталог плагинов можно подключить томом в `/app/plugins`. Текущее состояние показывает `plugin_state`, а `plugin_reload` перечитывает каталог без перезапуска контейнера.

Плагин — **один Python-файл** в каталоге плагинов. Объявлены шесть hooks — `on_startup`, `on_request`, `on_rank`, `on_result` (в рамках вызова) и `on_template`, `on_memory` (формируют векторные коллекции) — и таблица `QUERY_ALIASES` (замены терминов в нормализованном запросе `templatesearch` и `recall`).

```powershell
# Прогнать плагин на фикстурах образа: без данных, коллекций и модели
docker run --rm -v "E:/plugins/mcp_templates/10-aliasy.py:/tmp/my_plugin.py" `
  comol/template-search-mcp:latest-beta python main.py --dry-run /tmp/my_plugin.py
```

{% hint style="warning" %}
`plugin_reload` — изменяющий инструмент, как и `add_template`. По умолчанию оба не регистрируются: их включает `MCP_ENABLE_WRITE_TOOLS`, и вместе с ним обязателен стойкий `MCP_OPERATOR_TOKEN` (иначе сервер не стартует). Токен передаётся в заголовке `Authorization`. Без них каталог перечитывается перезапуском контейнера, а `plugin_state` доступен всегда. `remember` к ним не относится: проектная память пишется без токена при любой конфигурации.
{% endhint %}

Правка `on_template` или `on_memory` делает сохранённые коллекции устаревшими, и следующий старт пересобирает их — отпечаток хранится в `index_meta.json`. Записи веб-интерфейса `/extend` проходят через `on_template` и `on_memory`, но не через `on_request` и `on_result`: те срабатывают только на вызовах MCP-инструментов.

Полный контракт — в образе (`/app/plugin_api.py`, `/app/plugins/AGENTS.md`, `/app/plugins/example.py`). Общие правила и рецепты: [Доработка MCP: система плагинов](../../sistema-pluginov/) и [справочник хуков TemplatesSearchServer](../../sistema-pluginov/spravochnik-hukov.md#templatessearchserver).

## Первый запуск

При первом запуске:
1. Скачивается образ
2. Индексируются встроенные шаблоны
3. Запускается веб-интерфейс

### Мониторинг

```powershell
docker logs -f template_search_mcp
```

## Проверка работы

### MCP endpoint

```powershell
docker ps --filter name=template_search_mcp
```

### Liveness и readiness

| Endpoint | Назначение |
|----------|------------|
| `http://localhost:8004/health` | Процесс жив |
| `http://localhost:8004/ready` | Индексы построены и recovery завершён; до готовности возвращает 503 |

```powershell
Invoke-RestMethod http://localhost:8004/ready
```

### Веб-интерфейс

Откройте в браузере: `http://localhost:8004/extend/`. Для записи войдите через `/extend/login` с настроенной учётной записью.

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-templates-mcp": {
      "url": "http://localhost:8004/mcp",
      "connection_id": "1c_templates_service_001",
      "headers": {
        "Authorization": "Bearer <MCP_OPERATOR_TOKEN>"
      }
    }
  }
}
```

Блок `headers` нужен только после включения `add_template` / `plugin_reload`; в остальных случаях удалите его — поиск и проектная память (`remember` / `recall`) работают без него.
