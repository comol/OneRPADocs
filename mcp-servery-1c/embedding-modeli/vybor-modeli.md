# Выбор Embedding модели

Детальное сравнение всех вариантов embedding моделей для MCP-серверов.

## Сводная таблица

| Модель | Размерность | Качество | Скорость | Требования | Офлайн |
|--------|-------------|----------|----------|------------|--------|
| **Qwen3-Embedding-4B** (LM Studio) | 2560 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | GPU 4 ГБ | ✅ |
| **Qwen3-Embedding-8B** (LM Studio) | ~4096 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | GPU 8 ГБ | ✅ |
| **qwen3:embedding-4b** (Ollama) | 2560 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | GPU 4 ГБ | ✅ |
| **nomic-embed-text** (Ollama) | 768 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | GPU 2 ГБ | ✅ |
| **multilingual-e5-large** (CPU) | 1024 | ⭐⭐⭐⭐⭐ | ⭐⭐ | 4 ГБ RAM | ✅ |
| **multilingual-e5-base** (CPU) | 768 | ⭐⭐⭐⭐ | ⭐⭐⭐ | 2 ГБ RAM | ✅ |
| **multilingual-e5-small** (CPU) | 384 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 1 ГБ RAM | ✅ |

## Рекомендации по сценариям

### Сценарий 1: Есть NVIDIA GPU (4+ ГБ)

**Рекомендация: LM Studio + Qwen3-Embedding-4B**

```env
OPENAI_API_BASE=http://host.docker.internal:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL=Qwen3-Embedding-4B
```

Плюсы:
- Лучшее качество поиска
- Быстрая индексация (минуты)
- Отличная поддержка русского языка

### Сценарий 2: Есть NVIDIA GPU (8+ ГБ)

**Рекомендация: LM Studio + Qwen3-Embedding-8B**

```env
OPENAI_API_BASE=http://host.docker.internal:1234/v1
OPENAI_API_KEY=lm-studio
OPENAI_MODEL=Qwen3-Embedding-8B
```

Плюсы:
- Максимальное качество
- Ещё лучше понимает контекст

### Сценарий 3: Нет GPU, но есть время

**Рекомендация: CPU + multilingual-e5-base**

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-base
```

Плюсы:
- Не требует GPU
- Хороший баланс качество/скорость
- Полностью офлайн

### Сценарий 4: Нет GPU, нужна скорость

**Рекомендация: CPU + multilingual-e5-small**

```env
EMBEDDING_MODEL=intfloat/multilingual-e5-small
```

Плюсы:
- Самая быстрая на CPU
- Минимальные требования
- Приемлемое качество

### Сценарий 5: Уже используете Ollama

**Рекомендация: Ollama + qwen3:embedding-4b**

```env
OPENAI_API_BASE=http://host.docker.internal:11434/v1
OPENAI_API_KEY=ollama
OPENAI_MODEL=qwen3:embedding-4b
```

Плюсы:
- Один инструмент для всего
- Автозапуск службы
- CLI управление

## Качество поиска

### Что влияет на качество

1. **Размерность вектора** — больше = точнее, но медленнее
2. **Обучение модели** — Qwen обучен на русском языке
3. **Квантизация** — Q8 лучше Q4

### Тест на примере 1С

Запрос: "Как получить остатки товаров на складе"

| Модель | Найдено релевантных | Время поиска |
|--------|---------------------|--------------|
| Qwen3-Embedding-4B | 5/5 | 50 мс |
| multilingual-e5-base | 4/5 | 100 мс |
| multilingual-e5-small | 3/5 | 30 мс |

## Скорость индексации

Примерное время индексации справки платформы 1С (~5000 документов):

| Модель | Время индексации |
|--------|-----------------|
| Qwen3-Embedding-4B (GPU) | ~5 минут |
| Qwen3-Embedding-8B (GPU) | ~10 минут |
| multilingual-e5-small (CPU) | ~30 минут |
| multilingual-e5-base (CPU) | ~60 минут |
| multilingual-e5-large (CPU) | ~120 минут |

## Смена модели

{% hint style="warning" %}
При смене embedding модели требуется полная переиндексация всех данных.
{% endhint %}

### Процесс смены модели

1. Остановите контейнер
2. Измените переменные окружения
3. Установите `RESET_DATABASE=true`
4. Запустите контейнер
5. Дождитесь переиндексации
6. Измените `RESET_DATABASE=false` для следующих запусков

### Пример

```powershell
# Остановить контейнер
docker stop 1c_help_mcp

# Удалить старый контейнер
docker rm 1c_help_mcp

# Запустить с новой моделью и переиндексацией
docker run -d -p 8003:8003 `
  --name 1c_help_mcp `
  -e LICENSE_KEY=YOUR_LICENSE_KEY `
  -e RESET_DATABASE=true `
  -e OPENAI_API_BASE=http://host.docker.internal:1234/v1 `
  -e OPENAI_API_KEY=lm-studio `
  -e OPENAI_MODEL=Qwen3-Embedding-8B `
  -v "C:/Program Files/1cv8/8.3.23.1997/bin:/1c_docs" `
  -v "E:/bases/mcp_docs:/app/chroma_db" `
  comol/1c_help_mcp:latest
```

## Автоматическое определение размерности

MCP-серверы автоматически:

1. Тестируют embedding модель при запуске
2. Определяют размерность вектора
3. Сравнивают с существующим индексом
4. Пересоздают индекс если размерность изменилась

### Логи при смене модели

```
INFO - Testing embedding API with model: Qwen3-Embedding-8B
INFO - ✓ Embedding dimension: 4096
INFO - Found existing vector index with dimension: 2560
WARNING - ⚠️ DIMENSION MISMATCH DETECTED!
INFO - 🔄 Rebuilding vector index with correct dimensions...
INFO - ✓ Vector index created successfully!
```

## Итоговые рекомендации

| Ваша ситуация | Рекомендуемая модель |
|---------------|---------------------|
| GPU 4+ ГБ, хотите лучшее качество | LM Studio + Qwen3-Embedding-4B |
| GPU 8+ ГБ, нужно максимальное качество | LM Studio + Qwen3-Embedding-8B |
| Нет GPU, важно качество | CPU + multilingual-e5-base |
| Нет GPU, важна скорость | CPU + multilingual-e5-small |
| Уже используете Ollama | Ollama + qwen3:embedding-4b |
| Минимальные требования к VRAM | Ollama + nomic-embed-text |
