# Требования

MCP-серверы поставляются в виде Linux-контейнеров и не привязаны к операционной системе хоста. Их можно запускать на Windows, Linux, macOS и других платформах, где доступен Docker или совместимая среда запуска контейнеров, — то есть практически везде.

## Системные требования

| Компонент | Минимум | Рекомендуется |
|-----------|---------|---------------|
| **ОС** | Windows 10 (2004+), Linux или macOS с поддержкой Docker | Актуальная 64-битная ОС и актуальная версия Docker |
| **RAM** | 8 ГБ | 16 ГБ |
| **Диск** | HDD, 50 ГБ свободно | SSD, 100 ГБ свободно |
| **CPU** | 4 ядра | 8+ ядер |
| **GPU** | Не требуется | NVIDIA 4+ ГБ VRAM |

## Обязательные компоненты

1. **Docker Engine, Docker Desktop или совместимая среда** — для запуска Linux-контейнеров. На Windows см. [Docker Desktop и WSL2](docker-windows.md)
2. **[Cursor IDE](cursor-nastrojka.md)** — для работы с MCP

## Рекомендуемые компоненты

3. **[LM Studio](../embedding-modeli/lm-studio.md)** — для качественного embedding (если есть GPU)

## Сетевые требования

- Доступ к Docker Hub (hub.docker.com)
- Доступ к Hugging Face (huggingface.co) для скачивания моделей
- Локальные порты 8000-8008 должны быть свободны

Подробнее: [Сетевые требования](setevye-trebovaniya.md)

## Структура папок для данных

Создайте папки для хранения векторных баз данных:

Пример для Windows (PowerShell):

```powershell
# Создание структуры папок
New-Item -ItemType Directory -Force -Path @(
    "E:\bases\mcp_docs",
    "E:\bases\mcp_codemetadata",
    "E:\bases\mcp_ssl",
    "E:\bases\mcp_templates",
    "E:\bases\mcp_graph",
    "E:\bases\mcp_model_cache"
)
```

Пример для Linux и macOS (Bash):

```bash
mkdir -p ~/bases/{mcp_docs,mcp_codemetadata,mcp_ssl,mcp_templates,mcp_graph,mcp_model_cache}
```

| Папка | Назначение |
|-------|------------|
| `mcp_docs` | HelpSearchServer — индекс справки |
| `mcp_codemetadata` | CodeMetadataSearchServer — индекс метаданных |
| `mcp_ssl` | SSLSearchServer — индекс БСП |
| `mcp_templates` | TemplatesSearchServer — шаблоны кода |
| `mcp_graph` | Graph Metadata Search — данные Neo4j |
| `mcp_model_cache` | Общий кэш embedding моделей |

{% hint style="info" %}
Используйте любой абсолютный путь, доступный Docker на вашей платформе. Главное — использовать SSD для лучшей производительности. В примерах команд для Bash заменяйте перенос строк PowerShell (обратная кавычка) на символ обратной косой черты.
{% endhint %}
