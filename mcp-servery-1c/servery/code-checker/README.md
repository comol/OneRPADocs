# 1CCodeChecker

Проверка кода через 1С:Напарник.

## Назначение

1CCodeChecker интегрируется с сервисом 1С:Напарник для глубокого анализа кода 1С. Проверяет стиль, архитектуру и потенциальные проблемы.

## Возможности

ИИ получает инструменты для:

- Проверки стиля кода
- Обнаружения архитектурных ошибок
- Выявления потенциальных проблем
- Анализа технического долга

## Примеры использования

- "Проверь этот код через 1С:Напарник"
- "Найди проблемы в этом модуле"
- "Проанализируй код на соответствие стандартам"

## Требования

- Docker Desktop с WSL2
- Лицензионный ключ MCP
- **Токен 1С:Напарник** (только для партнёров 1С)
- Доступ в интернет к code.1c.ai

{% hint style="warning" %}
Токен 1С:Напарник можно получить только партнёрам фирмы 1С. Если у вас нет токена, используйте [SyntaxCheckServer](../syntax-check-server/) как альтернативу.
{% endhint %}

## Порт

**8007**

## Образ Docker

```
comol/1c-code-checker:latest
```

## Быстрый старт

```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIR_TOKEN `
  comol/1c-code-checker:latest
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

## Оригинальная идея

[github.com/artesk/1copilot_MCP](https://github.com/artesk/1copilot_MCP)

## Структура раздела

- [Установка](ustanovka.md) — команды запуска
- [Получение токена](poluchenie-tokena.md) — как получить токен
