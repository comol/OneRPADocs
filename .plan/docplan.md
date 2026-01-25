# План документации: MCP серверы для ИИ разработки на 1С

## Обзор проекта

Документация для набора MCP (Model Context Protocol) серверов, расширяющих возможности ИИ для работы с платформой 1С:Предприятие. MCP-серверы позволяют ИИ понимать код, метаданные, справку, бизнес-логику и конфигурации 1С.

### Источники информации

| Источник | Путь/URL | Назначение |
|----------|----------|------------|
| Основная документация | https://vibecoding1c.ru/mcp_server | Публичное описание серверов |
| **Инструкции установки (основа)** | `E:\MCP_Distr\` | Текстовые файлы с командами установки |

{% hint style="info" %}
**Основной источник команд установки:** `E:\MCP_Distr\*.txt` — эти файлы содержат актуальные команды Docker для каждого сервера.
{% endhint %}
| HelpSearchServer | `E:\1CMCP` | Репозиторий поиска по справке |
| 1CCodeChecker | `E:\MCP_1copilot\1copilot_MCP\MCP_1copilot` | Проверка кода через 1С:Напарник |
| CodeMetadataSearchServer | `E:\MCP_Metadata_new` | Поиск по метаданным и коду |
| Graph Metadata Search | `E:\MCP_Metadata_new` (Neo4j) | Графовый поиск по метаданным |
| SSLSearchServer | `E:\MCP_SSL` | Поиск по БСП |
| SyntaxCheckServer | `E:\MCP_syntax` | Проверка синтаксиса BSL |
| TemplatesSearchServer | `E:\MCP_Templates` | Поиск по шаблонам кода |
| FormsServer | `E:\MCP_Forms` | Генерация и анализ форм |

---

## Целевая платформа

| Параметр | Значение |
|----------|----------|
| **Операционная система** | Windows 10 (версия 2004+) / Windows 11 |
| **Метод установки** | Только Docker (Docker Desktop + WSL2) |
| **IDE** | Cursor |
| **Архитектура** | x64 (AMD64) |

### Почему только Docker на Windows?

1. **Единообразие** — одинаковые команды для всех пользователей
2. **Изоляция** — контейнеры не конфликтуют с системой
3. **Простота** — не нужно устанавливать Python, Neo4j и другие зависимости
4. **Обновления** — новая версия = новый образ, без ручного обновления
5. **Готовые образы** — все серверы доступны на Docker Hub (comol/*)

### Рекомендуемая конфигурация Embedding

| Вариант | Описание | Когда использовать |
|---------|----------|-------------------|
| **LM Studio + Qwen** | Локальная модель через LM Studio | Рекомендуется, высокое качество, GPU-ускорение |
| **CPU (e5-small)** | Встроенная модель в контейнере | Без GPU, минимальные требования |

{% hint style="info" %}
**Обновление от 25.01.2026:** Ollama исключён из рекомендаций. Для GPU-ускорения рекомендуется использовать LM Studio как более простое решение.
{% endhint %}

### Хранение данных

**Обязательно:** Монтировать внешние тома для сохранения:
- Векторных БД (ChromaDB) — `/app/chroma_db`
- Кэша моделей — `/app/model_cache`
- Данных Neo4j — для Graph Metadata Search

**Рекомендуемая структура на диске:**
```
E:\bases\
├── mcp_docs\          # HelpSearchServer
├── mcp_codemetadata\  # CodeMetadataSearchServer
├── mcp_ssl\           # SSLSearchServer
├── mcp_templates\     # TemplatesSearchServer
├── mcp_graph\         # Graph Metadata Search (Neo4j)
└── mcp_model_cache\   # Общий кэш моделей
```

---

## Структура документации

Документация размещается как раздел в корне: `mcp-servery-1c/`

```
mcp-servery-1c/
├── README.md                           # Введение и обзор
├── trebovaniya/
│   ├── README.md                       # Общие требования (Windows)
│   ├── docker-windows.md               # Установка Docker Desktop и WSL2 на Windows
│   ├── cursor-nastrojka.md             # Настройка Cursor IDE
│   └── setevye-trebovaniya.md          # Сеть и порты
│
├── embedding-modeli/
│   ├── README.md                       # Введение в Embedding модели
│   ├── lm-studio.md                    # LM Studio + Qwen (РЕКОМЕНДУЕТСЯ)
│   ├── cpu-modeli.md                   # CPU режим без GPU (e5, sentence-transformers)
│   ├── ollama.md                       # Альтернатива: Ollama
│   └── vybor-modeli.md                 # Сравнение и рекомендации
│
├── servery/
│   ├── README.md                       # Обзор всех серверов
│   │
│   ├── help-search-server/
│   │   ├── README.md                   # Описание и назначение
│   │   ├── ustanovka.md                # Установка и запуск
│   │   ├── konfiguraciya.md            # Параметры окружения
│   │   └── ispolzovanie.md             # Примеры использования
│   │
│   ├── graph-metadata-search/
│   │   ├── README.md                   # Описание и назначение
│   │   ├── ustanovka.md                # Установка (Neo4j + контейнер)
│   │   ├── podgotovka-dannyh.md        # Подготовка метаданных
│   │   └── konfiguraciya.md            # Параметры окружения
│   │
│   ├── code-metadata-search/
│   │   ├── README.md                   # Описание и назначение
│   │   ├── ustanovka.md                # Установка и запуск
│   │   ├── podgotovka-dannyh.md        # Выгрузка конфигурации и отчета
│   │   └── konfiguraciya.md            # Параметры окружения
│   │
│   ├── ssl-search-server/
│   │   ├── README.md                   # Описание и назначение (БСП)
│   │   ├── ustanovka.md                # Установка и запуск
│   │   └── konfiguraciya.md            # Параметры и версии БСП
│   │
│   ├── syntax-check-server/
│   │   ├── README.md                   # Описание (BSL Language Server)
│   │   └── ustanovka.md                # Установка и запуск
│   │
│   ├── templates-search-server/
│   │   ├── README.md                   # Описание шаблонов кода
│   │   ├── ustanovka.md                # Установка и запуск
│   │   ├── redaktirovanie-shablonov.md # Веб-интерфейс редактирования
│   │   └── svoi-shablony.md            # Добавление собственных шаблонов
│   │
│   ├── forms-server/
│   │   ├── README.md                   # Описание (генерация форм)
│   │   └── ustanovka.md                # Установка и запуск
│   │
│   └── code-checker/
│       ├── README.md                   # Описание (1С:Напарник)
│       ├── ustanovka.md                # Установка и запуск
│       └── poluchenie-tokena.md        # Получение токена 1С:Напарник
│
├── integraciya/
│   ├── README.md                       # Общие принципы интеграции
│   ├── cursor-mcp-json.md              # Формат mcp.json для Cursor
│   ├── neskolko-serverov.md            # Подключение нескольких серверов
│   └── cursor-rules.md                 # Project rules для 1С
│
├── prodvinutoe-ispolzovanie/
│   ├── README.md                       # Обзор продвинутых сценариев
│   ├── polnyj-primer-zapuska.md        # Запуск всех серверов (скрипт)
│   ├── keshirovanie-bd.md              # Персистентность векторных БД
│   ├── docker-compose.md               # Оркестрация через docker-compose
│   └── gpu-uskorenie.md                # Использование NVIDIA GPU (Windows 11)
│
├── ustranenie-nepoladok/
│   ├── README.md                       # Частые проблемы
│   ├── docker-problemy.md              # Проблемы Docker/WSL2
│   ├── indeksaciya.md                  # Проблемы индексации
│   └── podklyuchenie.md                # Проблемы подключения к MCP
│
└── prilozhenia/
    ├── porty-i-endpointy.md            # Таблица портов всех серверов
    ├── peremennye-okruzheniya.md       # Сводная таблица ENV
    └── glossarij.md                    # Термины: MCP, RAG, Embedding и т.д.
