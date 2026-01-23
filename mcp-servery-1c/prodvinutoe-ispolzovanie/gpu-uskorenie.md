# GPU ускорение

Использование видеокарты NVIDIA для ускорения embedding.

## Требования

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
    -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
    -v "E:/bases/mcp_docs:/app/chroma_db" `
    comol/1c_help_mcp:latest
```

{% hint style="warning" %}
Не все образы MCP-серверов поддерживают GPU напрямую. Рекомендуется использовать LM Studio или Ollama для GPU embedding.
{% endhint %}

## Рекомендуемый подход: LM Studio

Вместо GPU в контейнерах, используйте LM Studio на хосте:

1. **LM Studio** использует GPU хоста
2. **MCP-серверы** обращаются к LM Studio через API
3. Нет необходимости в `--gpus all`

```
┌────────────────┐     ┌────────────────┐
│  LM Studio     │◀────│  MCP Server    │
│  (GPU хоста)   │     │  (контейнер)   │
└────────────────┘     └────────────────┘
```

### Конфигурация

```env
OPENAI_API_BASE=http://host.docker.internal:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL=Qwen3-Embedding-4B
```

## Ollama с GPU

Ollama также использует GPU хоста:

```powershell
# Установка Ollama (использует GPU автоматически)
ollama pull qwen3:embedding-4b

# MCP-сервер обращается к Ollama
docker run -d -p 8003:8003 `
    --name 1c_help_mcp `
    -e LICENSE_KEY=YOUR_LICENSE_KEY `
    -e OPENAI_API_BASE=http://host.docker.internal:11434/v1 `
    -e OPENAI_API_KEY=ollama `
    -e OPENAI_MODEL=qwen3:embedding-4b `
    -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
    -v "E:/bases/mcp_docs:/app/chroma_db" `
    comol/1c_help_mcp:latest
```

## Сравнение производительности

| Конфигурация | Время индексации (5000 док) |
|--------------|----------------------------|
| CPU (e5-small) | ~30 минут |
| CPU (e5-base) | ~60 минут |
| GPU (Qwen-4B через LM Studio) | ~5 минут |
| GPU (Qwen-8B через LM Studio) | ~10 минут |

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
