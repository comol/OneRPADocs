# Настройка Cursor IDE

Cursor — это IDE на базе VS Code с встроенной поддержкой ИИ и протокола MCP.

## Установка Cursor

1. Перейдите на [cursor.com](https://cursor.com/)
2. Скачайте версию для Windows
3. Установите приложение
4. Войдите в аккаунт или создайте новый

## Расположение файла mcp.json

Файл `mcp.json` содержит конфигурацию MCP-серверов для Cursor.

### Путь к файлу

```
%APPDATA%\Cursor\User\globalStorage\mcp.json
```

Обычно это:

```
C:\Users\<username>\AppData\Roaming\Cursor\User\globalStorage\mcp.json
```

### Создание файла

Если файл не существует, создайте его:

```powershell
# Создание папки (если не существует)
New-Item -ItemType Directory -Force -Path "$env:APPDATA\Cursor\User\globalStorage"

# Создание пустого файла конфигурации
@'
{
  "mcpServers": {
  }
}
'@ | Out-File -FilePath "$env:APPDATA\Cursor\User\globalStorage\mcp.json" -Encoding utf8
```

## Формат конфигурации

### Один сервер

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

### Несколько серверов

```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    },
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    },
    "1c-ssl-mcp": {
      "url": "http://localhost:8008/mcp",
      "connection_id": "1c_ssl_service_001"
    }
  }
}
```

## Проверка подключения

### Перезапуск Cursor

После изменения `mcp.json` перезапустите Cursor:
1. Закройте все окна Cursor
2. Откройте Cursor заново

### Проверка в чате

1. Откройте чат с ИИ в Cursor (Ctrl+L)
2. Спросите что-нибудь, связанное с MCP-сервером
3. Убедитесь, что ИИ использует инструменты MCP

## Cursor Rules для 1С

Для эффективной работы с MCP-серверами добавьте правила для 1С.

### Установка правил

1. Склонируйте репозиторий правил:

```powershell
git clone https://github.com/comol/cursor_rules_1c.git
```

2. Скопируйте файлы в ваш проект или глобальные настройки Cursor

### Рекомендации

- Используйте правила из репозитория как основу
- Адаптируйте правила под вашу конфигурацию
- Включите информацию об MCP-инструментах в правила

## Устранение проблем

### Cursor не видит MCP-серверы

1. Проверьте, что контейнеры запущены: `docker ps`
2. Проверьте формат JSON в `mcp.json`
3. Убедитесь, что порты соответствуют запущенным контейнерам
4. Перезапустите Cursor

### Ошибка "Connection refused"

1. Проверьте, что контейнер запущен
2. Проверьте порт: `netstat -an | findstr :8002`
3. Проверьте логи контейнера: `docker logs <container_name>`

### MCP работает, но ИИ не использует инструменты

1. Добавьте Cursor Rules для 1С
2. Явно попросите ИИ использовать MCP-инструменты
3. Проверьте, что в правилах указаны доступные инструменты
