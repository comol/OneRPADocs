# Конфигурация HelpSearchServer

## Переменные окружения

### Обязательные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `LICENSE_KEY` | Лицензионный ключ | `YOUR_LICENSE_KEY` |

### Пути

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `1C_BIN_PATH` | Путь к папке bin внутри контейнера | `/1c_docs` |

### Управление индексацией

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `RESET_DATABASE` | Переиндексировать данные при запуске | `true` |
| `RESET_CACHE` | Перезагрузить embedding модель | `true` |

{% hint style="info" %}
После первой индексации установите `RESET_DATABASE=false` для быстрого запуска.
{% endhint %}

### Транспорт

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `USESSE` | Использовать SSE транспорт (для legacy клиентов) | `false` |

### Embedding модели (LM Studio / Ollama)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API сервера | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API (любой для локальных серверов) | `lm-studio` |
| `OPENAI_MODEL` | Название модели | `Qwen3-Embedding-4B` |

### Embedding модели (встроенные CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Название модели с Hugging Face | `intfloat/multilingual-e5-base` |

{% hint style="warning" %}
Не указывайте `OPENAI_API_KEY` если хотите использовать встроенную CPU модель.
{% endhint %}

## Монтируемые тома

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `C:/Program Files/1cv8/X.X.XX.XXXX/bin` | `/1c_docs` | Справка платформы (только чтение) |
| `E:/bases/mcp_docs` | `/app/chroma_db` | Векторная база данных |
| `E:/bases/mcp_model_cache` | `/app/model_cache` | Кэш embedding моделей |

## Примеры конфигураций

### Минимальная (CPU)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  comol/1c_help_mcp:latest
```

### Рекомендуемая (LM Studio)

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

### С Ollama

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

### CPU с выбором модели

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

## Конфигурация Cursor

### mcp.json

```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    }
  }
}
```

### Путь к файлу

```
%APPDATA%\Cursor\User\globalStorage\mcp.json
```

## GPU ускорение

### Windows 11 с NVIDIA

```powershell
docker run -d -p 8003:8003 `
  --gpus all `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

{% hint style="info" %}
GPU ускорение работает только с Windows 11 и актуальными драйверами NVIDIA.
{% endhint %}
