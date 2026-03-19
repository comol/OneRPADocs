# Быстрый старт

Минимальная настройка Конструктора MCP для начала работы.

## Предварительные требования

- Установленная платформа 1С:Предприятие 8.3.20+
- Веб-сервер Apache 2.4+ (или IIS)
- Лицензионный ключ Конструктора MCP
- MCP-клиент (Cursor, Claude Desktop или другой)

## Шаг 1. Установка расширения

1. Получите файл `OneMCP.cfe` из дистрибутива
2. Откройте вашу базу 1С в режиме «Конфигуратор»
3. Перейдите в меню **Конфигурация → Расширения конфигурации**
4. Нажмите **Добавить из файла** и выберите `OneMCP.cfe`
5. Обновите конфигурацию базы данных

## Шаг 2. Публикация HTTP-сервиса

1. Откройте публикацию базы на веб-сервере
2. Создайте отдельную публикацию только для MCP-сервиса
3. Используйте пример файла `default.vrd`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<point xmlns="http://v8.1c.ru/8.2/virtual-resource-system"
       xmlns:xs="http://www.w3.org/2001/XMLSchema"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       base="/mcptest"
       ib="File='C:\bases\YourBase';Usr='Admin';Pwd=''"
       enable="false">
    <debug enable="true" url="tcp://localhost"/>
    <httpServices publishByDefault="false"
                  publishExtensionsByDefault="false">
        <service name="APA_MCP"
                 rootUrl="mcp"
                 enable="true"
                 reuseSessions="autouse"
                 sessionMaxAge="20"
                 poolSize="10"
                 poolTimeout="5"/>
    </httpServices>
</point>
```

{% hint style="warning" %}
Замените параметры подключения к базе (`ib="..."`) на актуальные для вашей конфигурации.
{% endhint %}

## Шаг 3. Загрузка инструментов

1. Скачайте файл `ИнструментыДляРазработки.xml` из [репозитория](https://github.com/comol/mcp_designer_tools)
2. Откройте базу в режиме «1С:Предприятие»
3. Загрузите XML-файл стандартным механизмом загрузки данных конструктора

После загрузки в справочнике `APA_Инструменты` появятся:
- `vcexecutecode` — выполнение кода 1С
- `vcexecutequery` — выполнение запросов
- `vcvalidatequery` — проверка синтаксиса запросов
- `vcloggetlasterror` — получение последней ошибки

## Шаг 4. Настройка MCP-клиента

Создайте файл конфигурации для вашего MCP-клиента.

### Cursor IDE

Создайте файл `mcp.json` в папке `.cursor` вашего проекта:

```json
{
  "mcpServers": { 
    "MCP-1C": {
      "url": "http://localhost/mcptest/hs/mcp",
      "connection_id": "1c_mcp_001"
    }
  }
}
```

### Claude Desktop

Добавьте в файл конфигурации Claude Desktop:

```json
{
  "mcpServers": {
    "1c-constructor": {
      "url": "http://localhost/mcptest/hs/mcp",
      "connection_id": "1c_mcp_001"
    }
  }
}
```

## Шаг 5. Проверка работы

1. Откройте Cursor или другой MCP-клиент
2. Убедитесь, что MCP-сервер подключён (в Cursor — индикатор в статусной строке)
3. Попросите ИИ выполнить простой запрос:

```
Выполни запрос к 1С: ВЫБРАТЬ ПЕРВЫЕ 5 Справочник.Номенклатура.Наименование
```

Если всё настроено верно, ИИ вызовет инструмент `vcexecutequery` и вернёт результат.

## Что дальше?

- [Установка](ustanovka.md) — подробная инструкция по установке
- [Конфигурация](konfiguraciya.md) — настройка Apache и продуктивной среды
- [Инструменты](instrumenty.md) — создание собственных инструментов
