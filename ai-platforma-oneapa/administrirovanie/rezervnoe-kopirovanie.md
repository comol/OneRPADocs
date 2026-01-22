# Резервное копирование

Руководство по резервному копированию OneAPA.

## Что копировать

| Компонент | Частота | Важность |
|-----------|---------|----------|
| База 1С | Ежедневно | Критическая |
| Конфигурация 1С | При изменениях | Высокая |
| Настройки Proxy | При изменениях | Средняя |
| Docker образы | При обновлении | Средняя |

## Резервное копирование 1С

### Выгрузка базы

```bash
# Windows
"C:\Program Files\1cv8\8.3.x.x\bin\1cv8.exe" DESIGNER /S server\base /DumpIB backup.dt

# Linux
/opt/1cv8/x86_64/8.3.x.x/1cv8 DESIGNER /S server/base /DumpIB backup.dt
```

### Регламентное задание

Настройте автоматическое резервное копирование через регламентные задания 1С.

## Резервное копирование Proxy

### Конфигурация

```bash
# Копирование файлов
cp -r /opt/oneapa-proxy/*.py /backup/proxy/
cp /opt/oneapa-proxy/requirements.txt /backup/proxy/
```

### Docker образ

```bash
# Сохранение образа
docker save oneapa-proxy:latest | gzip > backup/oneapa-proxy.tar.gz

# Восстановление
docker load < backup/oneapa-proxy.tar.gz
```

## Скрипт резервного копирования

```bash
#!/bin/bash

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup/oneapa/$DATE"
mkdir -p $BACKUP_DIR

# Backup Docker image
docker save oneapa-proxy:latest | gzip > $BACKUP_DIR/proxy.tar.gz

# Backup config
cp -r /opt/oneapa-proxy/*.py $BACKUP_DIR/

# Cleanup old backups (30 days)
find /backup/oneapa -type d -mtime +30 -exec rm -rf {} \;

echo "Backup completed: $BACKUP_DIR"
```

## Восстановление

### База 1С

```bash
1cv8 DESIGNER /S server\base /RestoreIB backup.dt
```

### Proxy

```bash
# Восстановление образа
docker load < oneapa-proxy.tar.gz

# Запуск контейнера
docker run -d --name oneapa-proxy -p 9000:9000 oneapa-proxy:latest
```

## Тестирование бэкапов

**Регулярно проверяйте возможность восстановления!**

1. Разверните тестовую среду
2. Восстановите из бэкапа
3. Проверьте работоспособность