```

---

## Детальный план разделов

### 1. README.md (Введение)

**Содержание:**
- Что такое MCP-серверы и зачем они нужны
- Список всех MCP-серверов с кратким описанием
- Рекомендованный порядок установки (по приоритету)
- Ссылка на видеокурс https://vibecoding1c.ru/#course

**Приоритет важности серверов:**
1. HelpSearchServer - справка по платформе вашей версии
2. Graph Metadata Search - графовый поиск по метаданным
3. CodeMetadataSearchServer - поиск по метаданным и коду
4. SSLSearchServer - поиск по БСП
5. TemplatesSearchServer - шаблоны кода
6. SyntaxCheckServer - проверка синтаксиса
7. 1CCodeChecker - проверка через 1С:Напарник
8. FormsServer - генерация форм

---

### 2. Требования (Windows)

#### 2.1 docker-windows.md
**Предварительные требования:**
- Windows 10 версии 2004+ или Windows 11
- Включённая виртуализация в BIOS (Intel VT-x / AMD-V)
- Минимум 8 ГБ RAM (рекомендуется 16 ГБ)

**Установка WSL2:**
```powershell
wsl --install
wsl --set-default-version 2
```

**Установка Docker Desktop:**
- Скачать с https://www.docker.com/products/docker-desktop/
- Установить с включённой опцией WSL2
- Перезагрузить компьютер
- Проверить: `docker --version` и `docker run hello-world`

**Требования к дискам:**
- До 10 ГБ на каждый MCP сервер (образ + модели + индексы)
- Рекомендуется SSD для векторных БД

**Требования к сети:**
- Доступ к Docker Hub для скачивания образов
- Доступ к Hugging Face для embedding моделей (или использовать локальные)

#### 2.2 cursor-nastrojka.md
- Установка Cursor IDE для Windows
- Расположение файла mcp.json: `%APPDATA%\Cursor\User\globalStorage\`
- Включение MCP в настройках Cursor

#### 2.3 setevye-trebovaniya.md
- Список используемых портов (8000-8011)
- Настройка Windows Firewall
- Исключения для антивируса (Docker, WSL)
- Проверка доступности портов: `netstat -an | findstr :8003`

---

### 3. Embedding модели (КЛЮЧЕВОЙ РАЗДЕЛ)

#### 3.1 README.md
- Что такое Embedding модели и зачем они нужны для RAG
- Два основных варианта: **LM Studio** (рекомендуется) и **CPU** (встроенная модель)
- Автоматическое определение размерности векторов
- Влияние на качество и скорость поиска

#### 3.2 lm-studio.md (РЕКОМЕНДУЕМЫЙ ВАРИАНТ)

**Почему LM Studio:**
- Высокое качество embedding (Qwen3-Embedding)
- Использует GPU для ускорения
- OpenAI-совместимое API
- Бесплатно, работает локально

**Установка LM Studio:**
1. Скачать с https://lmstudio.ai/
2. Установить на Windows
3. Загрузить модель: `Qwen/Qwen3-Embedding-4B-GGUF`
4. Запустить локальный сервер (порт 1234 по умолчанию)

**Рекомендуемые модели для LM Studio:**
| Модель | Размерность | VRAM | Качество |
|--------|-------------|------|----------|
| `Qwen3-Embedding-4B` | 2560 | ~4 GB | Отличное |
| `Qwen3-Embedding-8B` | ~4096 | ~8 GB | Максимальное |

**Конфигурация MCP серверов для LM Studio:**
```env
OPENAI_API_BASE=http://host.docker.internal:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL=Qwen3-Embedding-4B
```

{% hint style="info" %}
`host.docker.internal` — специальный адрес для доступа из контейнера к хосту Windows
{% endhint %}

#### 3.3 cpu-modeli.md (БЕЗ GPU)

**Когда использовать:**
- Нет видеокарты NVIDIA
- Минимальные требования к ресурсам
- Работа полностью офлайн

**Встроенные модели (sentence-transformers):**
| Модель | Размерность | Скорость | Примечание |
|--------|-------------|----------|------------|
| `intfloat/multilingual-e5-small` | 384 | Быстрая | **По умолчанию** |
| `intfloat/multilingual-e5-base` | 768 | Средняя | Лучший баланс |
| `intfloat/multilingual-e5-large` | 1024 | Медленная | Высокое качество |

**Конфигурация (модель загружается автоматически):**
```env
EMBEDDING_MODEL=intfloat/multilingual-e5-small
# НЕ указывать OPENAI_API_KEY для использования встроенной модели
```

{% hint style="warning" %}
При первом запуске модель скачивается с Hugging Face (~500 MB - 2 GB)
{% endhint %}

#### 3.4 ollama.md (АЛЬТЕРНАТИВА)

**Когда использовать Ollama вместо LM Studio:**
- Уже используете Ollama для LLM
- Предпочитаете CLI интерфейс
- Нужна автоматизация через скрипты

**Установка Ollama на Windows:**
1. Скачать с https://ollama.com/download
2. Установить
3. Загрузить модель:
```powershell
ollama pull qwen3:embedding-4b
```

**Конфигурация:**
```env
OPENAI_API_BASE=http://host.docker.internal:11434/v1
OPENAI_API_KEY=ollama
OPENAI_MODEL=qwen3:embedding-4b
```

#### 3.5 vybor-modeli.md

**Сравнение вариантов:**
| Критерий | LM Studio + Qwen | CPU (e5-small) | Ollama |
|----------|------------------|----------------|--------|
| Качество | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Скорость | ⭐⭐⭐⭐⭐ (GPU) | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ (GPU) |
| Требования | GPU 4+ GB | CPU only | GPU 4+ GB |
| Простота | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Офлайн | Да | Да | Да |

**Рекомендации:**
| Сценарий | Рекомендация |
|----------|--------------|
| Есть GPU NVIDIA | LM Studio + Qwen3-Embedding-4B |
| Нет GPU | CPU + intfloat/multilingual-e5-base |
| Уже есть Ollama | Ollama + qwen3:embedding-4b |
| Максимальное качество | LM Studio + Qwen3-Embedding-8B |

**Смена модели:**
- При смене модели индекс пересоздается автоматически
- Система определяет размерность и пересоздает индекс
- Процесс переиндексации может занять несколько часов

---

### 4. Серверы (по каждому серверу)

#### Общая структура описания каждого сервера:

**README.md:**
- Назначение сервера
- Какие инструменты предоставляет ИИ
- Примеры запросов к серверу
- Когда использовать

**ustanovka.md:**
- Базовая команда docker run
- Продвинутая команда с персистентностью
- Объяснение всех параметров
- Примечание о времени индексации

**konfiguraciya.md:**
- Таблица переменных окружения
- Примеры конфигураций
- Настройка embedding модели

---

### 4.1 HelpSearchServer (Порт 8003)

**Назначение:** Поиск по справке платформы 1С конкретной версии.

**Почему важно:** ИИ получает актуальную информацию о методах и параметрах для вашей версии платформы.

**Рекомендуемая команда с LM Studio (Windows PowerShell):**
```powershell
docker run --rm -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "C:/Program Files/1cv8/8.3.XX.XXXX/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

