# Конфигурация 1CCodeChecker

## Переменные окружения

### Обязательные

| Переменная | Описание |
|------------|----------|
| `LICENSE_KEY` | Лицензионный ключ MCP-сервера |
| `ONEC_AI_TOKEN` | Токен для доступа к API 1С:Напарник (code.1c.ai) |

### API подключение

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ONEC_AI_BASE_URL` | Базовый URL API 1С.ai | `https://code.1c.ai` |
| `ONEC_AI_TIMEOUT` | Таймаут запросов к API (секунды). Влияет на connect/write/pool; SSE-чтение не ограничено | `30` |
| `ONEC_AI_INPUT_MAX_LENGTH` | Максимальная длина входных данных (символы) | `100000` |

### Режим работы

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MCP_TOOL_CALL_MODE` | Режим вызова upstream: `direct` или `standard` (см. ниже) | `direct` |
| `ONEC_AI_SKILL_NAME` | Режим сессии Напарника: `custom` (с инструментами) или `raw` (прямые ответы) | `custom` |

### Языковые настройки

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ONEC_AI_UI_LANGUAGE` | Язык интерфейса | `russian` |
| `ONEC_AI_PROGRAMMING_LANGUAGE` | Язык программирования по умолчанию | *(пусто)* |
| `ONEC_AI_SCRIPT_LANGUAGE` | Скриптовый язык: `ru` или `en` | `ru` |

### Конфигурация 1С

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `ONEC_CONFIG_NAME` | Название конфигурации 1С по умолчанию для инструмента `config_help` (например, `ERP`, `Бухгалтерия предприятия`) | *(пусто)* |

### Управление сессиями

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `MAX_ACTIVE_SESSIONS` | Максимальное количество активных сессий (диалогов) с API | `10` |
| `SESSION_TTL` | Время жизни сессии в секундах | `3600` |

### Транспорт

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `USESSE` | Использовать SSE-транспорт вместо streamable-http | `false` |
| `HTTP_PORT` | Порт HTTP-сервера | `8007` |

## Режимы вызова инструментов

Переменная `MCP_TOOL_CALL_MODE` определяет, как MCP-инструменты взаимодействуют с API 1С.ai:

### Direct mode (по умолчанию)

```
MCP_TOOL_CALL_MODE=direct
```

Инструменты вызывают **upstream-инструменты** 1С.ai напрямую с точными аргументами. Обеспечивает более точные и структурированные результаты.

Маппинг upstream-инструментов:

| MCP-инструмент | Upstream-инструмент |
|----------------|---------------------|
| `check_1c_code` (синтаксис) | `mcp__syntax-checker__validate` |
| `search_1c_documentation` | `mcp__knowledge-hub__Search_Documentation` |
| `onec_help` | `mcp__knowledge-hub__Search_Documentation` |
| `its_help` | `mcp__knowledge-hub__Search_ITS` |
| `fetch_its` | `mcp__knowledge-hub__Fetch_ITS` |
| `diff_1c_documentation_versions` | `mcp__knowledge-hub__Diff_Documentation_Versions` |

При сбое direct-вызова сервер автоматически переключается на промпт-режим (fallback).

### Standard mode

```
MCP_TOOL_CALL_MODE=standard
```

Все инструменты используют **текстовые промпты** (natural language) для взаимодействия с API. Менее точный, но более устойчивый вариант.

{% hint style="info" %}
Если в ответах инструментов появляется `[DIRECT_TOOL_ERROR]`, это может означать временную недоступность upstream API. Установите `MCP_TOOL_CALL_MODE=standard` для переключения на промпт-режим.
{% endhint %}

## Пример docker run со всеми настройками

```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIR_TOKEN `
  -e MCP_TOOL_CALL_MODE=direct `
  -e ONEC_AI_TIMEOUT=60 `
  -e ONEC_CONFIG_NAME="ERP" `
  -e ONEC_AI_SKILL_NAME=custom `
  -e MAX_ACTIVE_SESSIONS=10 `
  -e SESSION_TTL=3600 `
  comol/1c-code-checker:latest
```

## Транспорт

По умолчанию сервер использует **streamable-http** транспорт. Для legacy-клиентов, которые поддерживают только SSE, установите `USESSE=true`.

| Режим | Переменная | Эндпоинт |
|-------|-----------|----------|
| streamable-http (по умолчанию) | `USESSE=false` | `http://localhost:8007/mcp` |
| SSE | `USESSE=true` | `http://localhost:8007/mcp` |
