# Установка

## Предварительные требования

1. Docker Engine или Docker Desktop запущен
2. Токен 1С:Напарник (см. [Получение токена](poluchenie-tokena.md))
3. Доступ в интернет к code.1c.ai

## Команда запуска

### Минимальная (обязательные переменные)

```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIK_TOKEN `
  comol/1c-code-checker:latest
```

### Расширенная (с дополнительными настройками)

```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIK_TOKEN `
  -e ONEC_CONFIG_NAME="Бухгалтерия предприятия" `
  comol/1c-code-checker:latest
```

### Beta с чтением исходников из workspace

Чтобы не передавать большие модули через JSON, смонтируйте репозиторий только для чтения и передавайте инструментам аргумент `files`:

```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker_beta `
  -e LICENSE_KEY=YOUR_BETA_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIK_TOKEN `
  -e ONEC_AI_WORKSPACE_PATH_MAP="C:\Work\My1CProject=/workspace" `
  -v "C:/Work/My1CProject:/workspace:ro" `
  comol/1c-code-checker:latest-beta
```

В образе `ONEC_AI_WORKSPACE_ROOTS=/workspace`. Для относительных путей достаточно read-only монтирования; `ONEC_AI_WORKSPACE_PATH_MAP` нужен для абсолютных Windows/UNC-путей, которые передаёт клиент, и не расширяет разрешённые корни. Сервер разрешает только обычные файлы внутри объявленных корней, после полного разрешения ссылок, и никогда их не записывает. Stable- и beta-ключи не смешиваются; см. [Каналы образов](../../kanaly-obrazov.md).

Полный список переменных окружения — в разделе [Конфигурация](konfiguraciya.md).

## Проверка работы

### Статус контейнера

```powershell
docker ps --filter name=1c_code_checker
```

### Просмотр логов

```powershell
docker logs 1c_code_checker
```

При успешном запуске в логах будет:

```
License key validated successfully.
Starting 1C Code Checker MCP server on http://0.0.0.0:8007/mcp using streamable-http (health: /health)
```

### Проверка состояния

```powershell
# Состояние: config_ok, direct_mode_ok, доступность upstream-возможностей
Invoke-RestMethod -Uri "http://localhost:8007/health"

# Счётчики транспортных сессий MCP
Invoke-RestMethod -Uri "http://localhost:8007/metrics/sessions"

# Идентичность выпуска (версия, digest образа, время сборки)
Invoke-RestMethod -Uri "http://localhost:8007/release"
```

{% hint style="info" %}
`/health` не обращается к 1С.ai и не раскрывает токен. Пока конфигурация не прошла проверку при старте, эндпоинт отвечает `503` — в логе при этом перечислены **все** проблемные настройки сразу, а не только первая.
{% endhint %}

## Конфигурация Cursor

Добавьте в `.cursor/mcp.json` вашего проекта:

```json
{
  "mcpServers": {
    "1c-code-checker-mcp": {
      "url": "http://localhost:8007/mcp",
      "connection_id": "1c_code_checker_001"
    }
  }
}
```

После добавления перезапустите Cursor или обновите MCP-подключения через настройки.

## Использование

### Доступные инструменты

Сервер предоставляет **11 инструментов MCP**. Основные:

| Инструмент        | Что делает                                                      |
| ----------------- | --------------------------------------------------------------- |
| `check_1c_code`   | Проверка кода на ошибки (синтаксис, логика, производительность) |
| `review_1c_code`  | Code review (стиль, стандарты ИТС)                              |
| `rewrite_1c_code` | Переписывание кода ИИ с улучшениями                             |
| `modify_1c_code`  | Модификация кода по инструкции                                  |
| `onec_help`       | Поиск по документации платформы                                 |
| `its_help`        | Поиск по базе знаний ИТС                                        |
| `ask_1c_ai`       | Свободный вопрос к 1С:Напарник                                  |

Полный список — в разделе [Инструменты](instrumenty.md).

### Пример в Cursor

Пользователь: "Проверь этот код на ошибки"

```bsl
Процедура ЗаполнитьТаблицу()
    Запрос = Новый Запрос;
    Запрос.Текст = "ВЫБРАТЬ * ИЗ Справочник.Номенклатура";
    Результат = Запрос.Выполнить();
    Выборка = Результат.Выбрать();
    Пока Выборка.Следующий() Цикл
        Сообщить(Выборка.Наименование);
    КонецЦикла;
КонецПроцедуры
```

ИИ автоматически вызывает `check_1c_code` и возвращает результат анализа, включая проверку синтаксиса (через upstream syntax-checker в direct-режиме) и анализ логики/производительности.

## Устранение проблем

### Ошибка авторизации

Проверьте токен 1С:Напарник:

* Токен действителен и не истёк
* Токен правильно указан в переменной `ONEC_AI_TOKEN`

### Нет связи с code.1c.ai

Проверьте:

* Доступ в интернет из контейнера
* Firewall не блокирует code.1c.ai (порт 443)

### Ошибка лицензии

При сообщении "Invalid license key" или "License key not provided":

* Проверьте, что `LICENSE_KEY` указан и корректен

### Ошибки direct mode

Если в ответах появляется `[DIRECT_TOOL_ERROR]`:

* Установите `MCP_TOOL_CALL_MODE=standard` для переключения на промпт-режим
* Или перезапустите контейнер — возможна временная недоступность upstream API
