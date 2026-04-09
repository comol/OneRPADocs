# Несколько серверов

Подключение нескольких MCP-серверов одновременно.

## Преимущества

Каждый сервер специализируется на своей задаче:

- **HelpSearchServer** — справка платформы
- **SSLSearchServer** — справка БСП
- **SyntaxCheckServer** — проверка синтаксиса
- **TemplatesSearchServer** — шаблоны кода

Вместе они дают ИИ полный контекст для разработки на 1С.

## Несколько баз или версий платформы

Если вы работаете с **несколькими информационными базами** или **разными версиями платформы 1С**, можно запустить несколько экземпляров одного MCP-сервера на разных портах.

### Пример: две версии платформы

```powershell
# HelpSearchServer для платформы 8.3.23
docker run -d -p 8003:8003 `
  --name 1c_help_mcp_8323 `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs_8323:/app/chroma_db" `
  comol/1c_help_mcp:latest

# HelpSearchServer для платформы 8.3.25
docker run -d -p 8013:8003 `
  --name 1c_help_mcp_8325 `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -v "C:/Program Files/1cv8/8.3.25.1234/bin:/1c_docs" `
  -v "E:/bases/mcp_docs_8325:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

### Пример: несколько конфигураций

```powershell
# CodeMetadataSearchServer для конфигурации "Бухгалтерия"
docker run -d -p 8000:8000 `
  --name 1c_metadata_buh `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "E:/1C_Export/Buhgalteriya/Report:/app/metadata" `
  -v "E:/1C_Export/Buhgalteriya/Files:/app/code" `
  -v "E:/bases/mcp_metadata_buh:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest

# CodeMetadataSearchServer для конфигурации "УТ"
docker run -d -p 8010:8000 `
  --name 1c_metadata_ut `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "E:/1C_Export/UT/Report:/app/metadata" `
  -v "E:/1C_Export/UT/Files:/app/code" `
  -v "E:/bases/mcp_metadata_ut:/app/chroma_db" `
  comol/1c_code_metadata_mcp:latest
```

### Конфигурация Cursor для нескольких экземпляров

```json
{
  "mcpServers": {
    "1c-docs-8323": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_8323"
    },
    "1c-docs-8325": {
      "url": "http://localhost:8013/mcp",
      "connection_id": "1c_docs_8325"
    },
    "1c-metadata-buh": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_metadata_buh"
    },
    "1c-metadata-ut": {
      "url": "http://localhost:8010/mcp",
      "connection_id": "1c_metadata_ut"
    }
  }
}
```

{% hint style="info" %}
**Важно:** Для каждого экземпляра используйте:
- Уникальный порт на хосте (например, 8003, 8013, 8023)
- Уникальное имя контейнера (`--name`)
- Отдельную папку для индекса (`-v "...:/app/chroma_db"`)
- Уникальный `connection_id` в mcp.json
{% endhint %}

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
    @{Name="SSLSearchServer"; Port=8008}
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
