# Выгрузить файл на FTP

![](<../../../../.gitbook/assets/Выгрузить файл на FTP.png>)

Данный шаблон этапа существует для выгрузки (отправки) файлов на FTP. К примеру, в результате работы робота, формируется файл, который надо передать дальше на обработку. Этот шаблон имитирует отправку пользователем файла на файлообменник. Шаблон имеет следующие параметры:

* JSON параметры подключения. Все параметры подключения к ftp прописываются в одной строке, в формате JSON.  Указывается в виде строки JSON. Без фигурных скобок! Так как показано на рисунке. При этом тип выражения оставляем пустым. Если указать "Экранировать кавычки" будет вызвана ошибка.
* Полный путь к файлу. Здесь указывается путь до файла и имя самого файла с расширением.
* Каталог FTP. Указывается имя папки на FTP-сервере, куда мы отправляем файл.
