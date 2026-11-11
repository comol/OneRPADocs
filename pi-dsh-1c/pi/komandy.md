# Команды Pi

Штатные команды ядра здесь не повторяются: `/login`, `/logout`, `/model`, `/thinking`, `/settings`, `/resume`, `/new`, `/session`, `/tree`, `/fork`, `/compact`, `/copy`, `/export`, `/quit` и соседние. Ниже — то, что добавляет бандл 1С и подключённые пакеты.

Навык вызывается как `/skill:<имя>`. Промпт — инструкция агенту, а не мгновенная кнопка. Актуальный перечень в установленной копии показывает `/commands1c` (Ctrl+Alt+H).

Полный маршрут и режимы: [маршрут](../marshrut.md), [режимы](../rezhimy.md). Отличия от DSH: [соответствие](../sootvetstvie-komand.md).

## Проект и готовность

| Команда | Зачем |
|---|---|
| `/setup` | Пять полей `.dev.env` в TUI. Существующий файл не перезаписывает. Пароли — в `Pi-1C.exe --setup` |
| `/project-setup` | Мастер: CF/DT, проект, база, исходники, MCP, подготовка агентом |
| `/initproject` | Тестовая база из базы, `.cf` или `.dt`, выгрузка в git и первый коммит. `empty` — только каркас |
| `/ready1c` | Карточка «Готовность». `/ready1c off` её скрывает |
| `/project1c` | Параметры проекта без секретов |
| `/preflight1c` | Повторить проверку модели, проекта, MCP, платформы, backend и свежести |
| `/status1c`, `/status` | Снимок футера. `/status` добавляет id сессии и версию бандла |
| `/help1c` | Короткая памятка: модель, проект, MCP |
| `/process1c` | Маршрут без хода модели |
| `/settings1c` | Меню настроек. `list` — памятка без меню |
| `/commands1c` | Навигатор по темам. То же Ctrl+Alt+H |
| `/quote latest`, `/quote pick` | Цитата из последнего сообщения или на выбор в поле ввода. Ctrl+Alt+Q или `/quote-selection` добавляют выделенный текст; во внешнем терминале сначала скопируйте фрагмент. В Pi Web — «Цитировать выделенное» |
| `/doctor` | Диагностика проекта, только чтение |
| `/target1c` | Основная конфигурация или `extension:Имя` |
| `/add-dir` | Внешняя папка только для чтения: путь, `list`, `clear` |
| `/docs` | Документы проекта: `add`, `show`, `search`, `read` |
| `/clear` | Новый разговор |
| `/usage` | Расход токенов без текста промптов |
| `/app-update` | Обновление поставки: проверка, загрузка, отдельное подтверждение установки |
| `/bundle-updates` | Фоновая проверка новых правил и MCP: `now`, `status`, `on`, `off`, `interval <часы>` |
| `/checkupdates`, `/updaterules` | Где правила и MCP и кто их обновляет, без тихой установки |
| `/rulesmodel` | Профиль правил под модель: `opus5`, `sonnet5`, `fable5`, `gpt56`, `gpt6` |
| `/awake` | Не давать Windows спать: `on`, `off`, `status` |
| `/question-timeout` | Автовыбор обычного вопроса через 5 минут |

## MCP, база и выгрузка

| Команда | Зачем |
|---|---|
| `/mcp1c` | Восемь включённых MCP: ready или degraded. `reconnect` поднимает молчащие, `reconnect all` или `reconnect <id>` — принудительно. `cache` — кэш чтений. Выключенный `1c-data-mcp` в ready не входит. Без аргументов открывает штатный `/mcp`, подкоманды сохраняются |
| `/mcp-settings` | Адреса и включение без значений секретов |
| `/mcp-project` | Сторонний HTTP, SSE или stdio MCP только этого проекта. `reload` перечитывает файл |
| `/mcp`, `/pi-mcp` | Статус адаптера, включая Cognee, OpenViking и EDT-MCP, если они подключены |
| `/mcp-auth` | OAuth MCP, если сервер его требует |
| `/backend1c` | Способ связи с 1С. `probe` проверяет COM без изменения данных |
| `/index1c` | Индексация: `status`, `poll`, `start`, `reindex`, `cancel`, `retry` — только для методов, которые сервер объявил |
| `/freshness1c` | Свежесть выгрузки. Сам дамп не запускает |
| `/checkmcp` | Что делать с недоступными серверами |
| `/installmcp`, `/updatemcp` | Процедура установки или обновления из `MCP_Distr` |
| `/sync1c` | Проверить свежесть, выгрузить и переиндексировать |
| `/loadfrom1cbase` | База → файлы, конфигурация или расширение |
| `/getconfigfiles` | База → файлы, отдельные объекты |
| `/update1cbase` | Файлы → база и обновление структуры |
| `/1c-deploy` | Центр деплоя: `plan`, `run`, `status`, `cancel`. Нужен BUILD |
| `/restore-testbase` | `.dt` и текущие исходники → тестовая база |
| `/check-uuid` | Дубли UUID в XML |

