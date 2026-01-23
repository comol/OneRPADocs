# Ollama (альтернатива)

Ollama — это инструмент для локального запуска LLM с командной строки. Может использоваться как альтернатива LM Studio для embedding.

## Когда использовать Ollama

- Уже используете Ollama для других задач
- Предпочитаете CLI интерфейс
- Нужна автоматизация через скрипты
- Хотите использовать один инструмент для LLM и embedding

## Требования

- **Windows 10/11** (64-bit)
- **NVIDIA GPU** с 4+ ГБ VRAM (рекомендуется)
- Или работа на CPU (медленнее)

## Установка Ollama

### Шаг 1: Скачивание

1. Перейдите на [ollama.com/download](https://ollama.com/download)
2. Скачайте версию для Windows
3. Запустите установщик

### Шаг 2: Проверка установки

```powershell
ollama --version
```

### Шаг 3: Первый запуск

```powershell
# Ollama запускается автоматически как служба
# Проверка работы:
ollama list
```

## Установка embedding модели

### Qwen3 Embedding (рекомендуется)

```powershell
ollama pull qwen3:embedding-4b
```

### Nomic Embed (альтернатива)

```powershell
ollama pull nomic-embed-text
```

### Проверка установленных моделей

```powershell
ollama list
```

## Сравнение моделей Ollama

| Модель | Размерность | Размер | Качество |
|--------|-------------|--------|----------|
| `qwen3:embedding-4b` | 2560 | ~2.5 ГБ | ⭐⭐⭐⭐⭐ |
| `qwen3:embedding-8b` | ~4096 | ~5 ГБ | ⭐⭐⭐⭐⭐ |
| `nomic-embed-text` | 768 | ~300 МБ | ⭐⭐⭐⭐ |
| `mxbai-embed-large` | 1024 | ~700 МБ | ⭐⭐⭐⭐ |

## Настройка MCP-серверов

### Переменные окружения для Ollama

```env
OPENAI_API_BASE=http://host.docker.internal:11434/v1
OPENAI_API_KEY=ollama
OPENAI_MODEL=qwen3:embedding-4b
```

{% hint style="info" %}
Ollama по умолчанию работает на порту **11434**.
{% endhint %}

### Пример команды Docker

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:11434/v1 `
  -e OPENAI_API_KEY=ollama `
  -e OPENAI_MODEL=qwen3:embedding-4b `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

## Проверка работы

### Тест Ollama API

```powershell
# Проверка списка моделей
curl http://localhost:11434/api/tags

# Тест embedding
$body = @{
    model = "qwen3:embedding-4b"
    prompt = "Тестовый текст"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:11434/api/embeddings" `
    -Method Post `
    -Body $body
```

### Проверка из Docker

```powershell
docker run --rm curlimages/curl:latest `
    curl -s http://host.docker.internal:11434/api/tags
```

## Ollama как служба Windows

Ollama устанавливается как служба Windows и запускается автоматически.

### Управление службой

```powershell
# Статус службы
Get-Service ollama

# Остановить
Stop-Service ollama

# Запустить
Start-Service ollama

# Перезапустить
Restart-Service ollama
```

### Автозапуск

Служба Ollama настроена на автоматический запуск. Проверить:

```powershell
Get-Service ollama | Select-Object Name, Status, StartType
```

## Сравнение с LM Studio

| Аспект | LM Studio | Ollama |
|--------|-----------|--------|
| Интерфейс | GUI | CLI |
| Настройка модели | Визуальная | Команды |
| Автозапуск | Нужно настроить | Из коробки |
| Порт по умолчанию | 1234 | 11434 |
| Совместимость API | OpenAI | OpenAI (частично) |
| Выбор квантизации | Гибкий | Ограниченный |

## Использование нескольких моделей

Ollama позволяет быстро переключаться между моделями:

```powershell
# Использовать Qwen для качества
-e OPENAI_MODEL=qwen3:embedding-4b

# Использовать Nomic для скорости
-e OPENAI_MODEL=nomic-embed-text
```

## Устранение проблем

### Ollama не запускается

```powershell
# Проверить службу
Get-Service ollama

# Проверить логи
Get-Content "$env:LOCALAPPDATA\Ollama\logs\server.log" -Tail 50
```

### Порт 11434 занят

```powershell
# Найти процесс
netstat -ano | findstr :11434

# Изменить порт Ollama
# Создать переменную окружения OLLAMA_HOST=0.0.0.0:11435
```

### Модель не загружается

```powershell
# Удалить и загрузить заново
ollama rm qwen3:embedding-4b
ollama pull qwen3:embedding-4b
```

### MCP-сервер не подключается

1. Убедитесь, что Ollama запущен: `ollama list`
2. Проверьте порт: `netstat -an | findstr :11434`
3. Проверьте URL: `http://host.docker.internal:11434/v1`

## Скрипт для проверки

```powershell
# Проверка Ollama и MCP подключения
Write-Host "Проверка Ollama..."

# Проверка службы
$service = Get-Service ollama -ErrorAction SilentlyContinue
if ($service.Status -eq "Running") {
    Write-Host "✓ Служба Ollama запущена" -ForegroundColor Green
} else {
    Write-Host "✗ Служба Ollama не запущена" -ForegroundColor Red
    exit
}

# Проверка API
try {
    $response = Invoke-RestMethod -Uri "http://localhost:11434/api/tags"
    Write-Host "✓ Ollama API доступен" -ForegroundColor Green
    Write-Host "Установленные модели:"
    $response.models | ForEach-Object { Write-Host "  - $($_.name)" }
} catch {
    Write-Host "✗ Ollama API недоступен" -ForegroundColor Red
}
```
