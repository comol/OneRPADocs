# LM Studio (рекомендуется)

LM Studio — это приложение для локального запуска LLM и embedding моделей с OpenAI-совместимым API.

## Почему LM Studio?

- **Высокое качество** — модели Qwen3-Embedding дают отличные результаты для русского языка
- **GPU ускорение** — использует видеокарту для быстрой работы
- **OpenAI API** — совместим с MCP-серверами без изменений
- **Бесплатно** — полностью бесплатное приложение
- **Локально** — все данные остаются на вашем компьютере

## Требования

- **Windows 10/11** (64-bit)
- **NVIDIA GPU** с 4+ ГБ VRAM (для Qwen3-Embedding-4B)
- **8+ ГБ VRAM** (для Qwen3-Embedding-8B)
- **16+ ГБ RAM**

## Установка LM Studio

### Шаг 1: Скачивание

1. Перейдите на [lmstudio.ai](https://lmstudio.ai/)
2. Скачайте версию для Windows
3. Установите приложение

### Шаг 2: Первый запуск

1. Запустите LM Studio
2. Приложение проверит вашу систему
3. Дождитесь завершения инициализации

## Загрузка embedding модели

### Шаг 1: Поиск модели

1. Откройте вкладку **Search** (или нажмите Ctrl+F)
2. В поиске введите: `Qwen3-Embedding`
3. Найдите модель `Qwen/Qwen3-Embedding-4B-GGUF`

### Шаг 2: Выбор квантизации

| Вариант | Размер | VRAM | Качество |
|---------|--------|------|----------|
| Q8_0 | ~4.5 ГБ | ~5 ГБ | Лучшее |
| Q6_K | ~3.5 ГБ | ~4 ГБ | Отличное |
| Q4_K_M | ~2.5 ГБ | ~3 ГБ | Хорошее |

{% hint style="info" %}
Рекомендуется **Q8_0** или **Q6_K** для лучшего качества embedding.
{% endhint %}

### Шаг 3: Загрузка

1. Нажмите **Download** рядом с выбранной квантизацией
2. Дождитесь завершения загрузки (несколько минут)

## Запуск локального сервера

### Шаг 1: Загрузка модели

1. Перейдите на вкладку **Local Server** (иконка сервера слева)
2. В выпадающем списке выберите загруженную модель Qwen3-Embedding
3. Нажмите **Load Model**

### Шаг 2: Запуск сервера

1. Убедитесь, что модель загружена (статус "Model loaded")
2. Нажмите **Start Server**
3. Сервер запустится на порту **1234**

### Шаг 3: Проверка

Сервер должен показывать:
- Status: **Running**
- Port: **1234**
- URL: `http://localhost:1234`

## Настройка MCP-серверов

### Переменные окружения

Для подключения MCP-серверов к LM Studio используйте:

```env
EMBEDDING_API_BASE=http://host.docker.internal:1234/v1
EMBEDDING_API_KEY=lm-studio
EMBEDDING_MODEL=Qwen3-Embedding-4B
```

{% hint style="success" %}
**Упрощённая настройка:** Если LM Studio запущен локально на порту 1234 (по умолчанию), параметры `EMBEDDING_API_BASE` и `EMBEDDING_API_KEY` можно не указывать — MCP-серверы подключатся к LM Studio автоматически.
{% endhint %}

{% hint style="info" %}
`host.docker.internal` — специальный адрес для доступа из Docker-контейнера к хост-машине Windows.
{% endhint %}

### Пример команды Docker (упрощённый)

Если LM Studio запущен локально, достаточно указать только модель:

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -e 1C_BIN_PATH=/1c_docs `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:latest
```

{% hint style="info" %}
Путь `E:/bases/mcp_docs` — это **пример**. Используйте любой удобный путь для хранения индексов.
{% endhint %}

### Пример команды Docker (полный)

Если нужно явно указать параметры подключения к LM Studio:

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e EMBEDDING_API_BASE=http://host.docker.internal:1234/v1 `
  -e EMBEDDING_API_KEY=lm-studio `
  -e EMBEDDING_MODEL=Qwen3-Embedding-4B `
  -e 1C_BIN_PATH=/1c_docs `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/index" `
  comol/1c_help_mcp:latest
```

## Проверка работы

### Тест API из PowerShell

```powershell
# Тест embedding endpoint
$body = @{
    input = "Тестовый текст для embedding"
    model = "Qwen3-Embedding-4B"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:1234/v1/embeddings" `
    -Method Post `
    -Headers @{"Content-Type"="application/json"} `
    -Body $body
```

### Проверка из Docker

```powershell
docker run --rm curlimages/curl:latest `
    curl -s http://host.docker.internal:1234/v1/models
```

## Оптимизация производительности

### Настройки LM Studio

1. **GPU Layers**: Установите максимум (все слои на GPU)
2. **Context Length**: 512 достаточно для embedding
3. **Batch Size**: Увеличьте до 32-64 для быстрой индексации

### Рекомендации

- Закройте другие приложения, использующие GPU
- Не запускайте LLM и embedding одновременно (если мало VRAM)
- Используйте SSD для кэша моделей

## Автозапуск

### Создание задачи в планировщике

1. Откройте **Планировщик заданий** (Task Scheduler)
2. Создайте новую задачу
3. Триггер: **При входе в систему**
4. Действие: Запустить `LM Studio.exe`

{% hint style="warning" %}
LM Studio должен быть запущен до запуска MCP-серверов, иначе индексация завершится с ошибкой.
{% endhint %}

## Устранение проблем

### LM Studio не видит GPU

1. Обновите драйверы NVIDIA
2. Установите CUDA Toolkit
3. Перезапустите компьютер

### Ошибка "Out of memory"

1. Используйте модель с меньшей квантизацией (Q4_K_M)
2. Закройте другие приложения
3. Уменьшите Context Length

### MCP-сервер не подключается

1. Убедитесь, что LM Studio Server запущен
2. Проверьте, что порт 1234 не занят
3. Проверьте URL: `http://host.docker.internal:1234/v1`
