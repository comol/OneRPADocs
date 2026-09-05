# Установка CodeMetadataSearchServer

## Предварительные требования

1. Docker Engine или Docker Desktop запущен
2. LM Studio запущен (рекомендуется)
3. [Подготовлены данные](podgotovka-dannyh.md) из Конфигуратора

## Создание папок

```powershell
New-Item -ItemType Directory -Force -Path @(
    "E:\bases\mcp_codemetadata",
    "E:\1C_Export\Files"
)
```

## Выбор образа

{% hint style="warning" %}
Режимы `METADATA_SOURCE=xml`, `SOURCE_FORMAT=designer_xml|edt` и автоматическое получение метаданных прямо из выгрузки опубликованы в beta. Поэтому команды ниже используют `latest-beta`, `light-beta` и `arm64-beta`. Stable-теги `latest`, `light`, `arm64` сохраняют прежний контракт с обязательным текстовым отчётом конфигуратора.
{% endhint %}

| Тег | Размер | Описание |
|-----|--------|----------|
| `latest-beta` | ~2.9 GB | Полная beta: локальные embedding (CPU/GPU) + API |
| `light-beta` | ~290 MB | Облегчённая beta: embedding только через API (LM Studio, OpenRouter и т.д.) |
| `arm64-beta` | ~500 MB | Beta для Apple Silicon / ARM серверов |

{% hint style="info" %}
Если вы используете LM Studio или OpenRouter для embedding — выбирайте `light-beta`. Образ в 10 раз легче и запускается быстрее.
{% endhint %}

{% hint style="warning" %}
Явно закрепляйте `EMBEDDING_MODEL` и, если модель поддерживает переменную размерность, `EMBEDDING_DIMENSIONS`. Их смена полностью переэмбеддирует пять векторных дорожек (`metadata`, `metadata_xml`, `code`, `help`, `form_index`) и повторно расходует время и лимиты провайдера. Подробности и текущая рекомендация — на странице [конфигурации](konfiguraciya.md).
{% endhint %}

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
```

### С LM Studio (облегчённый образ)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:light-beta
```

### С CPU (без GPU)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
```

## Первый запуск

При первом запуске происходит индексация:

1. Определение формата Designer XML / 1C:EDT и чтение метаданных
2. Анализ файлов кода, форм, справки и зависимостей
3. Создание векторного индекса

{% hint style="warning" %}
**Важно!** Монтируйте постоянный том для данных (`-v "...:/app/chroma_db"`): он сохраняет индекс при пересоздании контейнера. Том сам по себе не гарантирует возобновление незавершённой дорожки и не защищает от повреждения хранилища. Не удаляйте его после прерывания; см. [перезапуск и сохранность прогресса](vremya-indeksacii.md#перезапуск-и-сохранность-прогресса).
{% endhint %}

### Мониторинг

```powershell
docker logs -f 1c_code_metadata_mcp
```

### Ожидаемое время индексации

Первый запуск крупной конфигурации может занимать много часов. Подтверждённых сопоставимых замеров полного первого запуска типовых УТ/ERP на CPU, локальной GPU и через API пока нет; прежнюю таблицу часов по числу объектов нельзя использовать как норму.

На странице [«Время первичной индексации»](vremya-indeksacii.md) разобраны три режима, оценка остатка по реальному прогрессу и протокол сопоставимого замера. `ETA` в логе относится к текущей дорожке, не ко всему серверу. Ночное окно планируйте по замеру на своей выгрузке, а не по размеру Docker-образа.

Готовое корректное поколение сохраняется в томе и при неизменных исходниках и настройках не требует полной перестройки. Прерывание незавершённой дорожки, смена модели и повреждение хранилища — другие сценарии; наличие тома само по себе их не устраняет.

## Проверка работы

```powershell
docker ps --filter name=1c_code_metadata_mcp
```

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-code-metadata-mcp": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_metadata_service_001"
    }
  }
}
```

## Обновление после изменений конфигурации

```powershell
# Удалить старый контейнер
docker rm -f 1c_code_metadata_mcp

# Запустить с инкрементальным обновлением
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e CODE_PATH="/app/code" `
  -e METADATA_SOURCE=xml `
  -e SOURCE_FORMAT=auto `
  -e RESET_DATABASE=false `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest-beta
```

При `INCREMENTAL_INDEXING=true` сервер сравнивает SHA-256 исходников и атомарно публикует новое поколение после проверки. `RESET_DATABASE=true` используйте только для осознанной полной перестройки.
