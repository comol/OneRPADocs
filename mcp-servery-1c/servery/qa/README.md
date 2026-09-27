# MCP QA — автоматизированное тестирование 1С

MCP QA позволяет ИИ управлять тестовой базой через логическую модель форм 1С:
читать окна и поля, выбирать строки, вводить значения, нажимать команды,
проверять результат, записывать и воспроизводить сценарии.

## Поставка и требования

- Образ: `comol/qa_mcp:latest` (версия **0.4.4**, тот же образ под тегом `0.4.4`), только Linux/amd64.
- Ключ в `config.env`: `LICENSE_KEY_QA`; внутри сервера: `LICENSE_KEY`.
- Порт: **8020**; MCP: `http://127.0.0.1:8020/mcp`; liveness: `/healthz`.
- Имя MCP-подключения: `1c-qa`; транспорт Streamable HTTP, с состоянием сеанса.
- На Windows отдельно запускается 1С с `/TestClient -TPort1538` и тестовой базой.
  Платформа 1С, её лицензия и база в Docker-образ не входят.
- Embeddings, LLM API и выгрузка метаданных не нужны. Для прямого подключения
  Docker не требуются расширение MCPQAClient, база-менеджер или внешняя обработка.
- Linux-контейнер предоставляет 10 инструментов `tc_*`. Запуск Windows-клиента,
  снимки окна и скрытый рабочий стол требуют Windows-развёртывания.

Вариантов `light` и `arm64` у QA нет. Глобальные `IMAGE_VARIANT`/`IMAGE_TAG`
к QA не применяются: используется отдельное значение `QA_IMAGE`.
С версии 0.4.4 ключ обязателен. Образ `0.4.3` ключ не проверял; при переходе
с него задайте `LICENSE_KEY_QA`, том `/data` переиспользуется.

## Установка (PowerShell)

Из корня распакованного `MCP_Distr`. Сначала проверьте, свободен ли порт 8020.
При существующем QA используйте его либо отдельные порт, имя контейнера и том.
Работающий сервер нельзя пересоздавать в ходе тестирования.

```powershell
$cfg = @{}
Get-Content -LiteralPath .\config.env | ForEach-Object {
  if ($_ -match '^([A-Z][A-Z0-9_]*)=(.*)$') { $cfg[$matches[1]] = $matches[2].Trim() }
}
if (-not $cfg['QA_IMAGE']) { throw 'Заполните QA_IMAGE в config.env' }
docker pull $cfg['QA_IMAGE']
if ($LASTEXITCODE -ne 0) { throw 'Не удалось загрузить образ QA' }
if (-not $cfg['LICENSE_KEY_QA']) { throw 'Заполните LICENSE_KEY_QA в config.env' }
$env:LICENSE_KEY = $cfg['LICENSE_KEY_QA']
$env:MCP_QA_HTTP_TOKEN = $cfg['QA_HTTP_TOKEN']
try {
  docker run -d --name qa-mcp --init --restart unless-stopped `
    --env LICENSE_KEY --env MCP_QA_HTTP_TOKEN `
    -p "127.0.0.1:$($cfg['QA_HTTP_PORT']):8020" `
    --add-host host.docker.internal:host-gateway `
    -v qa-mcp-data:/data $cfg['QA_IMAGE']
  if ($LASTEXITCODE -ne 0) { throw 'Ошибка запуска QA' }
} finally {
  Remove-Item Env:LICENSE_KEY -ErrorAction SilentlyContinue
  Remove-Item Env:MCP_QA_HTTP_TOKEN -ErrorAction SilentlyContinue
}
docker ps --filter name=qa-mcp
```

Дождитесь `healthy`, затем проверьте полноценный MCP-сеанс:

```powershell
docker exec qa-mcp python /app/check_http.py
```

Ожидаются `tool_count: 10` и успешный `list_connections`. `/healthz` проверяет
только HTTP, не доступность 1С. Ключ не записывается в `mcp.json`:

```json
{"mcpServers":{"1c-qa":{"url":"http://127.0.0.1:8020/mcp"}}}
```

Если задан `QA_HTTP_TOKEN`, добавьте клиенту `Authorization: Bearer <ваш токен>`.
Этот HTTP-токен отличается от лицензии. При изменении `QA_HTTP_PORT` поправьте URL.
Неверный/пустой ключ завершает сервер с `Invalid LICENSE_KEY` и кодом 1.
Можно вместо переменной передать `LICENSE_KEY_FILE`: смонтируйте файл только
для чтения и задайте путь внутри контейнера. Ошибка чтения файла блокирует запуск.

## Первый сеанс

На тестовой Windows-машине запустите клиент с `/TestClient -TPort1538`.
В одном MCP-сеансе:

```text
tc_session(action="list_connections")
tc_session(action="connect", host="host.docker.internal", port=1538, version="8.3.27.2130")
tc_app(action="get_active_window", connection_id="<полученный id>")
tc_session(action="disconnect", connection_id="<полученный id>")
```

Версию замените на точную версию работающей платформы. Для другой машины
вместо `host.docker.internal` укажите её доступный адрес. Одному порту
TestClient соответствует один управляющий сеанс. После переподключения
получите новые `connection_id` и ссылки на элементы. UI-действия изменяют
данные — используйте тестовую базу или копию.

## Состав образа

Прикладные модули скомпилированы Cython; точки входа, регистрация инструментов
и ASGI-интерфейс остаются в Python для совместимости интроспекции.
Сторонний адаптер 1c-testpilot не изменён; его исходники и лицензия AGPL-3.0-only
находятся в образе `/opt/testpilot-source.tar.gz`.
