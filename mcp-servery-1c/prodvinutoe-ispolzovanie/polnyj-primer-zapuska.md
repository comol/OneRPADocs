# Полный пример запуска

Скрипт для запуска всех MCP-серверов с LM Studio.

## Предварительные требования

1. Docker Desktop запущен
2. LM Studio запущен с моделью Qwen3-Embedding-4B
3. Созданы папки для данных

## Создание папок

```powershell
New-Item -ItemType Directory -Force -Path @(
    "E:\bases\mcp_docs",
    "E:\bases\mcp_codemetadata",
    "E:\bases\mcp_ssl",
    "E:\bases\mcp_templates",
    "E:\bases\mcp_model_cache"
)
```

## Скрипт запуска (start_mcp_servers.ps1)

```powershell
# ============================================
# Скрипт запуска MCP серверов для 1С
# ============================================

# Конфигурация
$LICENSE_KEY = "YOUR_LICENSE_KEY"
$LM_STUDIO_URL = "http://host.docker.internal:1234/v1"
$EMBEDDING_MODEL = "Qwen3-Embedding-4B"
$1C_VERSION = "8.3.23.1997"

Write-Host "Запуск MCP серверов для 1С..." -ForegroundColor Cyan

# --------------------------------------------
# 1. SyntaxCheckServer (порт 8002) - без embedding
# --------------------------------------------
Write-Host "Запуск SyntaxCheckServer..." -ForegroundColor Yellow
docker run -d -p 8002:8002 `
    --name 1c_syntaxcheck_mcp `
    -e LICENSE_KEY=$LICENSE_KEY `
    comol/1c_syntaxcheck_mcp:latest

# --------------------------------------------
# 2. HelpSearchServer (порт 8003)
# --------------------------------------------
Write-Host "Запуск HelpSearchServer..." -ForegroundColor Yellow
docker run -d -p 8003:8003 `
    --name 1c_help_mcp `
    -e LICENSE_KEY=$LICENSE_KEY `
    -e 1C_BIN_PATH=/1c_docs `
    -e RESET_DATABASE=false `
    -e EMBEDDING_API_BASE=$LM_STUDIO_URL `
    -e EMBEDDING_API_KEY=lm-studio `
    -e EMBEDDING_MODEL=$EMBEDDING_MODEL `
    -v "C:/Program Files/1cv8/$1C_VERSION/bin:/1c_docs" `
    -v "E:/bases/mcp_docs:/app/index" `
    -v "E:/bases/mcp_model_cache:/app/model_cache" `
    comol/1c_help_mcp:latest

# --------------------------------------------
# 3. SSLSearchServer (порт 8008)
# --------------------------------------------
Write-Host "Запуск SSLSearchServer..." -ForegroundColor Yellow
docker run -d -p 8008:8008 `
    --name mcp_ssl_server `
    -e LICENSE_KEY=$LICENSE_KEY `
    -e SSL_VERSION=3.1.11 `
    -e RESET_DATABASE=false `
    -e EMBEDDING_API_BASE=$LM_STUDIO_URL `
    -e EMBEDDING_API_KEY=lm-studio `
    -e EMBEDDING_MODEL=$EMBEDDING_MODEL `
    -v "E:/bases/mcp_ssl:/app/zvec_db" `
    comol/mcp_ssl_server:latest

# --------------------------------------------
# 4. TemplatesSearchServer (порт 8004)
# --------------------------------------------
Write-Host "Запуск TemplatesSearchServer..." -ForegroundColor Yellow
docker run -d -p 8004:8004 `
    --name template_search_mcp `
    -e LICENSE_KEY=$LICENSE_KEY `
    -e RESET_DATABASE=false `
    -e EMBEDDING_API_BASE=$LM_STUDIO_URL `
    -e EMBEDDING_API_KEY=lm-studio `
    -e EMBEDDING_MODEL=$EMBEDDING_MODEL `
    -v "E:/bases/mcp_templates:/app/data" `
    comol/template-search-mcp:latest

# --------------------------------------------
# Итог
# --------------------------------------------
Write-Host ""
Write-Host "Все серверы запущены!" -ForegroundColor Green
Write-Host ""
Write-Host "Проверка статуса:" -ForegroundColor Cyan
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

## Скрипт остановки (stop_mcp_servers.ps1)

```powershell
# Остановка всех MCP серверов
Write-Host "Остановка MCP серверов..." -ForegroundColor Yellow

$containers = @(
    "1c_help_mcp",
    "1c_syntaxcheck_mcp",
    "mcp_ssl_server",
    "template_search_mcp"
)

foreach ($container in $containers) {
    docker stop $container 2>$null
    docker rm $container 2>$null
}

Write-Host "Все серверы остановлены" -ForegroundColor Green
```

## Скрипт проверки (check_mcp_servers.ps1)

```powershell
# Проверка состояния серверов
Write-Host "Состояние MCP серверов:" -ForegroundColor Cyan
Write-Host ""

$servers = @(
    @{Name="SyntaxCheckServer"; Port=8002},
    @{Name="HelpSearchServer"; Port=8003},
    @{Name="TemplatesSearchServer"; Port=8004},
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

## Использование

### Запуск

```powershell
.\start_mcp_servers.ps1
```

### Остановка

```powershell
.\stop_mcp_servers.ps1
```

### Проверка

```powershell
.\check_mcp_servers.ps1
```

## Настройка автозапуска

### Через Планировщик заданий

1. Откройте Планировщик заданий
2. Создать задачу
3. Триггер: При входе в систему
4. Действие: Запустить PowerShell с параметром `-File "E:\scripts\start_mcp_servers.ps1"`

{% hint style="warning" %}
LM Studio должен быть запущен до запуска скрипта!
{% endhint %}
