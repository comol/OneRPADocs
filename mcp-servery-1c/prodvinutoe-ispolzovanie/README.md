# Продвинутое использование

Расширенные сценарии работы с MCP-серверами.

## Содержание

- [Полный пример запуска](polnyj-primer-zapuska.md) — скрипт для всех серверов
- [Кеширование БД](keshirovanie-bd.md) — персистентность данных
- [Docker Compose](docker-compose.md) — оркестрация сервисов
- [GPU ускорение](gpu-uskorenie.md) — использование видеокарты

## Обзор

### Автоматизация

Создайте скрипты для:
- Запуска всех серверов одной командой
- Остановки и очистки
- Мониторинга состояния

### Персистентность

Сохраняйте данные между перезапусками:
- Векторные индексы
- Кэш моделей
- Данные Neo4j

### Производительность

Оптимизируйте скорость:
- GPU для embedding
- Правильные лимиты памяти
- SSD для баз данных

## Быстрые ссылки

### Скрипт запуска всех серверов

```powershell
# См. полный пример в polnyj-primer-zapuska.md
$LICENSE_KEY = "YOUR_LICENSE_KEY"

# Простые серверы (без embedding)
docker run -d -p 8002:8002 --name 1c_syntaxcheck_mcp -e LICENSE_KEY=$LICENSE_KEY comol/1c_syntaxcheck_mcp:latest
```

### Остановка всех серверов

```powershell
docker stop 1c_help_mcp 1c_syntaxcheck_mcp mcp_ssl_server template_search_mcp
```

### Проверка статуса

```powershell
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```
