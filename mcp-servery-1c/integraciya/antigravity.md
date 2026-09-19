# Google Antigravity

Antigravity подключает те же HTTP-серверы, что и Cursor, но читает их из своего файла и со своим именем поля адреса.

## Где лежит конфигурация

| Область | Файл |
|---------|------|
| Глобально | `%USERPROFILE%\.gemini\config\mcp_config.json` |
| Один проект | `.agents/mcp_config.json` в корне рабочей области |

{% hint style="success" %}
В IDE файл открывается через интерфейс: «…» в панели агента → **MCP Servers** → **Manage MCP Servers** → **View raw config**.
{% endhint %}

## Структура файла

Схема — тот же словарь `mcpServers`, но адрес удалённого сервера задаётся полем **`serverUrl`**. Поля `url` и `httpUrl` Antigravity не принимает — сервер с ними молча не подключится.

```json
{
  "mcpServers": {
    "1c-docs-mcp":            { "serverUrl": "http://localhost:8003/mcp" },
    "1c-graph-metadata-mcp":  { "serverUrl": "http://localhost:8006/mcp" },
    "1c-code-metadata-mcp":   { "serverUrl": "http://localhost:8000/mcp" },
    "1c-ssl-mcp":             { "serverUrl": "http://localhost:8008/mcp" },
    "1c-templates-mcp":       { "serverUrl": "http://localhost:8004/mcp" },
    "1c-syntax-checker-mcp":  { "serverUrl": "http://localhost:8002/mcp" },
    "1c-code-checker-mcp":    { "serverUrl": "http://localhost:8007/mcp" }
  }
}
```

Оставьте только установленные серверы. Заголовки (например, `Authorization: Bearer …` для `add_template` / `plugin_reload` сервера шаблонов) задаются объектом `headers` рядом с `serverUrl`.

## Правила и скиллы 1c-rules

Antigravity читает правила проекта из `.agents/rules/*.md` (не более 12 000 символов на файл) и скиллы из `.agents/skills/<имя>/SKILL.md`; глобальные — `%USERPROFILE%\.gemini\GEMINI.md` и `%USERPROFILE%\.gemini\config\skills\`. Скиллы `1c-rules` — в стандартном формате `SKILL.md` и распознаются без преобразования. Отдельного адаптера `antigravity` у установщика правил пока нет: поставьте правила адаптером `other` (`install.ps1 -Tools other`) и перенесите `.ai-agent/rules` → `.agents/rules`, `.ai-agent/skills` → `.agents/skills`, либо скопируйте каталоги скиллов из репозитория правил напрямую.

## Проверка

После сохранения файла в панели **MCP Servers** у каждого сервера должен появиться список инструментов. Если списка нет — проверьте, что контейнер запущен (`docker ps`) и что адрес записан в `serverUrl`, а не в `url`.