**Команда с CPU (встроенная модель):**
```powershell
docker run --rm -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -v "C:/Program Files/1cv8/8.3.XX.XXXX/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

{% hint style="warning" %}
Замените `8.3.XX.XXXX` на вашу версию платформы, например `8.3.23.1997`
{% endhint %}

**Конфигурация Cursor (mcp.json):**
```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    }
  }
}
```

**Особенности путей в Windows:**
- Используйте прямые слеши: `C:/Program Files/1cv8/` или `E:/bases/`
- Docker Desktop автоматически конвертирует пути

**Переменные окружения:**
| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `LICENSE_KEY` | Лицензионный ключ | Обязательно |
| `1C_BIN_PATH` | Путь к папке bin внутри контейнера | `/1c_docs` |
| `RESET_CACHE` | Перезагрузить embedding модель | `true` |
| `RESET_DATABASE` | Переиндексировать все данные | `true` |
| `USESSE` | SSE транспорт (legacy клиенты) | `false` |
| `OPENAI_API_BASE` | URL LM Studio API | `http://host.docker.internal:1234/v1` |
| `OPENAI_API_KEY` | Ключ (любой для LM Studio) | `lm-studio` |
| `OPENAI_MODEL` | Модель embedding | `Qwen3-Embedding-4B` |

