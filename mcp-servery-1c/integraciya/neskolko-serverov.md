# Несколько серверов

Подключение нескольких MCP-серверов одновременно.

## Преимущества

Каждый сервер специализируется на своей задаче:

- **HelpSearchServer** — справка платформы
- **SSLSearchServer** — справка БСП
- **SyntaxCheckServer** — проверка синтаксиса
- **TemplatesSearchServer** — шаблоны кода

Вместе они дают ИИ полный контекст для разработки на 1С.

## Полная конфигурация

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

## Рекомендуемые комбинации

### Минимальный набор

Для быстрого старта без подготовки данных:

```json
{
  "mcpServers": {
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    },
    "1c-forms-mcp": {
      "url": "http://localhost:8011/mcp",
      "connection_id": "1c_forms_service_001"
    }
  }
}
```

### Стандартный набор

Для большинства разработчиков:

```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
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
    }
  }
}
```

### Полный набор с метаданными

Для работы с конкретной конфигурацией:

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
    }
  }
}
```

## Проверка всех серверов

### PowerShell скрипт

```powershell
$servers = @(
    @{Name="HelpSearchServer"; Port=8003},
    @{Name="CodeMetadataSearchServer"; Port=8000},
    @{Name="SyntaxCheckServer"; Port=8002},
    @{Name="TemplatesSearchServer"; Port=8004},
    @{Name="GraphMetadataSearch"; Port=8006},
    @{Name="1CCodeChecker"; Port=8007},
    @{Name="SSLSearchServer"; Port=8008},
    @{Name="FormsServer"; Port=8011}
)

foreach ($server in $servers) {
    $result = Test-NetConnection -ComputerName localhost -Port $server.Port -WarningAction SilentlyContinue
    if ($result.TcpTestSucceeded) {
        Write-Host "✓ $($server.Name) (порт $($server.Port))" -ForegroundColor Green
    } else {
        Write-Host "✗ $($server.Name) (порт $($server.Port))" -ForegroundColor Red
    }
}
```

## Порядок использования ИИ

Когда вы задаёте вопрос, ИИ:

1. Анализирует вопрос
2. Определяет, какие MCP-серверы могут помочь
3. Вызывает инструменты соответствующих серверов
4. Объединяет результаты в ответ

### Пример комбинированного запроса

Вопрос: "Напиши процедуру для получения остатков товаров и проверь синтаксис"

ИИ:
1. Использует **TemplatesSearchServer** для поиска шаблона
2. Использует **HelpSearchServer** для уточнения методов
3. Генерирует код
4. Использует **SyntaxCheckServer** для проверки
