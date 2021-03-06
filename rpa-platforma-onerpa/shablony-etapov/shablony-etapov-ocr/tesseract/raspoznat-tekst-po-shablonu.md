# Распознать текст по шаблону

![](<../../../../.gitbook/assets/Распознать текст по шаблону.png>)

Данный шаблон этапа для распознавания сканов множества однотипных документов с получением считанных структуры и данных из документов и возможностью дальнейшей обработки, в частности автоматического создания и записи в учетной программе (например 1с:бухгалтерия) документов. Основное преимущество этого шаблона в том, что создав 1 раз шаблон разметки для документа, вы можете распознать неограниченное количество сканов с аналогичной структурой. Соответствующее [видео с примером](https://www.youtube.com/watch?v=h40ZJWTR5R4). Шаблон содержит следующие параметры:

* Скан. Полный путь до файла сканированного документа. Этот параметр может задаваться динамически с предыдущего этапа, формирующего полный путь до файла. Например, шаблон этапа Файловая система - Текущий файл в каталоге.
* Шаблон разметки. Для удобства формирования шаблона, нажмите кнопку "Заполнить шаблон". Появится отдельная форма. Выберите файл картинки сканированного документа.   Скан откроется в верхней части формы. Далее в поле "Тип объекта" выберите тип документа в учетной программе, который нужно будет автоматически создать на основе распознанного скана. В нашем примере это Демо\_РТУ.  В нижней части формы есть вкладки Реквизиты и Табличные части. Заполним сначала вкладку Реквизиты. Сначала заполним реквизит "Контрагент". Для этого отметим на картинке прямоугольником точно те данные, которые хотим видеть внесенными для контрагента. Это может быть одно наименование. В нашем случае будет наименование и ИНН. Нажимаем кнопку "Перенести".  В графе "Код значения" может быть прописан код для обработки значения реквизита особым образом. Но обычно это не требуется.
* Якорь. Это текст, который всегда присутствует на документе и служащий для стопроцентного позиционирования скана, если документ плохо или криво отсканирован.&#x20;

![](<../../../../.gitbook/assets/Распознавание по шаблону - заполнение шаблона.png>)

Вкладка "Табличные части".  Обводим по очереди те реквизиты, которые мы хотим внести в документ. ВНИМАНИЕ! Первый реквизит будет ключевым, поэтому целесообразно выбирать не количество или сумму, а наименование или номенклатуру. выделение происходит аналогично как для реквизитов. Обводим прямоугольником и нажимаем кнопку "Перенести". Заполнив все реквизиты, обозначаем также конец таблицы - например слово ИТОГО  - якорь табличной части.&#x20;

После заполнения всех реквизитов, то есть когда весь шаблон для работы со сканом готов, нажимаем кнопку "Создать (тест)". Она позволяет посмотреть, как скан распознается и вносится в учетную программу.  Если тестовое создание прошло успешно, в левом верхнем углу нажимаем кнопку "ОК". Шаблон обработки скана записан.&#x20;

![](<../../../../.gitbook/assets/распознавание по шаблону - ТЧ.png>)

* Структура данных распознанного документа. Исходящий параметр. Распознанный документ с распознанной структурой и в том формате, который мы выбрали. В примере формат excel. Распознанные данные могут использоваться для дальнешей обработки в работе робота.