**Монтируемые тома:**
| Том | Путь в контейнере | Назначение |
|-----|-------------------|------------|
| Справка 1С | `/1c_docs` | Папка bin платформы (только чтение) |
| Векторная БД | `/app/chroma_db` | ChromaDB индекс |
| Кэш моделей | `/app/model_cache` | Скачанные embedding модели |

---

### 4.2 Graph Metadata Search (Порт 8006)

**Назначение:** Графовый поиск по метаданным конфигурации 1С через Neo4j.

**Особенности:**
- Понимает связи между объектами
- Анализирует архитектуру конфигурации
- Полезен для больших/чужих конфигураций

**Требования:**
- Neo4j база данных
- Отчет по метаданным из конфигуратора
- Выгрузка конфигурации в файлы

**Использование docker-compose.yml** (описать структуру)

---

### 4.3 CodeMetadataSearchServer (Порт 8000)

**Назначение:** Поиск по метаданным, справке конфигурации, паттернам кода.

**Подготовка данных в Конфигураторе 1С:**
1. Конфигурация → Отчет из конфигурации → сохранить в папку (например `E:\1C_Export\Report`)
2. Конфигурация → Выгрузить в файлы → сохранить в папку (например `E:\1C_Export\Files`)

**Рекомендуемая команда с LM Studio:**
```powershell
docker run --rm -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v E:/1C_Export/Report:/app/metadata `
  -v E:/1C_Export/Files:/app/code `
  -v E:/bases/mcp_codemetadata:/app/chroma_db `
  comol/1c_code_metadata_mcp:latest
```

**Команда с CPU (встроенная модель):**
```powershell
docker run --rm -d -p 8000:8000 `
  --name 1c_code_metadata_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e METADATA_PATH="/app/metadata" `
  -e CODE_PATH="/app/code" `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-small `
  -v E:/1C_Export/Report:/app/metadata `
  -v E:/1C_Export/Files:/app/code `
  -v E:/bases/mcp_codemetadata:/app/chroma_db `
  comol/1c_code_metadata_mcp:latest
```

**Монтируемые тома:**
| Том | Путь в контейнере | Назначение |
|-----|-------------------|------------|
| Отчет метаданных | `/app/metadata` | Текстовый отчет из Конфигуратора |
| Файлы кода | `/app/code` | Выгрузка конфигурации или EDT |
| Векторная БД | `/app/chroma_db` | ChromaDB индекс |

---

### 4.4 SSLSearchServer (Порт 8008)

**Назначение:** Поиск по Стандартной библиотеке подсистем (БСП).

**Важно:** Указать вашу версию БСП через `SSL_VERSION`.

**Рекомендуемая команда с LM Studio:**
```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v E:/bases/mcp_ssl:/app/chroma_db `
  comol/mcp_ssl_server:latest
```

**Команда с CPU:**
```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -v E:/bases/mcp_ssl:/app/chroma_db `
  comol/mcp_ssl_server:latest
```

{% hint style="info" %}
Узнать версию БСП: Конфигуратор → Справка → О программе → Библиотека стандартных подсистем
{% endhint %}

---

### 4.5 SyntaxCheckServer (Порт 8002)

**Назначение:** Проверка синтаксиса кода через BSL Language Server.

