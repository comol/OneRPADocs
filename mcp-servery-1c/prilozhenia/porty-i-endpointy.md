# Порты и эндпоинты

Сводная таблица портов всех MCP-серверов.

## Порты серверов

По умолчанию современные образы используют `streamable-http` на `/mcp`. Если клиент требует SSE, включите `USESSE=true`; у Graph Metadata Search эта переменная называется `MCP_USE_SSE`. У большинства серверов URL остаётся `/mcp`; SyntaxCheckServer использует стандартные SSE-пути `/sse` и `/messages/`.

Graph Metadata Search по умолчанию обслуживает `streamable-http` без серверной сессии и отвечает на POST обычным JSON (`FASTMCP_STATELESS_HTTP=true`, `FASTMCP_JSON_RESPONSE=true`). Для клиента, которому нужна stateful-сессия с SSE-ответами на том же `/mcp`, задайте обе переменные `false`. `/mcp` и `/mcp/` Graph Metadata Search обслуживает напрямую, без перенаправления (образы от 13.10.2026; раньше в режиме по умолчанию `POST /mcp` получал `307 Temporary Redirect` на `/mcp/`, и клиент, который не следует перенаправлениям, не подключался).

| Сервер | Порт | MCP Endpoint | Дополнительные |
|--------|------|--------------|----------------|
| CodeMetadataSearchServer | 8000 | `/mcp` | `/live`, `/ready`, `/health` (устаревший алиас `/live`) |
| SyntaxCheckServer | 8002 | `/mcp` (`streamable-http`) или `/sse` (SSE) | `/messages/` (SSE) |
| HelpSearchServer | 8003 | `/mcp` | `/health`, `/ready`, `/plugins`, `/plugins/reload` |
| TemplatesSearchServer | 8004 | `/mcp` | `/health`, `/ready`, `/extend/`, `/extend/memory` (веб-интерфейс) |
| Graph Metadata Search | 8006 | `/mcp` | `/health` (`/healthz`), `/ready` (`/readyz`) |
| 1CCodeChecker | 8007 | `/mcp` | `/health`, `/ready`, `/metrics/sessions`, `/release`, `/plugins`, `/plugins/reload` |
| MCP QA | 8020 | `/mcp` (stateful Streamable HTTP) | `/healthz` (liveness), `/`; `POST /client-bus/v1/poll`, `POST /client-bus/v1/result` — сетевой канал расширения `MCPQAClient` с токеном сеанса в `X-MCP-QA-Token`; исходящее подключение к тест-клиенту 1С, по умолчанию `host.docker.internal:1538` |
| Конвертация данных 2.0 (KD20) | 8009 | `/mcp` (Streamable HTTP) | `/healthz` (liveness) |
| SSLSearchServer | 8008 | `/mcp` | `/ready` |

## Сессии MCP и ответ 404 на долгой работе

Большинство серверов работает со stateful-сессиями Streamable HTTP: в ответ на `initialize` сервер выдаёт заголовок `Mcp-Session-Id`, и клиент передаёт его в каждом следующем запросе. Сервер закрывает сессию в трёх случаях:

- **простой** — дольше заданного времени не было ни одного запроса с этим `Mcp-Session-Id`. Таймер сбрасывает любой такой запрос: вызов инструмента, `tools/list`, `ping`;
- **максимальное время жизни** — от `initialize` прошло больше заданного, даже если сессией пользуются постоянно. Запросы этот срок не продлевают;
- **перезапуск контейнера** — сессии хранятся в памяти и после рестарта или обновления образа не восстанавливаются.

| Сервер | Простой | Время жизни | Переменные (простой / время жизни) |
|--------|---------|-------------|-------------------------------------|
| 1CCodeChecker (8007) | 900 с (15 мин) | 28800 с (8 ч) | `MCP_TRANSPORT_SESSION_IDLE_TIMEOUT` / `MCP_TRANSPORT_SESSION_MAX_LIFETIME` |
| SSLSearchServer (8008) | 900 с (15 мин) | 14400 с (4 ч) | `SESSION_IDLE_TTL` / `SESSION_MAX_LIFETIME` |
| HelpSearchServer (8003) | 1800 с (30 мин) | 86400 с (24 ч) | `MCP_SESSION_IDLE_TTL_SECONDS` / `MCP_SESSION_MAX_LIFETIME_SECONDS` |
| SyntaxCheckServer (8002) | 1800 с (30 мин) | 28800 с (8 ч) | `MCP_SESSION_IDLE_TTL_SECONDS` / `MCP_SESSION_MAX_LIFETIME_SECONDS` |
| TemplatesSearchServer (8004) | 1800 с (30 мин) | 28800 с (8 ч) | `MCP_SESSION_IDLE_TIMEOUT` / `MCP_SESSION_MAX_LIFETIME` |
| MCP QA (8020), KD20 (8009) | 1800 с (30 мин) | не ограничено | не настраиваются: это значение по умолчанию библиотеки MCP; открытый поток `GET /mcp` тоже удерживает сессию |
| CodeMetadataSearchServer (8000), Graph Metadata Search (8006) | — | — | по умолчанию сессий нет (`FASTMCP_STATELESS_HTTP=true`), истекать нечему |

Значения задаются в секундах, целым положительным числом, через `-e` при запуске контейнера. Значения «без ограничения» нет — для долгих прогонов задайте большое число, например `-e MCP_TRANSPORT_SESSION_IDLE_TIMEOUT=28800`. Ноль и отрицательное число 1CCodeChecker и HelpSearchServer не принимают: контейнер не стартует и называет переменную. Остальные серверы на неверное значение берут значение по умолчанию.

Запрос с идентификатором закрытой сессии получает HTTP `404`. Тело ответа у серверов двух видов, и оба означают одно и то же — «сессия закрыта, начните новую»:

- 1CCodeChecker: `{"error": "session_expired", "reason": "expired_idle", "detail": "..."}` (`reason` — `expired_idle` или `expired_lifetime`) и заголовок `x-mcp-session-state: expired`. Если сервер сессию не знает (например, после перезапуска), — `"error": "session_unknown"`;
- остальные: `{"jsonrpc":"2.0","id":"server-error","error":{"code":-32600,"message":"Session not found"}}`. У SyntaxCheckServer до выпуска 13.10.2026 в этом ответе было ещё поле `data` с `"problem": "unknown_path"` и текстом «Nothing is bound at '/mcp'» — адрес при этом верный.

По спецификации MCP клиент, получив `404` на запрос с `Mcp-Session-Id`, обязан начать новую сессию: отправить `initialize` без идентификатора. Если клиент этого не делает и после простоя сервер «пропадает» до ручного переподключения:

1. Переподключите сервер в клиенте (выключить и включить в списке MCP-серверов) — это и есть новая сессия, перезапускать контейнер не нужно.
2. Увеличьте простой переменной из таблицы, например до 8 ч для IDE-плагина или долгого агентного прогона.
3. Держите сессию активной: любой запрос, в том числе `ping`, сбрасывает таймер простоя. Время жизни так не продлить — после него переподключение всё равно понадобится.

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
    "1c-kd20": {
      "url": "http://127.0.0.1:8009/mcp"
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
    @{Name="KD20"; Port=8009; Endpoint="/mcp"}
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
