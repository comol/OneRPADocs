# Проблемы индексации

## Индексация занимает слишком много времени

### Нормальное время индексации

| Сервер | CPU (e5-small) | GPU (Qwen-4B) |
|--------|---------------|---------------|
| HelpSearchServer | 30-60 мин | 5-10 мин |
| SSLSearchServer | 20-40 мин | 3-5 мин |
| CodeMetadataSearchServer | 1-3 часа | 15-30 мин |

### Ускорение

1. **Используйте LM Studio с GPU** вместо CPU модели
2. **Увеличьте ресурсы Docker**
3. **Используйте SSD** для томов

### Мониторинг прогресса

```powershell
docker logs -f <container_name>
```

## Ошибка при смене embedding модели

### Симптом

```
DIMENSION MISMATCH DETECTED!
```

### Решение

Это нормальное поведение. Система автоматически пересоздаст индекс.

Если нужно принудительно:

```powershell
docker rm -f <container_name>

docker run -d -p PORT:PORT `
    --name <container_name> `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e RESET_DATABASE=true `
    # ... остальные параметры
```

## Ошибка скачивания модели

### Симптом

```
Error downloading model from Hugging Face
```

### Решение

1. Проверьте интернет-соединение
2. Проверьте доступ к huggingface.co
3. Используйте VPN если сайт заблокирован
4. Скачайте модель заранее в кэш

## Ошибка подключения к LM Studio

### Симптом

```
Connection refused to http://host.docker.internal:1234
```

### Решение

1. Убедитесь, что LM Studio запущен
2. Убедитесь, что Server запущен в LM Studio
3. Проверьте порт (1234 по умолчанию)
4. Проверьте, что модель загружена

```powershell
# Проверка из хоста
curl http://localhost:1234/v1/models
```

## Недостаточно памяти

### Симптом

```
RuntimeError: Unable to allocate memory
```

### Решение

1. Увеличьте память Docker:
   - Docker Desktop → Settings → Resources → Memory

2. Используйте модель меньшего размера:
   ```env
   EMBEDDING_MODEL=intfloat/multilingual-e5-small
   ```

3. Закройте другие приложения

## Индекс повреждён

### Симптом

Ошибки при поиске, некорректные результаты.

### Решение

Пересоздайте индекс:

```powershell
# Остановить контейнер
docker stop <container_name>

# Удалить данные индекса
Remove-Item -Recurse -Force "E:\bases\mcp_XXX\*"

# Запустить с переиндексацией
docker start <container_name>
# или запустить заново с RESET_DATABASE=true
```

## Файлы не найдены

### Симптом

```
FileNotFoundError: [path] not found
```

### Проверка путей

```powershell
# Проверить путь на хосте
Test-Path "C:\Program Files\1cv8\8.3.23.1997\bin"

# Проверить содержимое
Get-ChildItem "C:\Program Files\1cv8\8.3.23.1997\bin"
```

### Формат путей для Docker

```powershell
# Правильно
-v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs"

# Неправильно (обратные слеши без экранирования)
-v "C:\Program Files\1cv8\8.3.23.1997\bin:/1c_docs"
```

## Логи индексации

### Просмотр в реальном времени

```powershell
docker logs -f <container_name>
```

### Сохранение в файл

```powershell
docker logs <container_name> > indexing.log 2>&1
```

### Поиск ошибок

```powershell
docker logs <container_name> 2>&1 | Select-String -Pattern "error|Error|ERROR"
```
