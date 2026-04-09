# Сетевые требования

MCP-серверы работают как локальные HTTP-сервисы. Для их работы необходимо настроить сеть.

## Используемые порты

| Сервер                   | Порт | Протокол |
| ------------------------ | ---- | -------- |
| CodeMetadataSearchServer | 8000 | HTTP     |
| SyntaxCheckServer        | 8002 | HTTP     |
| HelpSearchServer         | 8003 | HTTP     |
| TemplatesSearchServer    | 8004 | HTTP     |
| Graph Metadata Search    | 8006 | HTTP     |
| 1CCodeChecker            | 8007 | HTTP     |
| SSLSearchServer          | 8008 | HTTP     |

{% hint style="info" %}
Все серверы работают только локально (localhost). Внешний доступ не требуется.
{% endhint %}

## Проверка доступности портов

### Проверка свободного порта

```powershell
# Проверить, занят ли порт 8003
netstat -an | findstr :8003
```

Если порт свободен — вывод будет пустым.

### Проверка всех MCP-портов

```powershell
# Проверить все порты MCP-серверов
@(8000, 8002, 8003, 8004, 8006, 8007, 8008) | ForEach-Object {
    $port = $_
    $result = netstat -an | findstr ":$port"
    if ($result) {
        Write-Host "Порт $port занят:" -ForegroundColor Red
        Write-Host $result
    } else {
        Write-Host "Порт $port свободен" -ForegroundColor Green
    }
}
```

### Найти процесс, занимающий порт

```powershell
# Найти PID процесса на порту 8003
netstat -ano | findstr :8003

# Найти имя процесса по PID
Get-Process -Id <PID>
```

## Windows Firewall

### Автоматическое разрешение

Docker Desktop обычно автоматически создаёт правила Firewall. Если возникают проблемы:

### Ручное создание правила

```powershell
# Запустите от имени администратора
# Разрешить входящие подключения на порты MCP
New-NetFirewallRule -DisplayName "MCP Servers" `
    -Direction Inbound `
    -LocalPort 8000,8002,8003,8004,8006,8007,8008 `
    -Protocol TCP `
    -Action Allow
```

### Проверка правил

```powershell
Get-NetFirewallRule -DisplayName "MCP*" | Format-Table Name, Enabled, Direction
```

## Антивирус

Некоторые антивирусы могут блокировать локальные HTTP-соединения.

### Добавление исключений

Добавьте в исключения антивируса:

* Процесс `docker.exe`
* Папку Docker: `C:\Program Files\Docker\`
* Папку WSL: `C:\Users\<username>\AppData\Local\Docker\`

## Внешний доступ к интернету

Для работы MCP-серверов требуется доступ в интернет:

### При первом запуске

| Ресурс           | Назначение                                                                                                                                      |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `hub.docker.com` | Скачивание Docker-образов                                                                                                                       |
| `huggingface.co` | Скачивание embedding моделей. <mark style="color:$danger;">**Внимание. Доступ к Huggiungface может быть заблокирован на территории РФ.**</mark> |

### В процессе работы

| Сервер        | Ресурс       | Назначение      |
| ------------- | ------------ | --------------- |
| 1CCodeChecker | `code.1c.ai` | API 1С:Напарник |

{% hint style="info" %}
После первого запуска большинство серверов могут работать офлайн.
{% endhint %}

## Прокси-сервер

Если в вашей сети используется прокси:

### Настройка Docker для прокси

1. Откройте Docker Desktop
2. **Settings** → **Resources** → **Proxies**
3. Включите **Manual proxy configuration**
4. Укажите адреса HTTP и HTTPS прокси
5. Добавьте в bypass: `localhost,127.0.0.1`

### Переменные окружения для контейнеров

```powershell
docker run -d `
  -e HTTP_PROXY=http://proxy.company.com:8080 `
  -e HTTPS_PROXY=http://proxy.company.com:8080 `
  -e NO_PROXY=localhost,127.0.0.1 `
  # ... остальные параметры
```

## Диагностика проблем

### Контейнер запущен, но не отвечает

```powershell
# Проверить, слушает ли контейнер порт
docker exec <container_name> netstat -tlnp

# Проверить логи
docker logs <container_name>
```

### Соединение отклонено

```powershell
# Проверить доступность из PowerShell
Test-NetConnection -ComputerName localhost -Port 8003
```

### Проверка через curl

```powershell
# Если установлен curl
curl http://localhost:8003/health
```
