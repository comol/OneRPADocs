# Переменные окружения

Сводная таблица переменных окружения для всех MCP-серверов.

## Общие переменные

Эти переменные используются большинством серверов:

| Переменная | Описание | Обязательная | По умолчанию |
|------------|----------|--------------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Да | — |
| `RESET_DATABASE` | Переиндексировать данные | Нет | `true` |
| `RESET_CACHE` | Перезагрузить модель | Нет | `true` |
| `USESSE` | SSE транспорт | Нет | `false` |

## Embedding модели (LM Studio / Ollama)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `OPENAI_API_BASE` | URL API | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ API | `lm-studio` |
| `OPENAI_MODEL` | Модель embedding | `Qwen3-Embedding-4B` |

## Embedding модели (CPU)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `EMBEDDING_MODEL` | Модель с Hugging Face | `intfloat/multilingual-e5-base` |

{% hint style="info" %}
Если указан `OPENAI_API_KEY`, используется внешнее API. Иначе — встроенная CPU модель.
{% endhint %}

## Переменные по серверам

### HelpSearchServer (порт 8003)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `1C_BIN_PATH` | Путь к bin в контейнере | `/1c_docs` |
| `RESET_DATABASE` | Переиндексировать | `true` |
| `RESET_CACHE` | Перезагрузить модель | `true` |

### CodeMetadataSearchServer (порт 8000)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `METADATA_PATH` | Путь к метаданным | `/app/metadata` |
| `CODE_PATH` | Путь к коду | `/app/code` |
| `RESET_DATABASE` | Переиндексировать | `true` |

### SSLSearchServer (порт 8008)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `SSL_VERSION` | Версия БСП | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `true` |

### Graph Metadata Search (порт 8006)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `NEO4J_URI` | URI Neo4j | `bolt://neo4j:7687` |
| `NEO4J_USERNAME` | Пользователь | `neo4j` |
| `NEO4J_PASSWORD` | Пароль | Обязательно |
| `METADATA_DIRECTORY` | Путь к метаданным | `/app/metadata` |
| `PROJECT_NAME` | Название проекта | `1C Metadata Project` |

### 1CCodeChecker (порт 8007)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `ONEC_AI_TOKEN` | Токен 1С:Напарник | Обязательно |

### SyntaxCheckServer (порт 8002)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `USESSE` | SSE транспорт | `false` |

### FormsServer (порт 8011)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `PORT` | Порт сервера | `8011` |

### TemplatesSearchServer (порт 8004)

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `RESET_DATABASE` | Переиндексировать | `true` |

## Примеры

### Минимальный набор (CPU)

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY
```

### С LM Studio

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
-e OPENAI_API_KEY=lm-studio `
-e OPENAI_MODEL=Qwen3-Embedding-4B
```

### С Ollama

```powershell
-e LICENSE_KEY=YOUR_LICENSE_KEY `
-e RESET_DATABASE=false `
-e OPENAI_API_BASE=http://host.docker.internal:11434/v1 `
-e OPENAI_API_KEY=ollama `
-e OPENAI_MODEL=qwen3:embedding-4b
```
