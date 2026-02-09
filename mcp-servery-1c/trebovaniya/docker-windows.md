# Docker Desktop и WSL2 на Windows

Docker Desktop — единственный способ запуска MCP-серверов на Windows. Все серверы поставляются в виде готовых Docker-образов.

## Предварительные требования

- Windows 10 версии 2004 или новее (Build 19041+)
- Windows 11 (любая версия)
- Включённая виртуализация в BIOS (Intel VT-x / AMD-V)
- Права администратора

## Шаг 1: Установка WSL2

WSL2 (Windows Subsystem for Linux 2) требуется для Docker Desktop.

### Автоматическая установка (рекомендуется)

Откройте PowerShell **от имени администратора** и выполните:

```powershell
wsl --install
```

Перезагрузите компьютер после завершения.

### Проверка установки

```powershell
wsl --version
```

Должна отобразиться версия WSL 2.x.x.

### Установка версии WSL2 по умолчанию

```powershell
wsl --set-default-version 2
```

## Шаг 2: Установка Docker Desktop

### Скачивание

1. Перейдите на [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Скачайте **Docker Desktop for Windows**
3. Запустите установщик

### Установка

1. В процессе установки убедитесь, что отмечена опция **Use WSL 2 instead of Hyper-V**
2. Завершите установку
3. Перезагрузите компьютер

### Первый запуск

1. Запустите Docker Desktop
2. Примите условия лицензии
3. Дождитесь запуска (статус "Docker Desktop is running")

## Шаг 3: Проверка работоспособности

### Проверка версии Docker

```powershell
docker --version
```

Ожидаемый вывод: `Docker version 24.x.x` или новее.

### Проверка работы контейнеров

```powershell
docker run hello-world
```

Должно появиться сообщение "Hello from Docker!".

### Проверка доступа к Docker Hub

```powershell
docker pull alpine:latest
```

Образ должен успешно скачаться.

## Настройка ресурсов

Docker Desktop по умолчанию использует ограниченные ресурсы. Для MCP-серверов рекомендуется увеличить лимиты.

### Изменение настроек

1. Откройте Docker Desktop
2. Перейдите в **Settings** → **Resources** → **Advanced**
3. Установите:
   - **Memory**: минимум 4 ГБ, рекомендуется 8 ГБ
   - **CPUs**: минимум 2, рекомендуется 4
   - **Disk image size**: минимум 50 ГБ
4. Нажмите **Apply & Restart**

## Типичные проблемы

### "WSL 2 installation is incomplete"

```powershell
# Скачайте и установите обновление ядра WSL2
# https://aka.ms/wsl2kernel
```

### Docker Desktop не запускается

1. Убедитесь, что виртуализация включена в BIOS
2. Проверьте, что Hyper-V не конфликтует с другими гипервизорами (VirtualBox, VMware)
3. Перезапустите службу Docker:

```powershell
Restart-Service docker
```

### Недостаточно места на диске

```powershell
# Очистка неиспользуемых данных Docker
docker system prune -a

# Проверка использования диска
docker system df
```

### Антивирус блокирует Docker

Добавьте исключения в антивирус:
- `C:\Program Files\Docker\`
- `C:\ProgramData\Docker\`
- `C:\Users\<username>\AppData\Local\Docker\`

## Полезные команды

```powershell
# Список запущенных контейнеров
docker ps

# Список всех контейнеров
docker ps -a

# Остановить контейнер
docker stop <container_name>

# Удалить контейнер
docker rm <container_name>

# Просмотр логов контейнера
docker logs <container_name>

# Просмотр логов в реальном времени
docker logs -f <container_name>

# Перезапуск контейнера
docker restart <container_name>
```

## Обновление образов MCP-серверов

{% hint style="warning" %}
**Рекомендуется сохранять текущий образ перед обновлением!** Если новая версия окажется нерабочей, вы сможете быстро откатиться на предыдущую версию без повторного скачивания.
{% endhint %}

### Полная процедура обновления MCP-сервера

```powershell
# 1. Сохранить текущий образ под резервным тегом
docker tag comol/1c_help_mcp:latest comol/1c_help_mcp:previous

# 2. Скачать новую версию образа
docker pull comol/1c_help_mcp:latest

# 3. Остановить и удалить старый контейнер
docker stop 1c_help_mcp
docker rm 1c_help_mcp

# 4. Запустить контейнер с новым образом (индекс сохранится в томе!)
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

{% hint style="info" %}
Благодаря монтированию тома (`-v "E:/bases/mcp_docs:/app/chroma_db"`) ваш индекс сохранится при обновлении контейнера. Переиндексация не потребуется!
{% endhint %}

### Откат на предыдущую версию

Если после обновления сервер работает некорректно, откатитесь на сохранённый образ:

```powershell
# 1. Остановить и удалить проблемный контейнер
docker stop 1c_help_mcp
docker rm 1c_help_mcp

# 2. Запустить контейнер из предыдущего образа
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:previous
```

{% hint style="info" %}
После того как вы убедились, что новая версия работает стабильно, можно удалить резервный образ для освобождения места на диске:

```powershell
docker rmi comol/1c_help_mcp:previous
```
{% endhint %}

### Просмотр информации об образах

```powershell
# Список локальных образов (включая резервные)
docker images | Select-String "comol"

# Подробная информация об образе
docker inspect comol/1c_help_mcp:latest
```

## Флаг --rm в командах Docker

В некоторых примерах вы можете встретить флаг `--rm`:

```powershell
docker run --rm -d -p 8003:8003 ...
```

### Что делает флаг --rm

Флаг `--rm` автоматически **удаляет контейнер** после его остановки.

{% hint style="warning" %}
**Не рекомендуется для MCP-серверов!** При использовании `--rm`:
- Контейнер удаляется при остановке (даже случайной)
- Если вы не примонтировали том — **все данные индекса будут потеряны**
- Усложняется диагностика проблем (нет доступа к логам после остановки)
{% endhint %}

### Рекомендация

Убирайте флаг `--rm` из команд запуска MCP-серверов:

```powershell
# Не рекомендуется (с --rm)
docker run --rm -d -p 8003:8003 --name 1c_help_mcp ...

# Рекомендуется (без --rm)
docker run -d -p 8003:8003 --name 1c_help_mcp ...
```

Для управления контейнерами используйте:

```powershell
# Остановить контейнер (данные сохраняются)
docker stop 1c_help_mcp

# Запустить существующий контейнер
docker start 1c_help_mcp

# Удалить контейнер (когда действительно нужно)
docker rm 1c_help_mcp
```

## Следующий шаг

После установки Docker Desktop настройте [Cursor IDE](cursor-nastrojka.md).
