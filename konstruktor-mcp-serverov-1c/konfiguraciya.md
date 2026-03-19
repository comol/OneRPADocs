# Конфигурация

Настройка веб-сервера, публикации 1С и MCP-клиентов.

## Публикация HTTP-сервиса

### Файл default.vrd

Файл `default.vrd` описывает публикацию базы 1С на веб-сервере. Для MCP-сервера рекомендуется создать отдельную публикацию только с HTTP-сервисом `APA_MCP`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<point xmlns="http://v8.1c.ru/8.2/virtual-resource-system"
       xmlns:xs="http://www.w3.org/2001/XMLSchema"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       base="/mcptest"
       ib="File='E:\bases\YourBase';Usr='MCPUser';Pwd='password'"
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

### Параметры публикации

| Параметр | Описание | Рекомендация |
|----------|----------|--------------|
| `base` | URL-путь публикации | `/mcptest` или `/mcp` |
| `ib` | Строка подключения к базе | Используйте отдельного пользователя |
| `reuseSessions` | Повторное использование сеансов | `autouse` для производительности |
| `sessionMaxAge` | Время жизни сеанса (секунды) | 20-60 |
| `poolSize` | Размер пула соединений | 10-50 в зависимости от нагрузки |
| `poolTimeout` | Таймаут ожидания соединения | 5-10 секунд |

### Типы подключения

#### Файловая база

```xml
ib="File='E:\bases\YourBase';Usr='Admin';Pwd=''"
```

#### Клиент-серверная база

```xml
ib="Srvr='server1c';Ref='database';Usr='MCPUser';Pwd='password'"
```

{% hint style="warning" %}
**Безопасность**: Для MCP-сервера рекомендуется создать отдельного пользователя 1С с минимально необходимыми правами. Не используйте учётную запись администратора.
{% endhint %}

## Настройка Apache

### Одна публикация

Для простой конфигурации с одной публикацией добавьте в `httpd.conf`:

```apache
# MCP publication
Alias "/mcptest" "C:/basepublish/mcp/"
<Directory "C:/basepublish/mcp/">
    AllowOverride All
    Options None
    Order allow,deny
    Allow from all
    SetHandler 1c-application
    ManagedApplicationDescriptor "C:/basepublish/mcp/default.vrd"
</Directory>
```

### Несколько публикаций одной базы

Если нужно разделить основное приложение и MCP-сервис:

```apache
# Основное приложение 1С
Alias "/aiagents" "C:/basepublish/"
<Directory "C:/basepublish/">
    AllowOverride All
    Options None
    Order allow,deny
    Allow from all
    SetHandler 1c-application
    ManagedApplicationDescriptor "C:/basepublish/default.vrd"
</Directory>

# MCP сервис (отдельная публикация)
Alias "/mcptest" "C:/basepublish/mcp/"
<Directory "C:/basepublish/mcp/">
    AllowOverride All
    Options None
    Order allow,deny
    Allow from all
    SetHandler 1c-application
    ManagedApplicationDescriptor "C:/basepublish/mcp/default.vrd"
</Directory>
```

### Перезапуск Apache

После изменения конфигурации перезапустите Apache:

```cmd
# Windows
net stop Apache2.4
net start Apache2.4

# Или через службы
httpd -k restart
```

## Настройка MCP-клиентов

### Cursor IDE

Создайте файл `mcp.json` в папке `.cursor` вашего проекта или в глобальной папке настроек:

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

#### Несколько серверов

```json
{
  "mcpServers": { 
    "MCP-1C-Dev": {
      "url": "http://localhost/mcptest/hs/mcp",
      "connection_id": "1c_dev_001"
    },
    "MCP-1C-Prod": {
      "url": "http://server1c/mcpprod/hs/mcp",
      "connection_id": "1c_prod_001"
    }
  }
}
```

### Claude Desktop

Файл конфигурации Claude Desktop находится:
- Windows: `%APPDATA%\Claude\config.json`
- macOS: `~/Library/Application Support/Claude/config.json`

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

### Continue

Добавьте в конфигурацию Continue (`config.json`):

```json
{
  "mcpServers": [
    {
      "name": "1C MCP",
      "url": "http://localhost/mcptest/hs/mcp"
    }
  ]
}
```

## Параметры подключения

| Параметр | Описание | Обязательный |
|----------|----------|--------------|
| `url` | Полный URL HTTP-сервиса MCP | Да |
| `connection_id` | Идентификатор подключения | Рекомендуется |

### Формат URL

```
http[s]://<host>[:<port>]/<base>/hs/mcp
```

Где:
- `<host>` — имя хоста или IP-адрес веб-сервера
- `<port>` — порт (опционально, по умолчанию 80/443)
- `<base>` — путь публикации из `default.vrd`

Примеры:
- `http://localhost/mcptest/hs/mcp`
- `http://192.168.1.100:8080/mcp/hs/mcp`
- `https://1c.company.ru/mcpservice/hs/mcp`

## Проверка подключения

### Через браузер

Откройте в браузере URL HTTP-сервиса. При корректной настройке вы увидите ответ MCP-сервера (JSON с ошибкой о неверном методе — это нормально, сервер работает).

### Через curl

```bash
curl -X POST http://localhost/mcptest/hs/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'
```

Ожидаемый ответ — JSON со списком доступных инструментов.

### Через Cursor

1. Откройте Cursor с настроенным `mcp.json`
2. В статусной строке должен появиться индикатор подключения MCP
3. Откройте чат с ИИ и спросите: «Какие инструменты 1С доступны?»
