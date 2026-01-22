# API Reference

Справочник по REST API Proxy-сервера OneAPA.

## Базовый URL

```
http://<proxy-server>:9000
```

## Аутентификация

API не требует аутентификации для внутреннего использования. Для production рекомендуется настроить reverse proxy с авторизацией.

## Endpoints

### GET /health

Проверка работоспособности сервера.

**Запрос:**
```bash
curl http://localhost:9000/health
```

**Ответ:**
```json
{
  "message": "OK",
  "version": "1.0.1",
  "build_date": "2025-12-03"
}
```

**Коды ответа:**
| Код | Описание |
|-----|----------|
| 200 | Сервер работает |
| 500 | Внутренняя ошибка |

---

### POST /load

Загрузка агентов и инструментов из 1С.

**Запрос:**
```bash
curl -X POST http://localhost:9000/load \
  -H "Content-Type: application/json" \
  -d '{
    "agents": [...],
    "toolsendpoint": "http://1c-server/hs/oneapa/tools"
  }'
```

**Тело запроса (LoadRequest):**

```json
{
  "agents": [
    {
      "agent": {
        "id": "agent-001",
        "name": "Помощник",
        "model": {
          "service": "OpenAI",
          "token": "sk-...",
          "folder": "gpt-4o",
          "url": ""
        },
        "infobase": "my_base",
        "description": "Универсальный помощник",
        "systemprompt": "Ты — помощник...",
        "userprompt": "",
        "tools": [
          {
            "name": "get_date",
            "type": "http",
            "description": "Возвращает текущую дату",
            "format": "",
            "parameters": []
          }
        ],
        "mcps": []
      }
    }
  ],
  "toolsendpoint": "http://1c-server/hs/oneapa/tools"
}
```

**Ответ:**
```json
{
  "status": "success",
  "agents_loaded": ["agent-001"],
  "message": "Loaded 1 agents"
}
```

---

### POST /chat

Отправка сообщения агенту.

**Запрос:**
```bash
curl -X POST http://localhost:9000/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Привет!",
    "files": []
  }'
```

**Тело запроса (ChatRequest):**

```json
{
  "message": "Сколько у меня отпускных дней?",
  "files": [
    {
      "content": "base64_encoded_content",
      "name": "document.pdf"
    }
  ]
}
```

| Поле | Тип | Обязательный | Описание |
|------|-----|--------------|----------|
| message | string | Да | Текст сообщения |
| files | array | Нет | Прикреплённые файлы |

**Формат файла:**

| Поле | Тип | Описание |
|------|-----|----------|
| content | string | Base64-encoded содержимое |
| name | string | Имя файла с расширением |

**Ответ (ChatResponse):**

```json
{
  "response": "У вас осталось 14 дней отпуска.",
  "tool_results": [
    {
      "tool": "get_vacation_days",
      "result": {
        "days": 14
      }
    }
  ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| response | string | Ответ агента |
| tool_results | array | Результаты вызванных инструментов |

**Коды ответа:**

| Код | Описание |
|-----|----------|
| 200 | Успех |
| 400 | Неверный запрос |
| 500 | Внутренняя ошибка |
| 504 | Таймаут |

---

## Модели данных

### Model

```json
{
  "service": "OpenAI",
  "token": "sk-...",
  "folder": "gpt-4o",
  "url": ""
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| service | string | Провайдер: OpenAI, Yandex, LocalOllama, Sber, OpenRouter |
| token | string | API ключ |
| folder | string | Название модели |
| url | string | URL для локальных провайдеров |

### Tool

```json
{
  "name": "get_balance",
  "type": "http",
  "description": "Получить остаток на счёте",
  "format": "",
  "parameters": [
    {
      "name": "account",
      "type": "string",
      "description": "Номер счёта"
    }
  ]
}
```

### Parameter

```json
{
  "name": "account",
  "type": "string",
  "description": "Номер расчётного счёта"
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| name | string | Имя параметра |
| type | string | Тип: string, number, integer, boolean |
| description | string | Описание для LLM |

### Agent

```json
{
  "id": "agent-001",
  "name": "Помощник",
  "model": {...},
  "infobase": "my_base",
  "description": "Описание агента",
  "systemprompt": "Системный промпт",
  "userprompt": "Пользовательский промпт",
  "tools": [...],
  "mcps": [...]
}
```

## Примеры

### Python

```python
import requests

# Проверка здоровья
response = requests.get("http://localhost:9000/health")
print(response.json())

# Отправка сообщения
response = requests.post(
    "http://localhost:9000/chat",
    json={"message": "Привет!"}
)
print(response.json()["response"])
```

### JavaScript

```javascript
// Проверка здоровья
const health = await fetch('http://localhost:9000/health');
console.log(await health.json());

// Отправка сообщения
const response = await fetch('http://localhost:9000/chat', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({message: 'Привет!'})
});
const data = await response.json();
console.log(data.response);
```

### 1С

```bsl
// Проверка здоровья
HTTPСоединение = Новый HTTPСоединение("localhost", 9000);
Запрос = Новый HTTPЗапрос("/health");
Ответ = HTTPСоединение.Получить(Запрос);
Результат = Ответ.ПолучитьТелоКакСтроку();

// Отправка сообщения
Запрос = Новый HTTPЗапрос("/chat");
Запрос.Заголовки.Вставить("Content-Type", "application/json");
ТелоЗапроса = Новый Структура("message", "Привет!");
Запрос.УстановитьТелоИзСтроки(СериализоватьJSON(ТелоЗапроса));
Ответ = HTTPСоединение.ОтправитьДляОбработки(Запрос);
```

## Ограничения

| Параметр | Значение |
|----------|----------|
| Таймаут запроса | 900 секунд |
| Максимум итераций | 15 |
| Размер файла | Зависит от модели |

## Далее

- [Разработка агентов](../razrabotka-agentov/) — создание своих агентов
- [Администрирование](../administrirovanie/) — управление системой