**Особенности:** 
- Самый простой в установке
- Не требует данных конфигурации
- Не требует embedding модели
- Не требует внешнего тома

**Команда (Windows PowerShell):**
```powershell
docker run -d -p 8002:8002 `
  --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest
```

{% hint style="success" %}
Этот сервер можно запустить сразу — он не требует подготовки данных или настройки embedding.
{% endhint %}

---

### 4.6 TemplatesSearchServer (Порт 8004)

**Назначение:** Поиск по шаблонам кода 1С.

**Особенности:**
- Содержит публичные шаблоны с https://fastcode.im/Templates
- Веб-интерфейс для редактирования: http://localhost:8004/extend/
- Можно добавлять свои шаблоны через веб-интерфейс

**Рекомендуемая команда с LM Studio:**
```powershell
docker run -d -p 8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-4B `
  -v "E:/bases/mcp_templates:/app/data" `
  comol/template-search-mcp:latest
```

**Команда с CPU:**
```powershell
docker run -d -p 8004:8004 `
  --name template_search_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=ai-forever/FRIDA `
  -v "E:/bases/mcp_templates:/app/data" `
  comol/template-search-mcp:latest
```

**Добавление своих шаблонов:**
1. Открыть http://localhost:8004/extend/
2. Добавить шаблон через веб-интерфейс
3. Или редактировать файлы напрямую в `E:/bases/mcp_templates`

---

### 4.7 FormsServer (Порт 8011)

**Назначение:** Контекст для генерации форм 1С.

**Особенности:**
- Не требует embedding модели
- Не требует внешних данных
- Предоставляет схемы форм для ИИ

**Инструменты MCP:**
- `get_xsd_schema()` — XSD схема форм 1С
- `get_json_schema()` — JSON схема форм 1С

**Команда (Windows PowerShell):**
```powershell
docker run -d -p 8011:8011 `
  --name 1c_forms_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  comol/1c_forms:latest
```

{% hint style="success" %}
Этот сервер можно запустить сразу — он содержит встроенные схемы форм.
{% endhint %}

---

### 4.8 1CCodeChecker (Порт 8007)

**Назначение:** Проверка кода через 1С:Напарник.

**Требования:** 
- Токен от 1С:Напарник (`ONEC_AI_TOKEN`)
- Доступен только партнёрам фирмы 1С
- Требует доступ в интернет к code.1c.ai

**Команда (Windows PowerShell):**
```powershell
docker run -d -p 8007:8007 `
  --name 1c_code_checker `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e ONEC_AI_TOKEN=YOUR_NAPARNIR_TOKEN `
  comol/1c-code-checker:latest
```

{% hint style="warning" %}
Токен 1С:Напарник можно получить только партнёрам фирмы 1С. 
Если у вас нет токена — используйте SyntaxCheckServer как альтернативу.
{% endhint %}

**Оригинальная идея:** https://github.com/artesk/1copilot_MCP

---

### 5. Интеграция

#### 5.1 cursor-mcp-json.md
**Полный пример mcp.json с несколькими серверами:**
```json
{
  "mcpServers": {
    "1c-docs-mcp": {
      "url": "http://localhost:8003/mcp",
      "connection_id": "1c_docs_service_001"
    },
    "1c-code-metadata-mcp": {
      "url": "http://localhost:8000/mcp",
      "connection_id": "1c_metadata_service_001"
    },
    "1c-ssl-mcp": {
      "url": "http://localhost:8008/mcp",
      "connection_id": "1c_ssl_service_001"
    },
    "1c-syntax-checker-mcp": {
      "url": "http://localhost:8002/mcp",
      "connection_id": "1c_lsp_service_001"
    },
    "1c-templates-mcp": {
      "url": "http://localhost:8004/mcp",
      "connection_id": "1c_templates_service_001"
    },
    "1c-forms-mcp": {
      "url": "http://localhost:8011/mcp",
      "connection_id": "1c_forms_service_001"
    },
    "1c-graph-metadata-mcp": {
      "url": "http://localhost:8006/mcp",
      "connection_id": "1c_graph_service_001"
    }
  }
}
```

#### 5.2 cursor-rules.md
- Ссылка на https://github.com/comol/cursor_rules_1c
- Важность включения rules для работы инструментов

---

### 6. Продвинутое использование

#### 6.1 gpu-uskorenie.md
- Добавление `--gpus all` для NVIDIA GPU
- Требования: Windows 11 + NVIDIA драйвер с поддержкой WSL2
- Установка NVIDIA Container Toolkit
- Проверка работоспособности: `docker run --gpus all nvidia/cuda:11.0-base nvidia-smi`

#### 6.2 keshirovanie-bd.md
- Монтирование томов для векторных БД
- Параметры RESET_CACHE и RESET_DATABASE
- Монтирование /app/model_cache для embedding моделей

#### 6.3 docker-compose.md
- Пример docker-compose.yml для всех серверов
- Сервисные зависимости
- Health checks

#### 6.4 polnyj-primer-zapuska.md

