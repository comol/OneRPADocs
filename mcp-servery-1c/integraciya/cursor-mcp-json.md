# Формат mcp.json

Файл `mcp.json` содержит конфигурацию MCP-серверов для Cursor.

## Расположение файла

```
%APPDATA%\Cursor\User\globalStorage\mcp.json
```

Обычно:
```
C:\Users\<username>\AppData\Roaming\Cursor\User\globalStorage\mcp.json
```

## Структура файла

```json
{
  "mcpServers": {
    "имя-сервера": {
      "url": "http://localhost:PORT/mcp",
      "connection_id": "уникальный_идентификатор"
    }
  }
}
```

## Параметры

| Параметр | Описание |
|----------|----------|
| `имя-сервера` | Уникальное имя для идентификации в Cursor |
| `url` | URL MCP endpoint сервера |
| `connection_id` | Уникальный идентификатор подключения |

## Пример: один сервер

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

## Пример: все серверы

```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    },
    "1c-code-metadata-mcp": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_metadata_service_001"
    },
    "1c-graph-metadata-mcp": {
      "url": "http://localhost:8006/mcp",
      "connection_id": "1c_graph_service_001"
    },
    "1c-ssl-mcp": {
      "url": "http://localhost:8008/mcp",
      "connection_id": "1c_ssl_service_001"
    },
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    },
    "1c-templates-mcp": {
      "url": "http://localhost:8004/mcp",
      "connection_id": "1c_templates_service_001"
    },
    "1c-forms-mcp": {
      "url": "http://localhost:8011/mcp",
      "connection_id": "1c_forms_service_001"
    },
    "1c-code-checker-mcp": {
      "url": "http://localhost:8007/mcp",
      "connection_id": "1c_code_checker_001"
    }
  }
}
```

## Создание файла через PowerShell

```powershell
# Путь к файлу
$mcpPath = "$env:APPDATA\Cursor\User\globalStorage\mcp.json"

# Создать директорию если не существует
New-Item -ItemType Directory -Force -Path (Split-Path $mcpPath)

# Создать конфигурацию
$config = @{
    mcpServers = @{
        "1c-syntax-checker-mcp" = @{
            url = "http://localhost:8002/mcp"
            connection_id = "1c_lsp_service_001"
        }
    }
}

# Сохранить в файл
$config | ConvertTo-Json -Depth 3 | Out-File -FilePath $mcpPath -Encoding utf8

Write-Host "Файл создан: $mcpPath"
```

## Валидация JSON

Проверьте корректность JSON перед сохранением:

```powershell
# Прочитать и проверить
$content = Get-Content "$env:APPDATA\Cursor\User\globalStorage\mcp.json" -Raw
try {
    $json = $content | ConvertFrom-Json
    Write-Host "JSON валиден" -ForegroundColor Green
} catch {
    Write-Host "Ошибка JSON: $_" -ForegroundColor Red
}
```

## После изменения

1. Сохраните файл
2. Закройте все окна Cursor
3. Откройте Cursor заново
4. Проверьте подключение в чате

## Частые ошибки

### Неправильный JSON

```json
// Неправильно - запятая после последнего элемента
{
  "mcpServers": {
    "server": {
      "url": "http://localhost:8002/mcp",
    }
  }
}
```

```json
// Правильно
{
  "mcpServers": {
    "server": {
      "url": "http://localhost:8002/mcp"
    }
  }
}
```

### Дублирование имён

Каждый сервер должен иметь уникальное имя в `mcpServers`.

### Неправильный порт

Убедитесь, что порт соответствует запущенному контейнеру.
