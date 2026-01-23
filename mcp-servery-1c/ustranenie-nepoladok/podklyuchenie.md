# Проблемы подключения

## Cursor не видит MCP-сервер

### Проверка 1: Контейнер запущен

```powershell
docker ps --filter name=<container_name>
```

### Проверка 2: Порт доступен

```powershell
Test-NetConnection -ComputerName localhost -Port 8002
```

### Проверка 3: Файл mcp.json

```powershell
Get-Content "$env:APPDATA\Cursor\User\globalStorage\mcp.json"
```

### Проверка 4: JSON валиден

```powershell
$content = Get-Content "$env:APPDATA\Cursor\User\globalStorage\mcp.json" -Raw
try {
    $json = $content | ConvertFrom-Json
    Write-Host "JSON валиден" -ForegroundColor Green
} catch {
    Write-Host "Ошибка JSON: $_" -ForegroundColor Red
}
```

### Решение

1. Перезапустите Cursor (закройте все окна)
2. Проверьте правильность URL в mcp.json
3. Убедитесь, что контейнер запущен и порт доступен

## Ошибка "Connection refused"

### Симптом

```
ECONNREFUSED 127.0.0.1:8002
```

### Причины

1. Контейнер не запущен
2. Контейнер всё ещё инициализируется
3. Порт неправильный

### Решение

```powershell
# Проверить статус
docker ps -a --filter name=<container_name>

# Проверить логи
docker logs <container_name>

# Перезапустить
docker restart <container_name>
```

## Порт уже занят

### Симптом

```
Error: port is already allocated
```

### Найти процесс

```powershell
netstat -ano | findstr :8002
# Найти PID и завершить процесс
taskkill /PID <PID> /F
```

### Использовать другой порт

```powershell
docker run -d -p 8102:8002 `
    --name 1c_syntaxcheck_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    comol/1c_syntaxcheck_mcp:latest
```

Обновите mcp.json:
```json
{
  "mcpServers": {
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8102/mcp",
      "connection_id": "1c_lsp_service_001"
    }
  }
}
```

## Windows Firewall блокирует

### Проверка

```powershell
Get-NetFirewallRule -DisplayName "*Docker*" | Format-Table Name, Enabled
```

### Добавление правила

```powershell
# От администратора
New-NetFirewallRule -DisplayName "MCP Servers" `
    -Direction Inbound `
    -LocalPort 8000,8002,8003,8004,8006,8007,8008,8011 `
    -Protocol TCP `
    -Action Allow
```

## MCP работает, но ИИ не использует инструменты

### Причина 1: Нет Cursor Rules

Добавьте правила, указывающие ИИ использовать MCP:

```markdown
При работе с кодом 1С используй MCP-серверы:
- 1c-syntax-checker-mcp для проверки синтаксиса
- 1c-docs-mcp для поиска по справке
```

### Причина 2: Неявный запрос

Попробуйте явно указать:
- "Проверь синтаксис этого кода через MCP"
- "Найди в справке метод Запрос.Выполнить"

### Причина 3: Сервер не подключён

Проверьте список подключённых серверов в Cursor.

## Timeout при запросах

### Симптом

Запросы к MCP-серверу зависают или возвращают timeout.

### Причины

1. Сервер перегружен (идёт индексация)
2. Недостаточно ресурсов
3. Проблемы с embedding моделью

### Решение

```powershell
# Проверить состояние
docker stats <container_name>

# Проверить логи
docker logs --tail 50 <container_name>
```

## Скрипт полной диагностики

```powershell
Write-Host "=== Диагностика MCP серверов ===" -ForegroundColor Cyan

# Docker
Write-Host "`nDocker:" -ForegroundColor Yellow
docker --version

# Контейнеры
Write-Host "`nКонтейнеры:" -ForegroundColor Yellow
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Порты
Write-Host "`nПорты:" -ForegroundColor Yellow
@(8000, 8002, 8003, 8004, 8006, 8007, 8008, 8011) | ForEach-Object {
    $result = Test-NetConnection -ComputerName localhost -Port $_ -WarningAction SilentlyContinue
    if ($result.TcpTestSucceeded) {
        Write-Host "  ✓ Порт $_ доступен" -ForegroundColor Green
    } else {
        Write-Host "  ✗ Порт $_ недоступен" -ForegroundColor Red
    }
}

# mcp.json
Write-Host "`nmcp.json:" -ForegroundColor Yellow
$mcpPath = "$env:APPDATA\Cursor\User\globalStorage\mcp.json"
if (Test-Path $mcpPath) {
    Write-Host "  Файл существует" -ForegroundColor Green
    try {
        Get-Content $mcpPath -Raw | ConvertFrom-Json | Out-Null
        Write-Host "  JSON валиден" -ForegroundColor Green
    } catch {
        Write-Host "  JSON невалиден!" -ForegroundColor Red
    }
} else {
    Write-Host "  Файл не найден!" -ForegroundColor Red
}
```
