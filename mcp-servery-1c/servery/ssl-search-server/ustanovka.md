# Установка SSLSearchServer

## Предварительные требования

1. Docker Engine или Docker Desktop запущен
2. LM Studio запущен (рекомендуется) или будет использоваться CPU

## Определение версии БСП

В Конфигураторе 1С:
1. Откройте конфигурацию
2. Справка → О программе
3. Найдите строку "Библиотека стандартных подсистем"
4. Запишите номер версии (например, `3.1.11`)

## Создание папки для индекса

```powershell
New-Item -ItemType Directory -Force -Path "E:\bases\mcp_ssl"
```

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_ssl:/app/zvec_db" `
  comol/mcp_ssl_server:latest-beta
```

### С CPU (без GPU)

```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -v "E:/bases/mcp_ssl:/app/zvec_db" `
  comol/mcp_ssl_server:latest-beta
```

{% hint style="warning" %}
Замените `3.1.11` на вашу версию БСП!
{% endhint %}

## Первый запуск

При первом запуске происходит:

1. Скачивание образа
2. Скачивание справки БСП указанной версии
3. Индексация документации

### Мониторинг

```powershell
docker logs -f mcp_ssl_server
```

## Проверка работы

```powershell
docker ps --filter name=mcp_ssl_server

# Готовность отвечать на поиск (её же читает HEALTHCHECK образа)
Invoke-RestMethod -Uri "http://localhost:8008/ready"
```

Состояние компонентов доступно и через MCP-инструменты: `vector_store_state` (рантайм zvec, обслуживающее поколение, итог последней миграции), `embedding_state` (провайдер эмбеддингов и его режим), `session_state` (границы и счётчики HTTP-сессий), `plugin_state` (загруженные плагины).

## Остановка

```powershell
docker stop mcp_ssl_server
```

По сигналу остановки сервер перестаёт принимать новые пакеты индексации, доводит начатое до контрольной точки и сбрасывает векторное хранилище — всё в пределах `SHUTDOWN_GRACE_SECONDS` (по умолчанию 10 секунд, ровно столько даёт `docker stop` до SIGKILL). Прерванная индексация не повреждает обслуживающее поколение.

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-ssl-mcp": {
      "url": "http://localhost:8008/mcp",
      "connection_id": "1c_ssl_service_001"
    }
  }
}
```

## Смена версии БСП

При обновлении БСП в вашей конфигурации:

```powershell
# Остановить и удалить контейнер
docker rm -f mcp_ssl_server

# Запустить с новой версией и переиндексацией
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.2.1 `
  -e RESET_DATABASE=true `
  -v "E:/bases/mcp_ssl:/app/zvec_db" `
  comol/mcp_ssl_server:latest-beta
```