**Полный пример: запуск всех серверов с LM Studio**

**Предварительно:**
1. Установить Docker Desktop + WSL2
2. Установить LM Studio, загрузить `Qwen3-Embedding-4B`, запустить сервер
3. Создать папки для данных:
```powershell
mkdir E:\bases\mcp_docs
mkdir E:\bases\mcp_codemetadata  
mkdir E:\bases\mcp_ssl
mkdir E:\bases\mcp_templates
mkdir E:\bases\mcp_model_cache
```

**Batch-скрипт запуска (start_mcp_servers.ps1):**
```powershell
# Общие переменные
$LICENSE_KEY = "YOUR_LICENSE_KEY"
$LM_STUDIO_URL = "http://host.docker.internal:1234/v1"
$EMBEDDING_MODEL = "Qwen3-Embedding-4B"

# 1. HelpSearchServer (порт 8003)
docker run -d -p 8003:8003 --name 1c_help_mcp `
  -e LICENSE_KEY=$LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=$LM_STUDIO_URL `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=$EMBEDDING_MODEL `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest

# 2. SyntaxCheckServer (порт 8002) - без embedding
docker run -d -p 8002:8002 --name 1c_syntaxcheck_mcp `
  -e LICENSE_KEY=$LICENSE_KEY `
  comol/1c_syntaxcheck_mcp:latest

# 3. FormsServer (порт 8011) - без embedding
docker run -d -p 8011:8011 --name 1c_forms_mcp `
  -e LICENSE_KEY=$LICENSE_KEY `
  comol/1c_forms:latest

# 4. SSLSearchServer (порт 8008)
docker run -d -p 8008:8008 --name mcp_ssl_server `
  -e LICENSE_KEY=$LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=$LM_STUDIO_URL `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=$EMBEDDING_MODEL `
  -v "E:/bases/mcp_ssl:/app/chroma_db" `
  comol/mcp_ssl_server:latest

# 5. TemplatesSearchServer (порт 8004)
docker run -d -p 8004:8004 --name template_search_mcp `
  -e LICENSE_KEY=$LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e OPENAI_API_BASE=$LM_STUDIO_URL `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=$EMBEDDING_MODEL `
  -v "E:/bases/mcp_templates:/app/data" `
  comol/template-search-mcp:latest

Write-Host "Все серверы запущены. Проверьте статус: docker ps"
```

**Проверка статуса:**
```powershell
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

### 7. Устранение неполадок (Windows)

#### 7.1 docker-problemy.md
- WSL2 не установлен или неправильная версия
- Docker Desktop не запускается
- Ошибка "WSL 2 installation is incomplete"
- Нет места на диске (очистка Docker: `docker system prune -a`)
- Hyper-V конфликт с другими гипервизорами
- Антивирус блокирует Docker

#### 7.2 indeksaciya.md
- Индексация занимает слишком много времени (нормально до нескольких часов)
- Ошибки при смене embedding модели
- Как посмотреть логи: `docker logs -f container_name`
- Мониторинг через Docker Desktop GUI

#### 7.3 podklyuchenie.md
- Cursor не видит MCP сервер
- Ошибки connection refused
- Порт уже занят: `netstat -ano | findstr :8003`
- Windows Firewall блокирует подключения
- Проверка работы контейнера: `docker ps`

---

### 8. Приложения

#### 8.1 porty-i-endpointy.md

| Сервер | Порт | Endpoint | Описание |
|--------|------|----------|----------|
| HelpSearchServer | 8003 | /mcp | Справка платформы |
| CodeMetadataSearchServer | 8000 | /mcp | Метаданные и код |
| SyntaxCheckServer | 8002 | /mcp | Проверка синтаксиса |
| TemplatesSearchServer | 8004 | /mcp, /extend/ | Шаблоны кода |
| Graph Metadata Search | 8006 | /mcp, /search | Графовый поиск |
| 1CCodeChecker | 8007 | /mcp | 1С:Напарник |
| SSLSearchServer | 8008 | /mcp | БСП |
| FormsServer | 8011 | /mcp | Формы |

#### 8.2 peremennye-okruzheniya.md
Сводная таблица всех переменных окружения по всем серверам.

#### 8.3 glossarij.md
- **MCP** - Model Context Protocol
- **RAG** - Retrieval-Augmented Generation
- **Embedding** - Векторное представление текста
- **BSL** - Built-in Scripting Language (встроенный язык 1С)
- **БСП** - Библиотека стандартных подсистем
- **ChromaDB** - Векторная база данных

---

## Изменения в SUMMARY.md

Добавить раздел в корень:

