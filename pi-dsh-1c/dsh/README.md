# DSH Desktop 1С

Поставка DeepSeek Harness для 1С. Основа — официальный Desktop **0.2.2-rc.1**, версия бандла **0.1.0**. Основная платформа — Windows x64. На macOS (Apple Silicon) это тоже окно Desktop, в Linux x64 та же сборка работает как Web в браузере: см. [Linux и macOS](../platformy.md). Отдельной Web-поставки для Windows нет. Браузерный и консольный режимы внутри Desktop сохранены как дополнительные.

Сайт сборки: [vibecoding1c.ru](https://vibecoding1c.ru/). Запуск — `DSH-Desktop-1C.exe` или `START_DSH_DESKTOP_1C.cmd`.

Обычный DSH даёт чат, модель и файлы. Этот бандл добавляет русский интерфейс, MCP 1С, OpenSpec, знания конфигурации, режимы ASK/PLAN/BUILD, дерево метаданных, редактор BSL и ручной откат файлов. Сверх этого — [ассистент](assistent.md), команды агентов, задачи по расписанию, доступ с телефона, Telegram, тикеты Jira, память Cognee и OpenViking, поручения Claude Code и Codex: см. [Дополнительные возможности](dopolnitelnye-vozmozhnosti.md).

## Три входа

| Вход | Когда |
|---|---|
| `DSH-Desktop-1C.exe` | Обычная работа. Папку проекта можно передать аргументом. `--select` выбирает проект, `--doctor` проверяет поставку, `--help` печатает ключи |
| `START_DSH_DESKTOP_1C.cmd` | Тот же EXE из командной строки |
| `dsh-1c.ps1` | Одна задача, подготовка профиля, ACP и диагностика. Окно Desktop — ключ `-Desktop` |

Приложение одно на пользователя. Закрытие окна оставляет его в трее. Полный перезапуск после смены адресов MCP — выход через значок в трее и новый запуск EXE.

Пресет проекта решает, какой контур 1С вообще включён. Для доработки конфигурации нужен **1С Full**. Подробности — [интерфейс и пресеты](interfejs-i-presety.md).

## Куда дальше

- [Установка и запуск](ustanovka-i-zapusk.md)
- [Интерфейс и пресеты](interfejs-i-presety.md)
- [Ассистент DSH](assistent.md)
- [Проект, MCP и модели](proekt-mcp-modeli.md)
- [Команды](komandy.md)
- [Проверка, откат и сдача](proverka-otkat-sdacha.md)
- [Автономия](avtonomiya.md)
- [Дополнительные возможности](dopolnitelnye-vozmozhnosti.md)
- [Ограничения](ogranicheniya.md)

Общее для обеих поставок: [модели, подписки и аккаунты](../modeli-i-podpiski.md), [защита персональных данных](../zashchita-pdn.md), [получение и обновление](../poluchenie-i-obnovlenie.md).
