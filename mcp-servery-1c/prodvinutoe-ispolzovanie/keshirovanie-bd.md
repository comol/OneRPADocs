# Кеширование БД

Сохранение векторных баз данных и кэша моделей между перезапусками.

{% hint style="danger" %}
**КРИТИЧЕСКИ ВАЖНО!** Обязательно монтируйте папки с векторными БД к контейнеру через тома (`-v`). Индексация может занимать от нескольких часов до суток. Без монтирования тома все индексы будут **потеряны** при остановке или перезапуске контейнера, и индексацию придётся начинать заново!
{% endhint %}

## Зачем кешировать

- **Экономия времени**: индексация может занимать от 2 до 20+ часов в зависимости от объёма данных
- **Экономия трафика**: модели не скачиваются повторно
- **Стабильность**: данные сохраняются при обновлении контейнера
- **Защита от потери данных**: при сбое Docker или перезагрузке системы индексы сохранятся

## Ключевые тома

| Том в контейнере | Сервер | Назначение |
|-----------------|--------|------------|
| `/app/index` | HelpSearchServer | Индекс zvec (поколения) |
| `/app/zvec_db` | SSLSearchServer | Векторная база zvec |
| `/app/chroma_db` | CodeMetadataSearchServer, TemplatesSearchServer | Каталог векторного хранилища (историческое имя тома; движок — zvec) |
| `/app/model_cache` | Все | Кэш embedding моделей |
| `/app/data` | Отдельные серверы | Прочие данные сервера |

{% hint style="info" %}
Имя `chroma_db` у части серверов сохранено как путь тома ради совместимости с существующими установками — ChromaDB внутри уже не используется. У CodeMetadataSearchServer каталог задаётся переменной `VECTOR_DB_PATH` (старое имя `CHROMA_DB_PATH` работает как алиас).
{% endhint %}

## Рекомендуемые пути на хосте

{% hint style="info" %}
Путь `E:\bases\` приведён как **пример**. Вы можете использовать любой удобный путь на вашем компьютере, например `D:\mcp_data\` или `C:\Users\<ваш_пользователь>\mcp_bases\`. Главное — это должен быть SSD-диск для лучшей производительности.
{% endhint %}

```
E:\bases\                    # <-- Пример, используйте свой путь
├── mcp_docs\                # HelpSearchServer — index (zvec)
├── mcp_codemetadata\        # CodeMetadataSearchServer — chroma_db (zvec)
├── mcp_ssl\                 # SSLSearchServer — zvec_db
├── mcp_templates\           # TemplatesSearchServer — chroma_db (zvec + SQLite)
├── mcp_graph\               # Graph Metadata Search — Neo4j
└── mcp_model_cache\         # Общий кэш моделей
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

{% hint style="warning" %}
При `RESET_DATABASE=true` полная переиндексация может занять от 2 до 20+ часов в зависимости от объёма данных и используемой модели. Планируйте переиндексацию заранее, например, запускайте её на ночь.
{% endhint %}

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
    -e 1C_BIN_PATH=/1c_docs `
    -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
    -v "E:/bases/mcp_docs:/app/index" `
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
    -v "E:/bases/mcp_ssl:/app/zvec_db" `
    comol/mcp_ssl_server:latest
```

### TemplatesSearchServer

```powershell
docker run -d -p 8004:8004 `
    --name template_search_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e RESET_DATABASE=false `
    -v "E:/bases/mcp_templates:/app/chroma_db" `
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

### Содержимое индекса

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
Remove-Item -Recurse -Force "E:\bases\mcp_docs\*" -Exclude "model_cache" 2>$null
```

## Миграция на другой компьютер

1. Остановите контейнеры
2. Скопируйте папку `E:\bases\` на новый компьютер
3. Запустите контейнеры с теми же путями
4. Используйте `RESET_DATABASE=false`
