# Проблемы Docker

## WSL2 не установлен

### Симптом

```
Docker Desktop requires WSL 2
```

### Решение

```powershell
# Установка WSL2 (от администратора)
wsl --install
wsl --set-default-version 2

# Перезагрузите компьютер
```

## "WSL 2 installation is incomplete"

### Симптом

Docker Desktop не запускается, сообщение о неполной установке WSL2.

### Решение

1. Скачайте обновление ядра WSL2: https://aka.ms/wsl2kernel
2. Установите обновление
3. Перезапустите Docker Desktop

## Docker Desktop не запускается

### Проверка службы

```powershell
Get-Service docker
```

### Перезапуск службы

```powershell
Restart-Service docker
```

### Полный перезапуск

1. Закройте Docker Desktop
2. Завершите процессы Docker в Диспетчере задач
3. Запустите Docker Desktop заново

## Hyper-V конфликт

### Симптом

Docker не работает вместе с VirtualBox или VMware.

### Решение

Docker Desktop с WSL2 не требует Hyper-V. Убедитесь, что:

1. В Docker Desktop Settings → General → Use WSL 2 based engine ✅
2. Отключите Hyper-V если не нужен:

```powershell
# От администратора
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
```

## Нет места на диске

### Симптом

```
no space left on device
```

### Проверка

```powershell
docker system df
```

### Очистка

```powershell
# Удалить неиспользуемые данные
docker system prune -a

# Удалить все тома (осторожно!)
docker volume prune
```

### Изменение расположения данных Docker

1. Docker Desktop → Settings → Resources → Disk image location
2. Укажите путь на диске с большим объёмом
3. Apply & Restart

## Антивирус блокирует Docker

### Симптом

Контейнеры не запускаются, ошибки доступа к файлам.

### Решение

Добавьте исключения в антивирус:

- `C:\Program Files\Docker\`
- `C:\ProgramData\Docker\`
- `C:\Users\<username>\AppData\Local\Docker\`
- `C:\Users\<username>\.docker\`

## Контейнер не запускается

### Проверка логов

```powershell
docker logs <container_name>
```

### Проверка статуса

```powershell
docker ps -a --filter name=<container_name>
```

### Проверка ресурсов

```powershell
docker stats --no-stream
```

## Образ не скачивается

### Симптом

```
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled
```

### Решение

1. Проверьте интернет-соединение
2. Проверьте настройки прокси в Docker Desktop
3. Попробуйте перезапустить Docker

```powershell
# Ручное скачивание
docker pull comol/1c_help_mcp:latest
```

## Медленная работа Docker

### Увеличение ресурсов

1. Docker Desktop → Settings → Resources
2. Увеличьте:
   - Memory: 4-8 ГБ
   - CPUs: 2-4
   - Swap: 2 ГБ

### Использование SSD

Переместите Docker данные на SSD:

1. Docker Desktop → Settings → Resources → Disk image location
2. Укажите путь на SSD
