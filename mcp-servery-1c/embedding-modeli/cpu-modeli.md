# CPU режим

Если у вас нет видеокарты NVIDIA или недостаточно VRAM, MCP-серверы могут использовать встроенные CPU-модели.

## Когда использовать CPU режим

* Нет видеокарты NVIDIA
* VRAM менее 4 ГБ
* Нужна максимальная простота настройки
* Работа полностью офлайн

## Доступные модели

MCP-серверы используют модели семейства **multilingual-e5** из библиотеки sentence-transformers.

| Модель                           | Размерность | Размер   | Скорость  | Качество |
| -------------------------------- | ----------- | -------- | --------- | -------- |
| `intfloat/multilingual-e5-small` | 384         | \~500 МБ | Быстрая   | ⭐⭐⭐      |
| `intfloat/multilingual-e5-base`  | 768         | \~1 ГБ   | Средняя   | ⭐⭐⭐⭐     |
| `intfloat/multilingual-e5-large` | 1024        | \~2 ГБ   | Медленная | ⭐⭐⭐⭐⭐    |

## Рекомендации по выбору

### Быстрая работа (по умолчанию)

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-small
```

* Самая быстрая индексация
* Минимальное потребление RAM
* Приемлемое качество поиска

### Лучший баланс

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-base
```

* Хороший баланс скорость/качество
* Рекомендуется для большинства случаев
* Умеренное потребление RAM

### Максимальное качество

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-large
```

* Лучшее качество поиска на CPU
* Медленная индексация
* Требует больше RAM

## Настройка MCP-серверов

### Использование CPU модели

Чтобы использовать встроенную CPU модель, **НЕ указывайте** параметры `OPENAI_API_*`:

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

{% hint style="warning" %}
Если указать `OPENAI_API_KEY`, сервер будет пытаться использовать внешнее API вместо встроенной модели.
{% endhint %}

### Кэширование модели

При первом запуске модель скачивается с Hugging Face. Чтобы не скачивать повторно:

```powershell
-v "E:/bases/mcp_model_cache:/app/model_cache"
```

## Первый запуск

### Что происходит при первом запуске

1. **Скачивание модели** (несколько минут)
   * Модель загружается с huggingface.co
   * Сохраняется в `/app/model_cache`
2. **Индексация данных** (от часов до суток)
   * Зависит от объёма данных
   * CPU модели работают значительно медленнее GPU

{% hint style="warning" %}
**Для пользователей из России:** Сайт huggingface.co может быть заблокирован. В этом случае:
- Используйте VPN для первоначального скачивания модели
- После скачивания модель кэшируется локально и VPN больше не нужен
- Обязательно монтируйте том `/app/model_cache` для сохранения скачанной модели
{% endhint %}

### Мониторинг прогресса

```powershell
# Просмотр логов в реальном времени
docker logs -f 1c_help_mcp
```

Пример вывода:

```
Downloading model intfloat/multilingual-e5-base...
Model downloaded successfully
Starting indexing...
Indexed 1000/5000 documents...
Indexed 2000/5000 documents...
```

## Оптимизация производительности

### Увеличение RAM для Docker

1. Docker Desktop → Settings → Resources
2. Увеличьте Memory до 8+ ГБ
3. Apply & Restart

### Использование нескольких ядер

CPU модели автоматически используют многопоточность. Убедитесь, что Docker имеет доступ к нескольким ядрам:

1. Docker Desktop → Settings → Resources
2. CPUs: 4+ ядер

### Параметр RESET\_DATABASE

```env
RESET_DATABASE=false
```

* При `false` — используется существующий индекс
* При `true` — полная переиндексация при каждом запуске

<mark style="color:$danger;">**ВНИМАНИЕ! Полная переиндексация может занимать много времени!**</mark>

## Сравнение с GPU

| Аспект              | CPU (e5-base)        | GPU (Qwen-4B через LM Studio) |
| ------------------- | -------------------- | ----------------------------- |
| Качество поиска     | Хорошее              | Отличное                      |
| Скорость индексации | 3-10 часов/1000 док  | 20-60 мин/1000 док            |
| Требования          | 8 ГБ RAM             | 4 ГБ VRAM                     |
| Настройка           | Простая              | Требует LM Studio             |

{% hint style="info" %}
Для GPU-ускорения рекомендуется использовать **LM Studio** — это самый простой и эффективный способ получить преимущества GPU без сложной настройки CUDA в Docker.
{% endhint %}

## Пример полной команды

### HelpSearchServer с CPU

```powershell
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e 1C_BIN_PATH=/1c_docs `
  -e RESET_CACHE=false `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/1c_help_mcp:latest
```

### SSLSearchServer с CPU

```powershell
docker run -d -p 8008:8008 `
  --name mcp_ssl_server `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e SSL_VERSION=3.1.11 `
  -e RESET_DATABASE=false `
  -e EMBEDDING_MODEL=intfloat/multilingual-e5-base `
  -v "E:/bases/mcp_ssl:/app/chroma_db" `
  -v "E:/bases/mcp_model_cache:/app/model_cache" `
  comol/mcp_ssl_server:latest
```

## Устранение проблем

### Ошибка скачивания модели

```
Error downloading model from Hugging Face
```

Решение:

1. Проверьте интернет-соединение
2. Проверьте доступ к huggingface.co
3. **Для пользователей из России:** Hugging Face может быть заблокирован — используйте VPN
4. После скачивания модели обязательно монтируйте том `/app/model_cache`, чтобы не скачивать повторно

### Недостаточно памяти

```
RuntimeError: Unable to allocate memory
```

Решение:

1. Используйте модель меньшего размера (e5-small)
2. Увеличьте RAM для Docker
3. Закройте другие приложения

### Медленная индексация

Это нормально для CPU режима. Советы:

1. Используйте e5-small для быстрой индексации
2. Увеличьте количество ядер для Docker
3. Запускайте индексацию ночью
