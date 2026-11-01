# Соответствие команд Pi и DSH

Команды специально не переименовывали. Одинаковое имя иногда значит разное. Стрелка в задачах выгрузки: «ИБ → файлы» — из базы в исходники, «файлы → ИБ» — из исходников в базу.

| Задача | Pi | DSH Desktop |
|---|---|---|
| Права ASK/PLAN/BUILD | `/access ask\|plan\|build` | `/mode ask\|plan\|build` |
| Вид задачи | `/mode full\|bugfix\|doc\|spec\|analysis` | `/taskmode auto\|doc\|spec\|analysis\|bugfix\|full` |
| Глубина проверок | `/quality` или `/sdlc` | `/sdlc` |
| ИБ → файлы, конфигурация или расширение | `/loadfrom1cbase` | `/loadfrom1cbase` |
| ИБ → файлы, выбранные объекты | `/getconfigfiles` | `/getconfigfiles` |
| ИБ → файлы, свежесть и переиндексация | `/sync1c` | `/sync1c` |
| файлы → ИБ | `/update1cbase`, `/1c-deploy` | `/update1cbase` |
| файлы → ИБ и UI-тесты | `/deploy-and-test`, `/test-fix-loop` | `/deploy-and-test` |
| `.dt` и исходники → тестовая ИБ | `/restore-testbase` | Навык `1c-metadata-manage` (`db-load-dt`), затем `/update1cbase` |
| BSL Language Server | `/bsl status` | `/bsl-check` |
| Расход контекста | `/context-usage`, `/context-export` | `/context1c` |
| OpenSpec | `/openspec` | `/openspec1c` |
| Продолжить или остановить цель | `/goal-resume`, `/goal-cancel` | `/goal1c recover`, `/goal1c stop` |
| Импорт настроек Codex, Claude, Cursor | `/import-agent` | `/dsh-1c` |
| Диагностика | `/doctor`, `/preflight1c` | `/lifecycle doctor` |
| Состояние MCP | `/mcp1c`, `/checkmcp` | `/status1c`, `/onec-check-mcp` |
| Кэш чтений MCP | `/mcp1c cache` | `/mcp-cache` |
| Обновления правил и MCP | `/bundle-updates`, `/checkupdates`, `/updaterules` | `/lifecycle updates` |
| Мастер проекта | `/setup`, `/project-setup`, `/initproject` | `/project-setup`, затем `/project-init` |
| Найти команду | `/commands1c` | `/capabilities` и меню «Команды» |
| Откат файлов | Git проекта | `/checkpoint`, `/rewind` |

## Одно имя — разный смысл

- **`/mode`.** В Pi это вид задачи. В DSH это права ASK/PLAN/BUILD.
- **`/quality`.** В Pi это глубина проверок, как `/sdlc`. В DSH это старый вход вида задачи. Глубину в обоих надёжнее задавать через `/sdlc`.
- **`/acp-connect`.** В Pi это исходящая сессия к другому агенту. В DSH команда объясняет входящее подключение; сервер ACP запускается как `dsh-1c.ps1 -Acp`.
- **`/context1c`.** В Pi это состав контекста и статус правил. В DSH это расход контекста, как `/context-usage` в Pi.
- **`/support`.** В Pi агент готовит текст, отправляете вы. В DSH `/support send` отправляет обращение.
- **`/continue`.** В Pi продолжает работу с подтверждённого состояния. В DSH работу продолжают обычным сообщением, а `/continue1c` только показывает статус автопродолжения.
- **`/profile`.** В Pi это совместимый алиас `/access`. В DSH это совместимый алиас `/mode`.

Навыки OpenSpec (`/opsx-propose`, `/opsx-explore`, `/opsx-apply`, `/opsx-update`, `/opsx-sync`, `/opsx-archive`) называются одинаково.
