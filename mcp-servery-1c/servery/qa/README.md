# MCP QA — автоматизированное тестирование 1С

MCP QA позволяет ИИ управлять тестовой базой через логическую модель форм 1С:
читать окна и поля, находить элементы, вводить значения, нажимать кнопки и
команды, проверять результат.

С версии 0.6.0 контейнер сам работает менеджером тестирования: он подключается
к уже запущенному тест-клиенту 1С и говорит с ним по протоколу тестирования
напрямую. Отдельный процесс 1С с `/TestManager` и база-менеджер не нужны,
платформы 1С в образе нет.

## Поставка и требования

- Образ: `comol/qa_mcp:latest` — версия **0.6.0**, тот же образ под тегом `0.6.0`;
  только Linux/amd64, вариантов `light` и `arm64` нет.
- Ключ в `config.env`: `LICENSE_KEY_QA`; внутри сервера: `LICENSE_KEY`
  или путь к файлу ключа в `LICENSE_KEY_FILE`.
- Порт: **8020**; MCP: `http://127.0.0.1:8020/mcp`; liveness: `/healthz`.
- Имя MCP-подключения: `1c-qa`; транспорт Streamable HTTP с состоянием сеанса.
- Режим контейнера — нативный: `MCP_QA_EXECUTOR=native`, в командной строке
  `--backend native`. Он включён в образе по умолчанию.
- Тест-клиент запускает человек на своей Windows-машине: тонкий клиент с
  `/TestClient -TPort1538` или веб-клиент с `?TestClient`. Контейнер
  подключается к адресу из `MCP_QA_TESTCLIENT`, по умолчанию
  `host.docker.internal:1538`. Платформа 1С, её лицензия и база в поставку
  не входят.
- Проверку подлинности канала тестирования выполняет библиотека `pyspnego`
  (NTLM), она входит в образ. Учётная запись Windows задаётся в
  `MCP_QA_TESTCLIENT_USER`, `MCP_QA_TESTCLIENT_PASSWORD`,
  `MCP_QA_TESTCLIENT_DOMAIN`.
- Embeddings, LLM API и выгрузка метаданных не нужны.

Глобальные `IMAGE_VARIANT`/`IMAGE_TAG` к QA не применяются: образ задаёт
отдельное значение `QA_IMAGE`.

## Ограничения версии 0.6.0

- **Вход из Linux-контейнера не проверен.** Подключение к тест-клиенту проверено
  на Windows под текущим пользователем. Из контейнера сеть, приветствие
  тест-клиента и начало знакомства проходят, но без учётной записи Windows
  в `MCP_QA_TESTCLIENT_USER`/`_PASSWORD`/`_DOMAIN` вход не выполняется.
  Вход из контейнера с заданной учётной записью ещё не проверялся.
- **Снимок экрана и скрытый рабочий стол** в нативном режиме невозможны:
  `ui_screenshot` и `qa_start(hidden_desktop=True)` отвечают ошибкой
  `executor_capability`. Контейнер не видит экран Windows. Визуальную
  проверку в отчёте отмечайте как невыполненную.
- **Не все операции перенесены.** Нативно работают: подключение и отключение,
  активное окно, дерево окна и снимок формы (`ui_window_tree`, `ui_snapshot`),
  поиск элемента, нажатие кнопки, флажка, группы и команды интерфейса, ввод и
  чтение текста, открытие по навигационной ссылке, ожидание открытия и закрытия
  формы, сообщения пользователю, текущая ошибка. Остальные инструменты
  (подробная работа с таблицами и списками, табличный документ, календарь,
  выбор значения, диалоги, `ui_eval`, `qa_run_script`, установка расширения
  и другие) отвечают `executor_capability`.
- Запуск 1С, установка расширения `MCPQAClient` и сборка баз выполняются
  не сервером, а на машине с платформой — вручную или сторонними инструментами.

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
    -e MCP_QA_TESTCLIENT=host.docker.internal:1538 `
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

Учётную запись Windows для входа в канал тестирования передавайте так же,
через переменные окружения процесса (`--env MCP_QA_TESTCLIENT_USER` и т. д.),
а не в командной строке открытым текстом.

Дождитесь `healthy`, затем проверьте полноценный MCP-сеанс:

```powershell
docker exec qa-mcp python /app/check_http.py
```

Ожидаются `"backend": "native"`, `"version": "0.6.0"`, `tool_count: 62` и ответ
`qa_status` с `"executor": "native"`. `/healthz` проверяет только HTTP,
не доступность 1С. Ключ не записывается в `mcp.json`:

```json
{"mcpServers":{"1c-qa":{"url":"http://127.0.0.1:8020/mcp"}}}
```

Если задан `QA_HTTP_TOKEN`, добавьте клиенту `Authorization: Bearer <ваш токен>`.
Этот HTTP-токен отличается от лицензии. При изменении `QA_HTTP_PORT` поправьте URL.
Неверный/пустой ключ завершает сервер с `Invalid LICENSE_KEY` и кодом 1.
Можно вместо переменной передать `LICENSE_KEY_FILE`: смонтируйте файл только
для чтения и задайте путь внутри контейнера. Ошибка чтения файла блокирует запуск.

## Первый сеанс

На тестовой Windows-машине запустите тест-клиент, например:

```text
1cv8c ENTERPRISE /F"C:\Bases\TestCopy" /TestClient -TPort1538 /DisableStartupDialogs
```

Затем в одном MCP-сеансе:

```text
qa_status()
qa_start(connection="test")
ui_active_window()
ui_window_tree(detail="lite")
qa_stop()
```

В нативном режиме `qa_start` не запускает 1С, а подключается к тест-клиенту:
адрес берётся из `MCP_QA_TESTCLIENT`, параметр `port` заменяет порт,
`connection` — только имя сеанса. `qa_stop` отключается от клиента и не
закрывает его. Для веб-клиента с `TestClientID=<ид>` задайте тот же
идентификатор в `MCP_QA_TESTCLIENT_ID`. UI-действия изменяют данные —
используйте тестовую базу или копию.

## Состав образа

Прикладные модули, включая реализацию протокола тестирования, скомпилированы
Cython. Точки входа, регистрация инструментов и ASGI-интерфейс остаются в Python
для совместимости интроспекции.