```markdown
* [MCP серверы для 1С](mcp-servery-1c/README.md)
  * [Требования](mcp-servery-1c/trebovaniya/README.md)
    * [Docker Desktop и WSL2 (Windows)](mcp-servery-1c/trebovaniya/docker-windows.md)
    * [Настройка Cursor](mcp-servery-1c/trebovaniya/cursor-nastrojka.md)
    * [Сетевые требования](mcp-servery-1c/trebovaniya/setevye-trebovaniya.md)
  * [Embedding модели](mcp-servery-1c/embedding-modeli/README.md)
    * [LM Studio (рекомендуется)](mcp-servery-1c/embedding-modeli/lm-studio.md)
    * [CPU режим](mcp-servery-1c/embedding-modeli/cpu-modeli.md)
    * [Ollama](mcp-servery-1c/embedding-modeli/ollama.md)
    * [Выбор модели](mcp-servery-1c/embedding-modeli/vybor-modeli.md)
  * [MCP серверы](mcp-servery-1c/servery/README.md)
    * [HelpSearchServer](mcp-servery-1c/servery/help-search-server/README.md)
      * [Установка](mcp-servery-1c/servery/help-search-server/ustanovka.md)
      * [Конфигурация](mcp-servery-1c/servery/help-search-server/konfiguraciya.md)
      * [Использование](mcp-servery-1c/servery/help-search-server/ispolzovanie.md)
    * [Graph Metadata Search](mcp-servery-1c/servery/graph-metadata-search/README.md)
      * [Установка](mcp-servery-1c/servery/graph-metadata-search/ustanovka.md)
      * [Подготовка данных](mcp-servery-1c/servery/graph-metadata-search/podgotovka-dannyh.md)
      * [Конфигурация](mcp-servery-1c/servery/graph-metadata-search/konfiguraciya.md)
    * [CodeMetadataSearchServer](mcp-servery-1c/servery/code-metadata-search/README.md)
      * [Установка](mcp-servery-1c/servery/code-metadata-search/ustanovka.md)
      * [Подготовка данных](mcp-servery-1c/servery/code-metadata-search/podgotovka-dannyh.md)
      * [Конфигурация](mcp-servery-1c/servery/code-metadata-search/konfiguraciya.md)
    * [SSLSearchServer](mcp-servery-1c/servery/ssl-search-server/README.md)
      * [Установка](mcp-servery-1c/servery/ssl-search-server/ustanovka.md)
      * [Конфигурация](mcp-servery-1c/servery/ssl-search-server/konfiguraciya.md)
    * [SyntaxCheckServer](mcp-servery-1c/servery/syntax-check-server/README.md)
      * [Установка](mcp-servery-1c/servery/syntax-check-server/ustanovka.md)
    * [TemplatesSearchServer](mcp-servery-1c/servery/templates-search-server/README.md)
      * [Установка](mcp-servery-1c/servery/templates-search-server/ustanovka.md)
      * [Редактирование шаблонов](mcp-servery-1c/servery/templates-search-server/redaktirovanie-shablonov.md)
      * [Свои шаблоны](mcp-servery-1c/servery/templates-search-server/svoi-shablony.md)
    * [FormsServer](mcp-servery-1c/servery/forms-server/README.md)
      * [Установка](mcp-servery-1c/servery/forms-server/ustanovka.md)
    * [1CCodeChecker](mcp-servery-1c/servery/code-checker/README.md)
      * [Установка](mcp-servery-1c/servery/code-checker/ustanovka.md)
      * [Получение токена](mcp-servery-1c/servery/code-checker/poluchenie-tokena.md)
  * [Интеграция](mcp-servery-1c/integraciya/README.md)
    * [Формат mcp.json](mcp-servery-1c/integraciya/cursor-mcp-json.md)
    * [Несколько серверов](mcp-servery-1c/integraciya/neskolko-serverov.md)
    * [Cursor Rules для 1С](mcp-servery-1c/integraciya/cursor-rules.md)
  * [Продвинутое использование](mcp-servery-1c/prodvinutoe-ispolzovanie/README.md)
    * [Полный пример запуска](mcp-servery-1c/prodvinutoe-ispolzovanie/polnyj-primer-zapuska.md)
    * [Кеширование БД](mcp-servery-1c/prodvinutoe-ispolzovanie/keshirovanie-bd.md)
    * [Docker Compose](mcp-servery-1c/prodvinutoe-ispolzovanie/docker-compose.md)
    * [GPU ускорение](mcp-servery-1c/prodvinutoe-ispolzovanie/gpu-uskorenie.md)
  * [Устранение неполадок](mcp-servery-1c/ustranenie-nepoladok/README.md)
    * [Проблемы Docker](mcp-servery-1c/ustranenie-nepoladok/docker-problemy.md)
    * [Проблемы индексации](mcp-servery-1c/ustranenie-nepoladok/indeksaciya.md)
    * [Проблемы подключения](mcp-servery-1c/ustranenie-nepoladok/podklyuchenie.md)
  * [Приложения](mcp-servery-1c/prilozhenia/README.md)
    * [Порты и эндпоинты](mcp-servery-1c/prilozhenia/porty-i-endpointy.md)
    * [Переменные окружения](mcp-servery-1c/prilozhenia/peremennye-okruzheniya.md)
    * [Глоссарий](mcp-servery-1c/prilozhenia/glossarij.md)
```

