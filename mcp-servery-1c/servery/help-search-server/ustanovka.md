# Установка

{% hint style="warning" %}
Эта страница описывает новые beta-сборки HelpSearchServer: теги `latest-beta`, `light-beta`, `arm64-beta` и индекс поколений в `/app/index`. Stable-теги используют прежнюю ChromaDB в `/app/chroma_db`; команды для каналов нельзя смешивать. См. [Каналы образов](../../kanaly-obrazov.md).
{% endhint %}

## Предварительные требования

1. Docker Engine или Docker Desktop запущен
2. LM Studio запущен с моделью Qwen3-Embedding (рекомендуется; без него используется встроенная CPU-модель)
3. Платформа 1С:Предприятие — **опционально**, если нужна справка именно вашей версии

## Создание папки для индекса

```powershell
New-Item -ItemType Directory -Force -Path "E:\bases\mcp_docs_beta"
```

{% hint style="info" %}
Путь `E:\bases\mcp_docs_beta` — это **пример**. Используйте любой удобный путь на вашем компьютере.
{% endhint %}

{% hint style="danger" %}
**КРИТИЧЕСКИ ВАЖНО!** Обязательно монтируйте папку для индекса (`-v "...:/app/index"`). Без этого при каждом пересоздании контейнера индексация будет начинаться заново, что может занять от нескольких часов до суток!
{% endhint %}

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_BETA_LICENSE_KEY `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_docs_beta:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest-beta
```

{% hint style="success" %}
Если LM Studio запущен локально на порту 1234 (по умолчанию), параметры `EMBEDDING_API_BASE` и `EMBEDDING_API_KEY` можно не указывать — это значения по умолчанию, сервер подключится сам.
{% endhint %}

### С CPU (без GPU)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_BETA_LICENSE_KEY `
  -v "E:/bases/mcp_docs_beta:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest-beta
```

## Справка своей версии платформы

Архив синтакс-помощника `shcntx_ru.hbk` поставляется вместе с образом, поэтому сервер работает и без установленной платформы. Чтобы индексировать справку **вашей** версии, смонтируйте папку `bin` и укажите на неё `1C_BIN_PATH`.

### Поиск папки bin

Типичные пути:

* `C:\Program Files\1cv8\8.3.23.1997\bin`
* `C:\Program Files (x86)\1cv8\8.3.23.1997\bin`

```powershell
Test-Path "C:\Program Files\1cv8\8.3.23.1997\bin"
```

### Запуск со своей справкой

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_BETA_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs_beta:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest-beta
```

{% hint style="warning" %}
Замените `8.3.23.1997` на вашу версию платформы! `1C_BIN_PATH` принимает и папку `bin`, и папку с самим `shcntx_ru.hbk`, и путь к файлу архива.
{% endhint %}

## Первый запуск

### Что происходит

1. **Скачивание образа** (\~3,5 ГБ для `latest-beta`, \~120 МБ для `light-beta`)
2. **HTTP-сервер поднимается сразу** — `/health`, `/ready` и `tools/list` отвечают с первой секунды, холодный старт никогда не выглядит как закрытый порт
3. **Загрузка embedding-модели** (в полном образе она уже внутри)
4. **Индексация корпусов** — справка платформы, руководства, спецификации форматов, стандарты (см. таблицу ниже)
5. Новое **поколение индекса** проверяется и начинает обслуживать запросы без перезапуска процесса

Пока индекс строится впервые, `docsearch` и `docinfo` отвечают статусом `indexing` с прогрессом — это не ошибка. Ничего не разрушается: упавшая сборка уходит в `index/quarantine`, а обслуживать продолжает предыдущее поколение.

### Ожидаемое время индексации

| Режим | Время индексации |
|-------|------------------|
| CPU (e5-small) | 10-20 часов |
| CPU (e5-base) | 15-30 часов |
| GPU (Qwen-4B через LM Studio) | 2-4 часа |

{% hint style="warning" %}
Планируйте первую индексацию заранее — запустите контейнер на ночь или на выходных. После завершения индекс сохранится в томе и повторная индексация не потребуется.
{% endhint %}

### Мониторинг прогресса

