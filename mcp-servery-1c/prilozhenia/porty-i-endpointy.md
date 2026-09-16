# Порты и эндпоинты

Сводная таблица портов всех MCP-серверов.

## Порты серверов

По умолчанию современные образы используют `streamable-http` на `/mcp`. Если клиент требует SSE, включите `USESSE=true`; у Graph Metadata Search эта переменная называется `MCP_USE_SSE`. У большинства серверов URL остаётся `/mcp`; SyntaxCheckServer использует стандартные SSE-пути `/sse` и `/messages/`.

Graph Metadata Search по умолчанию обслуживает `streamable-http` без серверной сессии и отвечает на POST обычным JSON (`FASTMCP_STATELESS_HTTP=true`, `FASTMCP_JSON_RESPONSE=true`). Для клиента, которому нужна stateful-сессия с SSE-ответами на том же `/mcp`, задайте обе переменные `false`.

| Сервер | Порт | MCP Endpoint | Дополнительные |
|--------|------|--------------|----------------|
| CodeMetadataSearchServer | 8000 | `/mcp` | `/live`, `/ready`, `/health` (устаревший алиас `/live`) |
| SyntaxCheckServer | 8002 | `/mcp` (`streamable-http`) или `/sse` (SSE) | `/messages/` (SSE) |
| HelpSearchServer | 8003 | `/mcp` | `/health`, `/ready`, `/plugins`, `/plugins/reload` |
| TemplatesSearchServer | 8004 | `/mcp` | `/health`, `/ready`, `/extend/`, `/extend/memory` (веб-интерфейс) |
| Graph Metadata Search | 8006 | `/mcp` | `/health` (`/healthz`), `/ready` (`/readyz`) |
| 1CCodeChecker | 8007 | `/mcp` | `/health`, `/ready`, `/metrics/sessions`, `/release`, `/plugins`, `/plugins/reload` |
| SSLSearchServer | 8008 | `/mcp` | `/ready` |

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
    @{Name="SSLSearchServer"; Port=8008; Endpoint="/mcp"}
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
