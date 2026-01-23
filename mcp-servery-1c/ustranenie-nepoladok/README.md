# Устранение неполадок

Решение типичных проблем при работе с MCP-серверами.

## Содержание

- [Проблемы Docker](docker-problemy.md) — Docker не запускается, WSL2
- [Проблемы индексации](indeksaciya.md) — долгая индексация, ошибки
- [Проблемы подключения](podklyuchenie.md) — Cursor не видит MCP

## Быстрая диагностика

### 1. Проверка Docker

```powershell
docker --version
docker ps
```

### 2. Проверка контейнеров

```powershell
docker ps -a --format "table {{.Names}}\t{{.Status}}"
```

### 3. Проверка портов

```powershell
@(8000, 8002, 8003, 8004, 8006, 8007, 8008, 8011) | ForEach-Object {
    $result = Test-NetConnection -ComputerName localhost -Port $_ -WarningAction SilentlyContinue
    Write-Host "Порт $_`: $($result.TcpTestSucceeded)"
}
```

### 4. Просмотр логов

```powershell
docker logs <container_name>
```

## Типичные ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| "port already in use" | Порт занят | Освободить порт или использовать другой |
| "path not found" | Неверный путь тома | Проверить путь, использовать прямые слеши |
| "connection refused" | Контейнер не запущен | Запустить контейнер, проверить логи |
| "license key invalid" | Неверный ключ | Проверить LICENSE_KEY |
| "embedding API error" | LM Studio не запущен | Запустить LM Studio |

## Сбор информации для поддержки

```powershell
# Версии
Write-Host "Docker: $(docker --version)"
Write-Host "Windows: $([Environment]::OSVersion.Version)"

# Статус контейнеров
docker ps -a

# Логи проблемного контейнера
docker logs --tail 100 <container_name>
```