---

## Рекомендации по написанию

### GitBook Markdown особенности

**Hints/Callouts:**
```markdown
{% hint style="info" %}
Информационное сообщение
{% endhint %}

{% hint style="warning" %}
Предупреждение
{% endhint %}

{% hint style="danger" %}
Важное предупреждение об опасности
{% endhint %}

{% hint style="success" %}
Успешное выполнение
{% endhint %}
```

### Особенности документации для Windows

1. **Все команды в формате PowerShell** — используется backtick (`) для переноса строк
2. **Пути в стиле Windows** — `E:\bases\mcp_docs`, `C:\Program Files\1cv8\`
3. **Docker Desktop** — единственный способ установки Docker на Windows
4. **WSL2 обязателен** — Docker Desktop требует WSL2 для работы контейнеров Linux
5. **Проверка портов** — команда `netstat -an | findstr :PORT`
6. **Firewall** — инструкции для Windows Defender Firewall

### Правила безопасности

1. **Не включать реальные лицензионные ключи** - использовать плейсхолдеры:
   - `YOUR_LICENSE_KEY`
   - `YOUR_NAPARNIR_TOKEN`
   - `sk-YOUR_OPENAI_KEY`

2. **Не включать реальные пути к данным** - использовать примеры:
   - `E:\bases\mcp_XXX`
   - `C:\Program Files\1cv8\8.3.XX.XXXX`

---

## Приоритет реализации

### Фаза 1 (Критично — быстрый старт)
1. README.md (введение, обзор серверов)
2. trebovaniya/docker-windows.md (Docker Desktop + WSL2)
3. **embedding-modeli/lm-studio.md** (рекомендуемый способ)
4. embedding-modeli/cpu-modeli.md (альтернатива без GPU)
5. servery/help-search-server/* (самый важный сервер)
6. integraciya/cursor-mcp-json.md

### Фаза 2 (Важно — основные серверы)
1. servery/syntax-check-server/* (простой, без embedding)
2. servery/forms-server/* (простой, без embedding)
3. servery/ssl-search-server/*
4. servery/code-metadata-search/* + подготовка данных

### Фаза 3 (Расширение)
1. servery/templates-search-server/* + свои шаблоны
2. servery/graph-metadata-search/* (Neo4j)
3. servery/code-checker/* (1С:Напарник)
4. embedding-modeli/ollama.md

### Фаза 4 (Продвинутое)
1. prodvinutoe-ispolzovanie/polnyj-primer-zapuska.md
2. prodvinutoe-ispolzovanie/docker-compose.md
3. prodvinutoe-ispolzovanie/gpu-uskorenie.md
4. ustranenie-nepoladok/*
5. prilozhenia/*

---

## Дополнительные материалы

### Видеоматериалы для ссылок
- Видеокурс по вайбкодингу: https://vibecoding1c.ru/#course
- Видео на странице: https://vibecoding1c.ru/mcp_server

### Связанные репозитории
- Cursor Rules для 1С: https://github.com/comol/cursor_rules_1c
- Оригинальная идея 1CCodeChecker: https://github.com/artesk/1copilot_MCP
- Шаблоны кода: https://fastcode.im/Templates

---

## Метаданные плана

| Параметр | Значение |
|----------|----------|
| Версия плана | 1.1 |
| Дата создания | 2026-01-23 |
| Дата обновления | 2026-01-25 |
| Формат документации | GitBook Markdown |
| Расположение | Корневой раздел |
| Количество файлов | ~50 файлов |
| Оценка объёма | ~15000-20000 слов |

---

## Изменения версии 1.1 (2026-01-25)

Внесены следующие ключевые изменения в документацию:

1. **Монтирование томов** — добавлены критические предупреждения о необходимости монтирования папок для векторных БД
2. **LM Studio** — указано, что base_url можно не указывать при локальном запуске
3. **mcp.json** — описано открытие через визуальный интерфейс Cursor
4. **Hugging Face** — добавлены предупреждения о возможной блокировке в РФ
5. **Ollama** — исключён из документации
6. **GPU ускорение** — рекомендуется использовать LM Studio
7. **docker pull** — добавлена документация по проверке и обновлению образов
8. **Флаг --rm** — объяснено назначение и рекомендация убирать
9. **Инструменты MCP** — добавлены таблицы инструментов для всех серверов
10. **Токен 1С:Напарник** — уточнено получение через ИТС и developer.1c.ru
11. **Несколько серверов** — добавлен раздел о запуске для разных баз/версий
12. **Выборочная индексация** — описана оптимизация через удаление файлов
13. **Пути E:/bases** — уточнено, что это примеры, не рекомендации
14. **Web интерфейсы** — добавлено описание для Templates и Graph metadata
15. **Время индексации** — увеличено в 20 раз для более реалистичных оценок
16. **FAQ** — использованы знания из FAQ (localhost в Docker, SSE транспорт)
