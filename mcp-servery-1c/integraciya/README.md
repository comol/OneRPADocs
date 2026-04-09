# Интеграция с Cursor

Настройка подключения MCP-серверов к Cursor IDE.

## Обзор

MCP-серверы подключаются к Cursor через файл конфигурации `mcp.json`. Каждый сервер регистрируется как отдельный MCP-провайдер.

## Содержание

- [Формат mcp.json](cursor-mcp-json.md) — структура конфигурации
- [Несколько серверов](neskolko-serverov.md) — подключение всех серверов
- [Cursor Rules для 1С](cursor-rules.md) — правила для эффективной работы

## Быстрый старт

### 1. Найдите файл mcp.json

```
%APPDATA%\Cursor\User\globalStorage\mcp.json
```

### 2. Добавьте конфигурацию сервера

```json
{
  "mcpServers": {
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    }
  }
}
```

### 3. Перезапустите Cursor

Закройте и откройте Cursor для применения изменений.

### 4. Проверьте подключение

Задайте вопрос, связанный с MCP-сервером. ИИ должен использовать инструменты сервера.

## Порядок подключения

Рекомендуемый порядок добавления серверов:

1. **SyntaxCheckServer** — проверка синтаксиса
2. **HelpSearchServer** — справка платформы
3. **SSLSearchServer** — справка БСП
4. **TemplatesSearchServer** — шаблоны кода
5. **CodeMetadataSearchServer** — метаданные конфигурации
6. **Graph Metadata Search** — графовый поиск
7. **1CCodeChecker** — 1С:Напарник