## Режимы, знания и OpenSpec

| Команда | Зачем |
|---|---|
| `/mode` | Вид задачи: `full`, `bugfix`, `doc`, `spec`, `analysis`, `status` |
| `/quality`, `/sdlc`, `/litemode` | Глубина проверок. `/litemode on` соответствует lite |
| `/access` | `ask`, `plan`, `build`, `status` |
| `/economymode` | `standard` или `economy` |
| `/uitests` | `on`, `manual`, `off`, `status` |
| `/config` | `init`, `status`, `analyze`, `update`, `apply`, `scaffold` |
| `/rule` | `add`, `list`, `show`, `audit`, `conflicts`, `disable` |
| `/learn` | Памятка, меню, факт, путь, URL, `approve`, `reject`, `retry`, `cancel` |
| `/openspec` | `init`, список, `use <change>`, `sync`, `archive`. `ticket <changes/имя> <ключ\|URL\|->` — связь с тикетом, `tickets` — открытые тикеты и источник |
| `/spec` | Открыть proposal, design, tasks или spec |
| `/tasks` | Прогресс и `set … done <revision> <доказательство>` |
| `/opsx-propose`, `/opsx-explore`, `/opsx-apply`, `/opsx-update`, `/opsx-sync`, `/opsx-archive` | Навыки цикла change |
| `/meta-apply` | План правки метаданных с revision |
| `/previewmode` | Показ diff метаданных: `on`, `auto`, `off` |

## Проверка, сдача и разбор

| Команда | Зачем |
|---|---|
| `/bsl` | `status`, `restart`, `preview` локального BSL Language Server |
| `/verify-changes` | Зафиксировать набор файлов и проверить |
| `/manual-check` | `Работает`, `Ошибка` или `Изменить` после неудачной автоматической проверки |
| `/receipt` | Отчёт сессии. `html` — экспорт |
| `/features` | Ворота проверок, строгость ревью, модель ревью, версия платформы, база БСП |
| `/help-version` | Справка и сравнение версий платформы и БСП |
| `/explain-changes` | Смысл правок текущей ветки |
| `/next-step` | Следующий шаг редактируемым запросом |
| `/debug1c` | Гипотезы и журнал регистрации |
| `/map1c` | Карта механизма: вопрос, `save`, `show` |
| `/preview1c` | Предпросмотр формы или макета, только чтение |
| `/graph1c` | Диаграмма связей |
| `/borrow1c` | Заимствование в расширение на файлах, без базы |
| `/vision` | Модель для скриншотов: `provider/id`, `fallback`, `off` |
| `/journal1c` | Журнал регистрации |
| `/formshot` | Снимок формы |
| `/deslop` | Список артефактов ИИ в BSL, применение по запросу |
| `/deploy-and-test` | Загрузка в тестовую базу и UI-тесты |
| `/test-fix-loop` | Цикл загрузки и исправления в бюджете итераций |
| `/build-release` | `.cf` и `.cfe` в `build/` |
| `/merge1c` | План нетипового обновления, по умолчанию dry-run |

## Автономия и делегирование

| Команда | Зачем |
|---|---|
| `/goal` | Автономный цикл. По умолчанию 300 вызовов, 40 итераций, 120 минут. Только BUILD |
| `/goal-resume`, `/goal-cancel` | Продолжить или остановить |
| `/auto`, `/roles` | Маршрут ролей и их модели |
| `/service-models` | Модель классификации и модель поиска |
| `/later`, `/heartbeat`, `/continue`, `/check`, `/loop` | Отложенное сообщение, пульс, продолжение, расписание |
| `/sessions`, `/operations` | Сводка фона и долгие операции |
| `/run` | Один субагент. Флаги `--bg` и `--fork` |
| `/subagents` | Модели, промпты и метаданные субагентов |
| `/subagent-cost`, `/subagents-doctor`, `/subagents-stop`, `/subagents-steer` | Стоимость, диагностика, остановка, подсказка |
| `/handoff`, `/background` | Новая сессия с черновиком, текущая остаётся родителем |
| `/acp-connect`, `/acp-disconnect`, `/acp1c` | Исходящая ACP-сессия |
| `/bg`, `/jobs`, `/logs`, `/kill` | Фон, список, вывод, остановка |
| `/yolo` | Обычные подтверждения. На старте выключает `YOLO_MODE=off` |
| `/profile` | Алиас `/access`: `explore`, `plan`, `build` |
| `/lesson` | `ok` или `fail` и факт. `list` показывает накопленное |
| `/project-agents` | Явно включить проектный `AGENTS.md` |
| `/queue`, `/q`, `/steer`, `/btw` | Очередь сообщений и побочный вопрос |
| `/plan` | Режим плана. Это не `/access plan` |

