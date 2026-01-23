# Установка FormsServer

## Предварительные требования

- Docker Desktop запущен

## Команда запуска

```powershell
docker run -d -p 8011:8011 `
  --name 1c_forms_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_forms:latest
```

## Переменные окружения

| Переменная | Описание | Обязательная |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да |
| `USESSE` | SSE транспорт | Нет |
| `PORT` | Порт сервера | Нет (default: 8011) |

## Проверка работы

### Статус контейнера

```powershell
docker ps --filter name=1c_forms_mcp
```

### Просмотр логов

```powershell
docker logs 1c_forms_mcp
```

## Конфигурация Cursor

### mcp.json

```json
{
  "mcpServers": {
    "1c-forms-mcp": {
      "url": "http://localhost:8011/mcp",
      "connection_id": "1c_forms_service_001"
    }
  }
}
```

## Использование

После подключения ИИ может:

1. Получить схему формы: вызов `get_xsd_schema()` или `get_json_schema()`
2. Понять структуру элементов формы
3. Сгенерировать валидный XML

### Пример сценария

Пользователь: "Создай форму документа с таблицей товаров"

ИИ:
1. Запрашивает схему через `get_json_schema()`
2. Анализирует доступные элементы
3. Генерирует XML формы

## Управление контейнером

```powershell
# Остановить
docker stop 1c_forms_mcp

# Запустить
docker start 1c_forms_mcp

# Удалить
docker rm -f 1c_forms_mcp
```
