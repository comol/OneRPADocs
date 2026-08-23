# GPU ускорение

Использование видеокарты NVIDIA для ускорения embedding.

{% hint style="success" %}
**Рекомендация:** Для GPU-ускорения используйте **LM Studio**. Это самый простой и надёжный способ — не требуется настройка CUDA в Docker, просто запустите LM Studio на хосте, и MCP-серверы будут использовать GPU через API.
{% endhint %}

## Рекомендуемый подход: LM Studio

Вместо сложной настройки GPU в Docker-контейнерах, используйте LM Studio на хосте:

1. **LM Studio** запускается на Windows и автоматически использует GPU
2. **MCP-серверы** в Docker обращаются к LM Studio через HTTP API
3. Нет необходимости в `--gpus all` или NVIDIA Container Toolkit

```
┌────────────────┐     ┌────────────────┐
│  LM Studio     │◀────│  MCP Server    │
│  (GPU хоста)   │     │  (контейнер)   │
└────────────────┘     └────────────────┘
```

Подробнее о настройке: [LM Studio](../embedding-modeli/lm-studio.md)

## Альтернатива: GPU напрямую в Docker

Если вам нужен GPU напрямую в контейнере (не рекомендуется для большинства случаев):

### Требования

- **Windows 11** (Windows 10 имеет ограниченную поддержку)
- **NVIDIA GPU** с драйвером версии 470+
- **Docker Desktop** с поддержкой WSL2 GPU
- **NVIDIA Container Toolkit**

## Проверка поддержки GPU

### Проверка драйвера

```powershell
nvidia-smi
```

Должна отобразиться информация о GPU.

### Проверка Docker GPU

```powershell
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

Если команда успешна — GPU доступен в Docker.

## Настройка Docker Desktop

1. Откройте Docker Desktop
2. Settings → Resources → WSL Integration
3. Включите интеграцию с вашим WSL дистрибутивом
4. Apply & Restart

## Использование GPU в контейнерах

### Параметр --gpus

```powershell
docker run -d -p 8003:8003 `
    --gpus all `
    --name 1c_help_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e 1C_BIN_PATH=/1c_docs `
    -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
    -v "E:/bases/mcp_docs:/app/index" `
    comol/1c_help_mcp:latest
```

{% hint style="warning" %}
Не все образы MCP-серверов поддерживают GPU напрямую. Рекомендуется использовать LM Studio или Ollama для GPU embedding.
{% endhint %}

## Конфигурация LM Studio для GPU

Если LM Studio запущен локально на порту по умолчанию (1234), параметры подключения можно не указывать:

```env
# Минимальная конфигурация (LM Studio на localhost:1234)
EMBEDDING_MODEL=Qwen3-Embedding-4B
```

Полная конфигурация, если нужно указать явно:

```env
EMBEDDING_API_BASE=http://host.docker.internal:1234/v1
EMBEDDING_API_KEY=lm-studio
EMBEDDING_MODEL=Qwen3-Embedding-4B
```

## Сравнение производительности

| Конфигурация | Время индексации (5000 док) |
|--------------|----------------------------|
| CPU (e5-small) | ~10-20 часов |
| CPU (e5-base) | ~20-40 часов |
| GPU (Qwen-4B через LM Studio) | ~1-2 часа |
| GPU (Qwen-8B через LM Studio) | ~2-4 часа |

{% hint style="info" %}
Время индексации сильно зависит от объёма данных, производительности CPU/GPU и выбранной модели. Приведённые значения — ориентировочные для типичной конфигурации 1С.
{% endhint %}

## Устранение проблем

### GPU не виден в Docker

```powershell
# Обновите драйвер NVIDIA
# Перезапустите Docker Desktop
# Проверьте WSL интеграцию
wsl --status
```

### Ошибка CUDA

```
CUDA error: out of memory
```

Решение:
- Закройте другие приложения, использующие GPU
- Используйте модель меньшего размера
- Уменьшите batch size

### LM Studio не использует GPU

1. Проверьте настройки LM Studio
2. Убедитесь, что выбран GPU в настройках
3. Перезапустите LM Studio

## Мониторинг GPU

### nvidia-smi

```powershell
# Однократно
nvidia-smi

# Мониторинг каждые 2 секунды
nvidia-smi -l 2
```

### В LM Studio

LM Studio показывает использование GPU в интерфейсе при загрузке модели и обработке запросов.
