# Установка CodeMetadataSearchServer

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен (рекомендуется)
3. [Подготовлены данные](podgotovka-dannyh.md) из Конфигуратора

## Создание папок

```powershell
New-Item -ItemType Directory -Force -Path @(
    "E:\bases\mcp_codemetadata",
    "E:\1C_Export\Report",
    "E:\1C_Export\Files"
)
```

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```

### С CPU (без GPU)

```powershell
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```

## Первый запуск

При первом запуске происходит индексация:

1. Парсинг отчета по метаданным
2. Анализ файлов кода
3. Создание векторного индекса

{% hint style="warning" %}
**Важно!** Обязательно монтируйте том для сохранения индекса (`-v "...:/app/chroma_db"`). Без этого при перезапуске контейнера индексация начнётся заново!
{% endhint %}

### Мониторинг

```powershell
docker logs -f 1c_code_metadata_mcp
```

### Ожидаемое время индексации

| Размер конфигурации | CPU режим | GPU (LM Studio) |
|---------------------|-----------|-----------------|
| Небольшая (до 1000 объектов) | 2-6 часов | 30-60 мин |
| Средняя (1000-5000 объектов) | 10-20 часов | 2-4 часа |
| Большая (5000+ объектов) | 20-60 часов | 5-10 часов |

{% hint style="info" %}
Рекомендуем запускать первую индексацию на ночь или на выходных. После завершения индекс сохранится в томе и повторная индексация не потребуется.
{% endhint %}

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

# Запустить с переиндексацией
docker run -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=true `
  -v "E:/1C_Export/Report:/app/metadata" `
  -v "E:/1C_Export/Files:/app/code" `
  -v "E:/bases/mcp_codemetadata:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```
