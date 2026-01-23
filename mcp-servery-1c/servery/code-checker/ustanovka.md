# Установка 1CCodeChecker

## Предварительные требования

1. Docker Desktop запущен
2. Токен 1С:Напарник (см. [Получение токена](poluchenie-tokena.md))
3. Доступ в интернет к code.1c.ai

## Команда запуска

```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIR_TOKEN `
  comol/1c-code-checker:latest
```

## Переменные окружения

| Переменная | Описание | Обязательная |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ MCP | Да |
| `ONEC_AI_TOKEN` | Токен 1С:Напарник | Да |
| `USESSE` | SSE транспорт | Нет |

## Проверка работы

### Статус контейнера

```powershell
docker ps --filter name=1c_code_checker
```

### Просмотр логов

```powershell
docker logs 1c_code_checker
```

## Конфигурация Cursor

```json
{
  "mcpServers": {
    "1c-code-checker-mcp": {
      "url": "http://localhost:8007/mcp",
      "connection_id": "1c_code_checker_001"
    }
  }
}
```

## Использование

### Инструмент MCP

Сервер предоставляет инструмент `check_1c_code`:

**Параметры:**
- `code` (обязательный) — код 1С для проверки
- `check_type` (опциональный) — тип проверки: `syntax`, `logic`, `performance`

### Пример в Cursor

Пользователь: "Проверь этот код"

```bsl
Процедура ТестоваяПроцедура()
    Сообщить("Тест");
КонецПроцедуры
```

ИИ вызывает `check_1c_code` и возвращает результат анализа.

## Сравнение с SyntaxCheckServer

| Аспект | 1CCodeChecker | SyntaxCheckServer |
|--------|---------------|-------------------|
| Провайдер | 1С:Напарник | BSL Language Server |
| Синтаксис | ✅ | ✅ |
| Логика | ✅ | ❌ |
| Стиль | ✅ | Частично |
| Требует токен | Да | Нет |
| Работает офлайн | Нет | Да |

## Устранение проблем

### Ошибка авторизации

Проверьте токен 1С:Напарник:
- Токен действителен
- Токен правильно указан в `ONEC_AI_TOKEN`

### Нет связи с code.1c.ai

Проверьте:
- Доступ в интернет из контейнера
- Firewall не блокирует code.1c.ai
