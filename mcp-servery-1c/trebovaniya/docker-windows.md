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

## Следующий шаг

После установки Docker Desktop настройте [Cursor IDE](cursor-nastrojka.md).
