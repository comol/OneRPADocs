# Установка HelpSearchServer

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен с моделью Qwen3-Embedding (рекомендуется)
3. Установлена платформа 1С:Предприятие

## Определение пути к справке

### Поиск папки bin

Справка платформы находится в папке `bin` установленной версии 1С.

Типичные пути:
- `C:\Program Files\1cv8\8.3.23.1997\bin`
- `C:\Program Files (x86)\1cv8\8.3.23.1997\bin`

### Проверка наличия справки

```powershell
# Проверить наличие файлов справки
Get-ChildItem "C:\Program Files\1cv8\8.3.23.1997\bin\*.chm"
```

Должны быть файлы типа `1cv8.chm`, `1cv8_ru.chm`.

## Создание папки для индекса

```powershell
New-Item -ItemType Directory -Force -Path "E:\bases\mcp_docs"
```

## Команды запуска

### С LM Studio (рекомендуется)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

### С CPU (без GPU)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

{% hint style="warning" %}
Замените `8.3.23.1997` на вашу версию платформы!
{% endhint %}

## Первый запуск

### Процесс индексации

При первом запуске происходит:

1. **Скачивание образа** (~2-3 ГБ)
2. **Загрузка embedding модели** (если CPU режим)
3. **Индексация справки** (5-60 минут в зависимости от модели)

### Мониторинг прогресса

```powershell
# Просмотр логов в реальном времени
docker logs -f 1c_help_mcp
```

Пример вывода:
```
Starting HelpSearchServer...
Loading embedding model...
Indexing documentation from /1c_docs...
Indexed 1000/5432 documents...
Indexed 2000/5432 documents...
...
Indexing complete! Starting MCP server on port 8003
```

### Проверка готовности

```powershell
# Проверить статус контейнера
docker ps --filter name=1c_help_mcp

# Проверить доступность
curl http://localhost:8003/health
```

## Последующие запуски

После первой индексации используйте `RESET_DATABASE=false`:

```powershell
# Остановка
docker stop 1c_help_mcp

# Запуск (индекс сохранён)
docker start 1c_help_mcp
```

## Обновление справки

При обновлении версии платформы 1С:

1. Остановите контейнер
2. Измените путь к новой версии
3. Установите `RESET_DATABASE=true`
4. Запустите контейнер

```powershell
docker stop 1c_help_mcp
docker rm 1c_help_mcp

docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=true `
  -v "C:/Program Files/1cv8/8.3.24.1234/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

## Проверка работы

### Тест MCP endpoint

```powershell
# Простая проверка доступности
Invoke-RestMethod -Uri "http://localhost:8003/health"
```

### Настройка Cursor

1. Откройте `%APPDATA%\Cursor\User\globalStorage\mcp.json`
2. Добавьте конфигурацию сервера
3. Перезапустите Cursor
4. Задайте вопрос о методах 1С

## Устранение проблем

### Контейнер не запускается

```powershell
# Проверить логи
docker logs 1c_help_mcp
```

### Ошибка "path not found"

Проверьте путь к папке bin:
```powershell
Test-Path "C:\Program Files\1cv8\8.3.23.1997\bin"
```

### Ошибка подключения к LM Studio

1. Убедитесь, что LM Studio Server запущен
2. Проверьте порт 1234
3. Используйте `host.docker.internal` вместо `localhost`
