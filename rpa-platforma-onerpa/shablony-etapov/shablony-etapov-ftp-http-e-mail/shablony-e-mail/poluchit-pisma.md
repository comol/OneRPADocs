# Получить письма

![](<../../../../.gitbook/assets/шаблон этапа ПолучитьПисьма.png>)

Шаблон этапа "Получить письма"  предназначен для того, чтобы получить одно или несколько электронных писем с одного ящика, по определенным отборам. Например на конкретную дату, или  все письма до нужной вам даты, или письма в диапазоне дат. Вы можете задать нужное количество вам писем. Например, только первые 5 из всех писем за один день. Это определяется параметром "Количество писем".

Давайте подробнее рассмотрим параметры шаблона.

Почтовый сервер: здесь строкой указывается сервер. Можно указать строку, которую вы пишите в строке браузера для доступа к почте

Имя пользователя: это то имя, которое увидит получатель вашего письма.

E-mail пользователя: почтовый ящик, из которого будут извлекаться письма

Пароль: от почтового ящика

!!!параметры отбора JSON: указывается в виде строки JSON. Без фигурных скобок! Так как показано на рисунке. При этом тип выражения оставляем пустым. Если указать "Экранировать кавычки" будет вызвана ошибка

Отсортировать массив писем с конца: как сортировать список писем прежде, чем окончательно произвести отбор писем. Нужно, если вам необходимо выбрать письма более поздние по времени в рамках одной даты.

Количество писем. Выбрать первые 5, или 7, или сколько вам необходимо.

Массив писем. - Исходящий. Это все письма, которые мы получим на выходе.&#x20;
