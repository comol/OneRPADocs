# Шаблоны этапов 1С

![](../../.gitbook/assets/Логотип\_компании\_«1С».png)

Данная группа шаблонов предоставляет возможность пользователю создать робота взаимодействующего с решениями на базе платформы 1С.

Группа содержит три основных части "Интеграция", "Работа с записанными действиями" и "Работа с объектами 1С".

**Часть "Интеграция"** позволяет получить объект из формата EnterpriseData и сохранить объект в формате EnterpriseData.

Для облегчения интеграции с программными продуктами фирмы «1С» разработан формат обмена данными EnterpriseData. Формат основан на XML и является бизнес-ориентированным — описанные в нем структуры данных соответствуют бизнес-сущностям (документам и элементам справочников), представленным в программах «1С», например: акт выполненных работ, приходный кассовый ордер, контрагент, договор и т. п. Это делает формат интуитивно понятным и легким в использовании.

Пример перевода объекта 1С в формат EnterpriseData и схема взаимодействия представлены ниже.

![](<../../.gitbook/assets/image (32).png>)

![](<../../.gitbook/assets/image (33).png>)



![](<../../.gitbook/assets/image (34).png>)

**Часть "Работа с записанными действиями"** позволяет совершать действия с информационными базами.

Для работы этой частью в дополнительных параметрах запуска необходимо прописать параметр "/TESTMANAGER"/.

**Часть "Работа с объектами 1С"** позволяет взаимодействовать с обработками и отчетами, включая внешними, открывать формы нового и уже существующего объекта, создавать и заполнять объект, выполнять код.

Описание кейса записи работы пользователя и взаимодействия с объектами 1С можно посмотреть на [видео](https://youtu.be/ZsuN2km2mf8).
