# Обновление

Процедура обновления компонентов OneAPA.

## Подготовка к обновлению

### Чек-лист

- [ ] Создана резервная копия базы 1С
- [ ] Создана резервная копия конфигурации Proxy
- [ ] Уведомлены пользователи
- [ ] Определено окно обслуживания
- [ ] Подготовлен план отката

## Обновление 1С конфигурации

### Шаг 1: Резервная копия

```bash
1cv8 DESIGNER /S server\base /DumpIB backup_before_update.dt
```

### Шаг 2: Объединение

1. Откройте базу в Конфигураторе
2. Меню → Сравнить, объединить с конфигурацией из файла
3. Выберите новую версию OneAPA
4. Выберите объекты для обновления
5. Выполните объединение

### Шаг 3: Обновление базы

```
Меню → Конфигурация → Обновить конфигурацию базы данных (F7)
```

### Шаг 4: Проверка

1. Откройте базу в режиме Предприятие
2. Проверьте работу агентов
3. Выполните тестовый запрос

## Обновление Proxy

### Шаг 1: Получение новой версии

```bash
cd /opt/oneapa-proxy
git pull  # или скопируйте файлы вручную
```

### Шаг 2: Обновление Docker

```bash
# Остановка
docker stop oneapa-proxy

# Сохранение старого образа
docker tag oneapa-proxy:latest oneapa-proxy:backup

# Сборка нового
docker build -t oneapa-proxy:latest .

# Запуск
docker run -d --name oneapa-proxy-new -p 9000:9000 oneapa-proxy:latest

# Проверка
curl http://localhost:9000/health

# Удаление старого
docker rm oneapa-proxy
docker rename oneapa-proxy-new oneapa-proxy
```

### Шаг 3: Синхронизация

Выполните "Обмен с Proxy" в 1С.

## Откат

### Откат 1С

```bash
1cv8 DESIGNER /S server\base /RestoreIB backup_before_update.dt
```

### Откат Proxy

```bash
docker stop oneapa-proxy
docker rm oneapa-proxy
docker tag oneapa-proxy:backup oneapa-proxy:latest
docker run -d --name oneapa-proxy -p 9000:9000 oneapa-proxy:latest
```

## Рекомендации

- Обновляйте в нерабочее время
- Тестируйте на тестовой среде
- Имейте план отката
- Документируйте изменения