Команды пакетов сценариев — `/workflows`, `/deep-research`, `/adversarial-review`, `/code-review`, `/multi-perspective`, `/codebase-audit`, `/effort`, `/ultracode` — запускают многошаговые схемы на субагентах. `/fusion` гоняет пять моделей в фоне.

## Контекст, стиль и самоулучшение

| Команда | Зачем |
|---|---|
| `/economy-settings` | Экономия, память, инструменты, лимиты цели |
| `/context-usage`, `/context-export` | Сколько заняло окно, без текста промптов |
| `/context1c` | Состав контекста и статус правил. В DSH эта команда про другое |
| `/context` | `usage`, `injections`, `config` |
| `/auto-compact` | Порог сжатия истории. Другого автосжатия нет |
| `/rtk` | Сжатие вывода инструментов |
| `/reasoning` | Уровень рассуждения модели |
| `/caveman` | Краткий стиль: `lite`, `full`, `ultra`, `off`. Файл проекта слабее явной команды сессии |
| `/quota` | Квоты провайдеров в футере |
| `/retry` | Повторы запросов к модели |
| `/lang` | Язык интерфейса. Бандл засевает `ru` |
| `/improve` | Памятка или задача на улучшение навыка с проверкой |
| `/self-improvement` | Та же памятка, без модели |
| `/local-extension` | Список, чтение, проверка, diff, export, enable, disable, rollback |
| `/skill-eval` | Оценка навыка в свежих сессиях. Ходы модели — только после подтверждения |
| `/llm-rules` | Очередь `LLM-RULES.md`: пишет файл только оператор |
| `/evolve`, `/retro` | Кандидаты правил и ретроспектива. Сами правила не записывают |
| `/skills` | Список навыков |

## Память, модели и поддержка

| Команда | Зачем |
|---|---|
| `/installtools`, `/tools` | Статус MCP, Cognee, EDT-MCP, браузера, Vercel Agent Desktop, Headroom, RTK, OpenWhispr |
| `/openwhispr`, `/install-openwhispr` | Внешняя диктовка |
| `/install-cognee`, `/install-openviking`, `/install-edt-mcp` | Процедуры подключения. Сервер команда сама не поднимает |
| `/install-agent-browser`, `/install-agent-desktop` | Браузер и окна рабочего стола. Сервер команда сама не поднимает |
| `/install-rtk`, `/install-headroom` | Статус сжатия. Второй компрессор одновременно не включают |
| `/openviking-ingest` | Загрузить локальный каталог в OpenViking |
| `/websearch`, `/curator`, `/search` | Редкий внешний поиск, не замена MCP 1С |
| `/login`, `/provider`, `/connect` | Вход: подписки и API. Повторный вход того же провайдера добавляет ещё один именованный аккаунт |
| `/accounts` | Аккаунты провайдеров: основной, порядок резервных, переименование, повторный вход, ротация. `/accounts cursor` — сразу Cursor |
| `/model-fallback` | Основная и резервные модели разных провайдеров для чата или новых чатов. `status` — сохранённая цепочка |
| `/subscriptions setup` | Заранее скачать и проверить официальные CLI Google Antigravity и Meta Muse |
| `/pii` | Защита персональных данных: `status`, `on`, `off`, `default on\|off`. По умолчанию выключена |
| `/claude-login`, `/claude-status` | Подписка Claude Max |
| `/cursor-login`, `/cursor-refresh-models` | Подписка Cursor и каталог моделей |
| `/cursor-runtime`, `/cursor-mode`, `/cursor-fast` | Локальный или cloud runtime, режим agent или plan, быстрый режим |
| `/import-agent` | Импорт контекста Codex, Claude или Cursor с предпросмотром |
| `/telegram-setup` | Токен бота и подключение текущей сессии |
| `/support`, `/supportstatus` | Черновик обращения и статус. Отправляет оператор |
| `/toggle-free` | Режим только бесплатных моделей и переключатели отдельных провайдеров пакета pi-free |
| `/codex` | Адаптер Codex. Команда есть только если найден CLI `codex` |

Отката файлов командой нет. Возвращайте их через Git проекта. Данные базы — `/restore-testbase` или собственная копия.
