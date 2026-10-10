# Docker Desktop и WSL2 на Windows

На Windows рекомендуемый способ запуска MCP-серверов — Docker Desktop с WSL2. Эта страница описывает только настройку Windows; сами серверы поставляются в виде Linux-контейнеров и также работают на Linux, macOS и других платформах с Docker-совместимой средой.

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
  -e 1C_BIN_PATH=/1c_docs `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:latest
```

{% hint style="info" %}
Благодаря монтированию тома (`-v "E:/bases/mcp_docs:/app/index"`) ваш индекс сохранится при обновлении контейнера. Переиндексация не потребуется!
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
  -e 1C_BIN_PATH=/1c_docs `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/index" `
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

## Где держать выгрузки конфигураций

Каталог с диска Windows (`D:\...`), смонтированный в контейнер, Docker Desktop с WSL2 читает через сетевой протокол 9P. Для выгрузки 1С это узкое место: обход 25 тыс. файлов выгрузки занимает около 100 с вместо 5 с на самом диске, а когда выгрузку читают несколько контейнеров одновременно (например, три CodeMetadataSearch и три GraphMetadataSearch для трёх баз) — больше 5 минут; под такой нагрузкой отдельные чтения файлов срываются (CodeMetadataSearch с 06.10.2026 повторяет их). Из файловой системы WSL2 (ext4) или с тома Docker те же файлы читаются в 10–100 раз быстрее: чтение каталога `CommonModules` одной конфигурации — 1 с с тома против 12 с с диска `D:`.

Поэтому на Windows выгрузки для CodeMetadataSearch и GraphMetadataSearch лучше держать не на диске Windows:

### Вариант 1: файловая система WSL2

В дистрибутиве Linux, который поставил `wsl --install` (по умолчанию Ubuntu), создайте каталог для выгрузок и копируйте туда выгрузку из Windows — так делается и первая копия, и обновление после новой выгрузки:

```powershell
robocopy D:\exports\KA \\wsl.localhost\Ubuntu\home\<пользователь>\exports\KA /MIR
```

Контейнеры запускайте из терминала этого дистрибутива, а выгрузку монтируйте по пути Linux:

```bash
docker run -d --name 1c_code_metadata_mcp ... -v /home/<пользователь>/exports/KA:/app/code:ro comol/1c_code_metadata_mcp:light
```

В Docker Desktop должна быть включена интеграция с этим дистрибутивом (Settings → Resources → WSL integration).

### Вариант 2: именованный том Docker

```powershell
docker volume create ka_export
docker run --rm -v ka_export:/dst -v D:\exports\KA:/src:ro alpine sh -c "rm -rf /dst/* && cp -a /src/. /dst/"
```

Копирование идёт через 9P один раз и занимает минуты; серверу передавайте `-v ka_export:/app/code:ro`. Серверы читают том, а не диск: после новой выгрузки повторите копирование — до планового обновления GraphMetadataSearch (`GRAPH_REFRESH_INTERVAL_SEC`) или перезапуска CodeMetadataSearch.

### Индексируйте базы по очереди

Первая индексация CodeMetadataSearch и GraphMetadataSearch упирается в чтение выгрузки и CPU. Шесть контейнеров на одной машине делят их между собой, и каждый идёт дольше, чем шёл бы один; запускайте первую индексацию баз последовательно.

Индексы самих серверов (`/app/data`, `/app/index` и другие каталоги из [Кеширования БД](../prodvinutoe-ispolzovanie/keshirovanie-bd.md)) к этому не относятся: они и так лежат на томах Docker.

## Следующий шаг

После установки Docker Desktop настройте [Cursor IDE](cursor-nastrojka.md).
