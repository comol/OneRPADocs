# Порты и эндпоинты

Сводная таблица портов всех MCP-серверов.

## Порты серверов

| Сервер | Порт | MCP Endpoint | Дополнительные |
|--------|------|--------------|----------------|
| CodeMetadataSearchServer | 8000 | `/mcp` | — |
| SyntaxCheckServer | 8002 | `/mcp` | — |
| HelpSearchServer | 8003 | `/mcp` | `/health` |
| TemplatesSearchServer | 8004 | `/mcp` | `/extend/` (веб-интерфейс) |
| Graph Metadata Search | 8006 | `/mcp` | `/search` (веб-интерфейс) |
| 1CCodeChecker | 8007 | `/mcp` | `/health` |
| SSLSearchServer | 8008 | `/mcp` | — |
| FormsServer | 8011 | `/mcp` | — |

## Neo4j (для Graph Metadata Search)

| Порт | Назначение |
|------|------------|
| 7474 | Neo4j Browser (HTTP) |
| 7687 | Neo4j Bolt Protocol |

## Полные URL для mcp.json

```json
{
  "mcpServers": {
    "1c-code-metadata-mcp": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_metadata_service_001"
    },
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    },
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    },
    "1c-templates-mcp": {
      "url": "http://localhost:8004/mcp",
      "connection_id": "1c_templates_service_001"
    },
    "1c-graph-metadata-mcp": {
      "url": "http://localhost:8006/mcp",
      "connection_id": "1c_graph_service_001"
    },
    "1c-code-checker-mcp": {
      "url": "http://localhost:8007/mcp",
      "connection_id": "1c_code_checker_001"
    },
    "1c-ssl-mcp": {
      "url": "http://localhost:8008/mcp",
      "connection_id": "1c_ssl_service_001"
    },
    "1c-forms-mcp": {
      "url": "http://localhost:8011/mcp",
      "connection_id": "1c_forms_service_001"
    }
  }
}
```

## Проверка доступности

### PowerShell скрипт

```powershell
$servers = @(
    @{Name="CodeMetadataSearchServer"; Port=8000; Endpoint="/mcp"},
    @{Name="SyntaxCheckServer"; Port=8002; Endpoint="/mcp"},
    @{Name="HelpSearchServer"; Port=8003; Endpoint="/mcp"},
    @{Name="TemplatesSearchServer"; Port=8004; Endpoint="/mcp"},
    @{Name="GraphMetadataSearch"; Port=8006; Endpoint="/mcp"},
    @{Name="1CCodeChecker"; Port=8007; Endpoint="/mcp"},
    @{Name="SSLSearchServer"; Port=8008; Endpoint="/mcp"},
    @{Name="FormsServer"; Port=8011; Endpoint="/mcp"}
)

foreach ($server in $servers) {
    $result = Test-NetConnection -ComputerName localhost -Port $server.Port -WarningAction SilentlyContinue
    $status = if ($result.TcpTestSucceeded) { "✓" } else { "✗" }
    $color = if ($result.TcpTestSucceeded) { "Green" } else { "Red" }
    Write-Host "$status $($server.Name) - localhost:$($server.Port)$($server.Endpoint)" -ForegroundColor $color
}
```

## Изменение портов

Если стандартный порт занят, можно использовать другой:

```powershell
# Вместо 8002 использовать 8102
docker run -d -p 8102:8002 `
    --name 1c_syntaxcheck_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    comol/1c_syntaxcheck_mcp:latest
```

Не забудьте обновить mcp.json:
```json
"url": "http://localhost:8102/mcp"
```
