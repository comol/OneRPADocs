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
docker run -d -p 8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_templates:/app/chroma_db" `
  comol/template-search-mcp:latest
```

### С CPU (без GPU)

```powershell
docker run -d -p 8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=ai-forever/FRIDA `
  -v "E:/bases/mcp_templates:/app/chroma_db" `
  comol/template-search-mcp:latest
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
| `PLUGIN_DIR` | Каталог Python-плагинов внутри контейнера | `/app/plugins` |
| `PLUGIN_STRICT_DERIVED_STATE` | Останавливать индексацию при ошибке derived-state hook | `false` |

Старые имена `OPENAI_API_BASE`, `OPENAI_API_KEY` и `OPENAI_MODEL` остаются совместимыми алиасами.

Свой каталог плагинов можно подключить томом в `/app/plugins`. Текущее состояние показывает `plugin_state`, а `plugin_reload` перечитывает каталог без перезапуска контейнера.

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
| `http://localhost:8004/ready` | Индексы построены и сервер готов отвечать |

```powershell
Invoke-RestMethod http://localhost:8004/ready
```

### Веб-интерфейс

Откройте в браузере: `http://localhost:8004/extend/`

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-templates-mcp": {
      "url": "http://localhost:8004/mcp",
      "connection_id": "1c_templates_service_001"
    }
  }
}
```
