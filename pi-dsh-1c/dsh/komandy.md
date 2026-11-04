# Команды DSH Desktop

Меню «Команды» ищет и по русскому названию. Ниже — команды контура 1С. Штатные команды ядра DSH (`/compact`, `/goal` и другие) остаются на своих местах.

Отличия от Pi — в [таблице соответствия](../sootvetstvie-komand.md). Режимы — на странице [режимов](../rezhimy.md).

## Доступ, качество и задача

| Команда | Зачем |
|---|---|
| `/mode ask\|plan\|build\|status` | Права записи. По умолчанию обычно BUILD |
| `/profile` | Алиас `/mode`: `explore`, `plan`, `build` |
| `/taskmode auto\|doc\|spec\|analysis\|bugfix\|full\|status` | Вид задачи. По умолчанию `auto` — «Авто»: вид определяется по тексту запроса. Старый `/quality` делает то же |
| `/sdlc lite\|standard\|full` | Глубина проверок, по умолчанию standard |
| `/economymode standard\|economy` | Обычное или расширенное делегирование |
| `/uitests on\|manual\|off` | UI-тесты. `on` соответствует auto |
| `/yolo on\|off\|status` | Обычные подтверждения текущего чата |
| `/process1c` | Маршрут без отдельной лекции |
| `/status1c`, `/status` | Снимок проекта, MCP, режима и OpenSpec |
| `/capabilities` | Поиск возможности по смыслу |

## Проект, MCP и правила

| Команда | Зачем |
|---|---|
| `/project-setup`, `/setup` | Мастер CF/DT, база, исходники, MCP |
| `/project-init` | Подготовка после мастера |
| `/config` | `init`, `status`, `analyze`, `update`, `apply`, `scaffold` |
| `/rule` | Добавить, показать, проверить конфликты, отключить |
| `/learn` | Факт, файл, URL, approve, reject, cancel |
| `/rules1c` | `status`, `plan`, `route <задача>`, развёртывание правил |
| `/lifecycle doctor` | Самопроверка. Алиас `/doctor` |
| `/lifecycle updates` | Проверка обновлений правил и MCP. `apply rules` применяет правила |
| `/lifecycle support` | Канал поддержки из меню жизненного цикла |
| `/installtools` | `status`, `menu`, план установки MCP и соседних инструментов |
| `/onec-check-mcp`, `/checkmcp` | Разбор отказа MCP |
| `/models1c` | Провайдеры и модели без секретов |

Тексты `/installmcp`, `/install-cognee`, `/install-openviking`, `/install-edt-mcp`, `/install-agent-browser` и `/install-windows-mcp` передают процедуру текущему чату. Установка идёт с обычными подтверждениями и сама сервер не поднимает.

## OpenSpec и знания поведения

| Команда | Зачем |
|---|---|
| `/openspec1c status\|init\|select\|show\|validate\|task` | Панель и команды change |
| `/opsx-explore`, `/opsx-propose`, `/opsx-apply`, `/opsx-update`, `/opsx-sync`, `/opsx-archive` | Навыки цикла. Для записи кода нужен BUILD |
| `/retro` | Ретроспектива. Файлы сама не пишет |
| `/evolve` | Очередь правил: `pending`, `diff`, `approve`, `reject` |
| `/lesson` | Урок к похожей задаче |
| `/improve` | Памятка или улучшение навыка с проверкой |
| `/self-improvement` | Та же памятка, без модели |
| `/local-extension` | Локальный plugin или skill |
| `/skill-eval` | План оценки. С `run` тратит вызовы модели |
| `/skills` | Список навыков |

## Выгрузка, проверка и сдача

| Команда | Зачем |
|---|---|
| `/loadfrom1cbase` | База → файлы. Аргумент: `base`, `extension:<Имя>` или `all`, либо перечень объектов |
| `/getconfigfiles` | Алиас частичной выгрузки |
| `/sync1c` | Свежесть, выгрузка, переиндексация. `apply` или `go` выполняют, `partial` — выбранное |
| `/update1cbase` | Файлы → база через `onec_deploy` после подтверждения |
| `/build-release` | Файлы → `.cf` или `.cfe` выбранного состава. База проекта не меняется |
| `/deploy-and-test` | Загрузка и проверка интерфейса |
| `/bsl-check` | Локальный BSL Language Server по выбранным файлам |
| `/deslop` | Артефакты ИИ в BSL. `apply` применяет |
| `/journal1c` | Журнал регистрации: число событий, объект, пользователь |
| `/formshot` | Снимок формы |
| `/meta-apply` | План правки метаданных, не выполненная запись |
| `/merge1c` | План нетипового обновления |
| `/features` | Ворота завершения, ручная проверка, строгость ревью, версии платформы и БСП |
| `/manual-check` | `Работает`, `Ошибка`, `Изменить` |
| `/help-version` | Справка по версии |
| `/receipt` | Отчёт действий. `html` кладёт файл в `.dsh/receipts` |

`/journal1c`, `/merge1c` и `/meta-apply` готовят процедуру и план. Саму операцию они не выполняют, пока вы отдельно не подтвердите инструмент.

## Контекст, фон и внешние входы

| Команда | Зачем |
|---|---|
| `/context1c` | Расход контекста. `export json`, `csv` или `markdown` |
| `/goal1c status\|recover\|stop` | Состояние автономной цели |
| `/later`, `/check`, `/loop`, `/heartbeat` | Отложенное сообщение, расписание, пульс |
| `/sessions` | Сводка фона |
| `/continue1c` | Только статус автопродолжения цели. Прерванный ход DSH продолжает сам; вручную — обычным сообщением в тот же чат |
| `/checkpoint`, `/rewind` | Ручная точка и возврат файлов |
| `/dsh-1c` | Импорт контекста: `discover`, `preview`, `apply`, `rollback` |
| `/support` | Черновик. `send` отправляет, `cancel` бросает |
| `/supportstatus` | Ответы и закрытие обращения |
| `/acp-connect` | Справка по входящему ACP. Сервер: `dsh-1c.ps1 -Acp`, мост к открытому Desktop: `-AcpConnect` |
| `/add-dir` | Соседняя папка только для чтения: путь, `list`, `clear` |
| `/pii` | Защита персональных данных, когда плагин включён: `status`, `on`, `off`, `project on\|off` |
| `/openviking-ingest` | Загрузка каталога в OpenViking |
| `/react` | Статус явного цикла ReAct |
| `/background`, `/clear` | Увести сессию или начать новый разговор |

## Инструменты, которыми пользуется модель

Помимо штатных `read`, `write`, `edit`, `grep` и `bash`:

- `mcp_search` и `mcp_call` — шлюз MCP
- `onec_metadata_tree`, `onec_metadata_open` — дерево метаданных
- `onec_bsl_query` — локальный BSL Language Server
- `knowledge_1c` — слой знаний
- `onec_openspec_progress`, `onec_openspec_handoff` — ход change
- `onec_deploy` — загрузка в базу после подтверждения
- `onec_com_execute` — вызов через COM, когда backend именно `com`
- `local_extension` — локальные навыки и плагины
- `subagent`, `subagent_fork`, `workflow` — делегирование

`onec_safe_db_mutation` в beta не регистрируется. Отдельного безопасного «измени базу без подтверждения» нет.
