# Установка Proxy-сервера

Подробная инструкция по установке Python Proxy-сервера OneAPA.

## Выбор способа установки

| Способ | Рекомендуется для | Сложность |
|--------|-------------------|-----------|
| [Docker](#установка-через-docker) | Production, быстрый старт | Низкая |
| [Python](#установка-из-исходников-python) | Разработка, отладка, кастомизация | Средняя |

## Установка через Docker

{% hint style="success" %}
Docker — рекомендуемый способ установки для production.
{% endhint %}

### Предварительные требования

- Docker 20.10 или выше
- 2 ГБ свободной памяти
- Файлы Proxy-сервера

### Шаг 1: Установка Docker

**Windows:**

1. Скачайте [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Запустите установщик
3. Перезагрузите компьютер
4. Запустите Docker Desktop

**Ubuntu/Debian:**

```bash
# Обновление пакетов
sudo apt-get update

# Установка зависимостей
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Добавление ключа Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Добавление репозитория
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
```

**CentOS/RHEL:**

```bash
# Установка yum-utils
sudo yum install -y yum-utils

# Добавление репозитория
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Установка Docker
sudo yum install -y docker-ce docker-ce-cli containerd.io

# Запуск службы
sudo systemctl start docker
sudo systemctl enable docker
```

### Шаг 2: Получение файлов

Скопируйте файлы Proxy-сервера на сервер:

```bash
# Структура каталога
OneAPAProxy/
├── main.py
├── api.py
├── agent_logic.py
├── data_store.py
├── graph_init.py
├── llm_utils.py
├── models.py
├── logging_config.py
├── cl_main.py
├── requirements.txt
├── Dockerfile
└── rebuild-container.ps1
```

### Шаг 3: Сборка образа

```bash
cd /path/to/OneAPAProxy

# Сборка Docker образа
docker build -t oneapa-proxy:latest .
```

### Шаг 4: Запуск контейнера

**Базовый запуск:**

```bash
docker run -d \
  --name oneapa-proxy \
  -p 9000:9000 \
  -e LICENSE_KEY=your_license_key \
  --restart unless-stopped \
  oneapa-proxy:latest
```

**С монтированием логов:**

```bash
docker run -d \
  --name oneapa-proxy \
  -p 9000:9000 \
  -e LICENSE_KEY=your_license_key \
  -v /var/log/oneapa:/app/log \
  --restart unless-stopped \
  oneapa-proxy:latest
```

### Шаг 5: Проверка

```bash
# Статус контейнера
docker ps

# Логи
docker logs oneapa-proxy

# Проверка API
curl http://localhost:9000/health
```

### Управление контейнером

```bash
# Остановка
docker stop oneapa-proxy

# Запуск
docker start oneapa-proxy

# Перезапуск
docker restart oneapa-proxy

# Удаление
docker rm -f oneapa-proxy

# Просмотр логов в реальном времени
docker logs -f oneapa-proxy
```

## Установка из исходников (Python)

### Предварительные требования

- Python 3.11, 3.12 или 3.13
- pip (менеджер пакетов)
- Git (опционально)

### Шаг 1: Установка Python

**Windows:**

1. Скачайте Python с [python.org](https://www.python.org/downloads/)
2. Запустите установщик
3. **Важно:** Отметьте "Add Python to PATH"
4. Завершите установку

**Ubuntu/Debian:**

```bash
sudo apt-get update
sudo apt-get install -y python3.11 python3.11-venv python3-pip
```

**CentOS/RHEL:**

```bash
sudo dnf install -y python3.11 python3.11-pip
```

### Шаг 2: Получение файлов

Скопируйте файлы Proxy-сервера:

```bash
cd /opt
mkdir oneapa-proxy
cd oneapa-proxy
# Скопируйте файлы в этот каталог
```

### Шаг 3: Создание виртуального окружения

```bash
# Создание
python3.11 -m venv venv

# Активация (Linux/Mac)
source venv/bin/activate

# Активация (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Активация (Windows CMD)
.\venv\Scripts\activate.bat
```

### Шаг 4: Установка зависимостей

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### Шаг 5: Настройка переменных окружения

**Linux:**

```bash
# Создайте файл .env или экспортируйте переменные
export LICENSE_KEY="your_license_key"
```

**Windows PowerShell:**

```powershell
$env:LICENSE_KEY = "your_license_key"
```

**Файл .env:**

```env
LICENSE_KEY=your_license_key
```

### Шаг 6: Запуск сервера

```bash
# Прямой запуск
python main.py

# Или через uvicorn
uvicorn main:app --host 0.0.0.0 --port 9000
```

### Шаг 7: Настройка автозапуска (Linux)

Создайте systemd service:

```bash
sudo nano /etc/systemd/system/oneapa-proxy.service
```

Содержимое:

```ini
[Unit]
Description=OneAPA Proxy Server
After=network.target

[Service]
Type=simple
User=oneapa
WorkingDirectory=/opt/oneapa-proxy
Environment="LICENSE_KEY=your_license_key"
ExecStart=/opt/oneapa-proxy/venv/bin/python main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Активация:

```bash
sudo systemctl daemon-reload
sudo systemctl enable oneapa-proxy
sudo systemctl start oneapa-proxy
```

### Шаг 8: Настройка автозапуска (Windows)

Используйте NSSM (Non-Sucking Service Manager):

```powershell
# Скачайте NSSM с https://nssm.cc/

# Установка службы
nssm install OneAPAProxy "C:\oneapa-proxy\venv\Scripts\python.exe" "C:\oneapa-proxy\main.py"
nssm set OneAPAProxy AppDirectory "C:\oneapa-proxy"
nssm set OneAPAProxy AppEnvironmentExtra "LICENSE_KEY=your_license_key"

# Запуск службы
nssm start OneAPAProxy
```

## Конфигурация

### Переменные окружения

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `PORT` | Порт сервера | 9000 |
| `HOST` | Адрес для прослушивания | 0.0.0.0 |
| `LOG_LEVEL` | Уровень логирования | INFO |

### Настройка порта

**Docker:**

```bash
docker run -d -p 8080:9000 oneapa-proxy:latest
```

**Python:**

```bash
uvicorn main:app --host 0.0.0.0 --port 8080
```

## Проверка установки

### API Health Check

```bash
curl http://localhost:9000/health
```

Ожидаемый ответ:

```json
{
  "message": "OK",
  "version": "1.0.1",
  "build_date": "2025-12-03"
}
```

### Web UI

Откройте в браузере: `http://localhost:9000/ui`

## Устранение проблем

### Порт уже используется

```bash
# Найти процесс
netstat -tlnp | grep 9000

# Освободить порт или использовать другой
docker run -p 9001:9000 ...
```

### Ошибки при установке зависимостей

```bash
# Обновите pip
pip install --upgrade pip

# Установите build tools (Ubuntu)
sudo apt-get install -y build-essential

# Windows: установите Visual Studio Build Tools
```

### Ошибка лицензии

Убедитесь, что переменная `LICENSE_KEY` установлена корректно.

### Контейнер не запускается

```bash
# Проверьте логи
docker logs oneapa-proxy

# Проверьте, что образ собран
docker images | grep oneapa
```

## Далее

{% content-ref url="nastrojka-docker.md" %}
[nastrojka-docker.md](nastrojka-docker.md)
{% endcontent-ref %}