```powershell
# Просмотр логов в реальном времени
docker logs -f 1c_help_mcp

# Готовность и состав обслуживающего поколения
Invoke-RestMethod -Uri "http://localhost:8003/ready"
```

`/ready` возвращает `starting`, `indexing`, `ready` или `degraded` и отдаёт `200` только тогда, когда поколение индекса обслуживает запросы и последняя сборка не упала. `HEALTHCHECK` образа обращается именно к нему.

## Миграция со старого индекса (chroma_db)

Beta-сервер больше не использует ChromaDB: индекс лежит в `/app/index` поколениями. Stable продолжает использовать `/app/chroma_db`. Если у вас том со старым индексом в `chroma_db`, перенесите его разовой командой — переиндексация не нужна, векторы переносятся как есть:

```powershell
docker run --rm `
  -v "E:/bases/mcp_docs_old:/app/chroma_db" `
  -v "E:/bases/mcp_docs_beta:/app/index" `
  comol/1c_help_mcp:latest-beta `
  sh -c "pip install chromadb && python3 legacy_migration.py --legacy chroma_db --index index"
```

Старый том читается через копию и остаётся байт-в-байт прежним; удалить его (флагом `--remove-legacy`) можно только после того, как построенное из него поколение стало обслуживающим.

{% hint style="info" %}
Если старого индекса нет — этот шаг не нужен, сервер соберёт индекс с нуля.
{% endhint %}

## Последующие запуски

```powershell
# Остановка
docker stop 1c_help_mcp

# Запуск (индекс сохранён в томе)
docker start 1c_help_mcp
```

`RESET_DATABASE` по умолчанию `false` — индекс при старте не разрушается. Ставьте `true`, только когда индекс нужно построить заново с нуля.

## Обновление справки

При обновлении версии платформы 1С достаточно перемонтировать новую папку `bin`: сервер увидит, что содержимое корпуса изменилось, и пересоберёт индекс в staging-поколении, продолжая отвечать из текущего.

```powershell
docker stop 1c_help_mcp
docker rm 1c_help_mcp

docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_BETA_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -v "C:/Program Files/1cv8/8.3.24.1234/bin:/1c_docs" `
  -v "E:/bases/mcp_docs_beta:/app/index" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest-beta
```

## Установка в закрытом контуре

Для контура без доступа в интернет существует **offline bundle** — автономная поставка, где колёса Python, embedding-модель, корпус и приложение лежат в одном каталоге с манифестом контрольных сумм и SBOM. Установка и запуск не делают ни одного сетевого обращения, а отсутствующий артефакт называется по имени, а не докачивается.

Bundle разворачивается без Docker:

```bash
python3 offline_bundle.py verify
python3 offline_bundle.py install --prefix /opt/helpsearch
```

Требования к машине (та же ОС, архитектура и версия CPython, что записаны в `manifest.json`) проверяются установщиком: несовпадение отклоняется, а не обходится. Bundle предоставляется по запросу.

## Проверка работы

```powershell
# Статус контейнера
docker ps --filter name=1c_help_mcp

# Живость процесса
Invoke-RestMethod -Uri "http://localhost:8003/health"

# Готовность отвечать на поиск
Invoke-RestMethod -Uri "http://localhost:8003/ready"
```

### Настройка Cursor

1. Откройте `%APPDATA%\Cursor\User\globalStorage\mcp.json`
2. Добавьте конфигурацию сервера
3. Перезапустите Cursor
4. Задайте вопрос о методах 1С

## Устранение проблем

### Контейнер не запускается

```powershell
docker logs 1c_help_mcp
```

Самая частая причина — не задан `LICENSE_KEY`: без него сервер завершает работу.

### `/ready` отвечает `degraded`

Последняя сборка индекса упала. Смотрите журнал: запись о падении содержит шаг, на котором сборка остановилась, и полную трассировку. Обслуживание при этом продолжает предыдущее поколение, если оно есть.

### Ошибка подключения к LM Studio

1. Убедитесь, что LM Studio Server запущен
2. Проверьте порт 1234
3. Используйте `host.docker.internal` вместо `localhost`

### Модель скачивается заново при каждом старте

Проверьте, что не задан `RESET_CACHE=true`, и что смонтирован том в `/app/model_cache`.
