# Кеширование БД

Сохранение векторных баз данных и кэша моделей между перезапусками.

## Зачем кешировать

- **Экономия времени**: индексация занимает минуты-часы
- **Экономия трафика**: модели не скачиваются повторно
- **Стабильность**: данные сохраняются при обновлении контейнера

## Ключевые тома

| Том в контейнере | Назначение |
|-----------------|------------|
| `/app/chroma_db` | Векторная база данных ChromaDB |
| `/app/model_cache` | Кэш embedding моделей |
| `/app/data` | Данные сервера (шаблоны и т.д.) |

## Рекомендуемые пути на хосте

```
E:\bases\
├── mcp_docs\          # HelpSearchServer - chroma_db
├── mcp_codemetadata\  # CodeMetadataSearchServer - chroma_db
├── mcp_ssl\           # SSLSearchServer - chroma_db
├── mcp_templates\     # TemplatesSearchServer - data
├── mcp_graph\         # Graph Metadata Search - neo4j
└── mcp_model_cache\   # Общий кэш моделей
```

## Параметры управления кэшем

### RESET_DATABASE

```env
RESET_DATABASE=false  # Использовать существующий индекс
RESET_DATABASE=true   # Переиндексировать при запуске
```

**Когда использовать `true`:**
- Первый запуск
- Изменились исходные данные
- Сменилась embedding модель

**Когда использовать `false`:**
- Обычный перезапуск
- Обновление контейнера
- Тестирование

### RESET_CACHE

```env
RESET_CACHE=false  # Использовать скачанную модель
RESET_CACHE=true   # Загрузить модель заново
```

## Примеры монтирования томов

### HelpSearchServer

```powershell
docker run -d -p 8003:8003 `
    --name 1c_help_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e RESET_DATABASE=false `
    -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
    -v "E:/bases/mcp_docs:/app/chroma_db" `
    -v "E:/bases/mcp_model_cache:/app/model_cache" `
    comol/1c_help_mcp:latest
```

### SSLSearchServer

```powershell
docker run -d -p 8008:8008 `
    --name mcp_ssl_server `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e SSL_VERSION=3.1.11 `
    -e RESET_DATABASE=false `
    -v "E:/bases/mcp_ssl:/app/chroma_db" `
    comol/mcp_ssl_server:latest
```

### TemplatesSearchServer

```powershell
docker run -d -p 8004:8004 `
    --name template_search_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e RESET_DATABASE=false `
    -v "E:/bases/mcp_templates:/app/data" `
    comol/template-search-mcp:latest
```

## Проверка кэша

### Размер папок

```powershell
Get-ChildItem "E:\bases" -Directory | ForEach-Object {
    $size = (Get-ChildItem $_.FullName -Recurse | Measure-Object -Property Length -Sum).Sum / 1MB
    Write-Host "$($_.Name): $([math]::Round($size, 2)) MB"
}
```

### Содержимое ChromaDB

```powershell
Get-ChildItem "E:\bases\mcp_docs" -Recurse
```

## Резервное копирование

### Создание бэкапа

```powershell
$date = Get-Date -Format "yyyyMMdd"
Compress-Archive -Path "E:\bases\*" -DestinationPath "E:\backup\mcp_bases_$date.zip"
```

### Восстановление

```powershell
Expand-Archive -Path "E:\backup\mcp_bases_20260123.zip" -DestinationPath "E:\bases" -Force
```

## Очистка кэша

### Полная очистка

```powershell
# Остановить контейнеры
docker stop 1c_help_mcp mcp_ssl_server template_search_mcp

# Удалить данные
Remove-Item -Recurse -Force "E:\bases\mcp_docs\*"
Remove-Item -Recurse -Force "E:\bases\mcp_ssl\*"
Remove-Item -Recurse -Force "E:\bases\mcp_templates\*"

# Запустить с переиндексацией
# ... (команды с RESET_DATABASE=true)
```

### Очистка только индекса

```powershell
# Сохранить кэш моделей, удалить только индексы
Remove-Item -Recurse -Force "E:\bases\mcp_docs\chroma.sqlite3" 2>$null
Remove-Item -Recurse -Force "E:\bases\mcp_docs\*" -Exclude "model_cache" 2>$null
```

## Миграция на другой компьютер

1. Остановите контейнеры
2. Скопируйте папку `E:\bases\` на новый компьютер
3. Запустите контейнеры с теми же путями
4. Используйте `RESET_DATABASE=false`
