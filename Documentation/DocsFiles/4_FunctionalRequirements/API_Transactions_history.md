# **API\_Получение истории транзакций**
## **История изменений**

|**Автор**|**Версия изменений**|**Описание**|**Задача**|
| --- | --- | --- | --- |
|Вознесенский М.А.|1.0|Draft|Исходная версия документа|TASK-1|
**Цель:** сервис предназначен для получения списка транзакций по выбранной карте.

**URL:** [endpoint]/api/cards/transactions

**Метод:** GET

**Доступ:** авторизованная зона. Сервис вызывается с фронта.

**Входные параметры:**

|**Наименование параметра**|**Тип**|**Формат**|**Описание**|**Обязательность**|
| --- | --- | --- | --- | --- |
|cardnumber|String| |Виртуальный или полный номер карты|Да|
|begindate|String|yyyy-mm-dd|Дата начала периода за который запрашивается информация|Да|
|enddate|String|yyyy-mm-dd|Дата окончания периода за который запрашивается информация|Да|
|NumberOfOps|Number| |Максимальное количество операций, которое нужно вернуть в ответ|Нет|
|Cardid|Number| |ИД карты, используется как альтернатива cardnumber|Нет|
**Выходные параметры:**

|**Наименование поля**|**Тип**|**Описание**
| --- | --- | --- |
|cards/transactions|List[Object]|Список транзакций|
|cards/transactions/transaction\_date|DateTime|Дата операции|
|cards/transactions/auth\_date|Date|Дата операционного дня проведения расчетов по операции|
|cards/transactions/description|String|Назначение операции|
|cards/transactions/authorization\_code|Number|Код авторизации. Код, присвоенный Участником, разрешившим совершение операции.|
|cards/transactions/auth\_amount|Number|Сумма операции в валюте транзакции|
|cards/transactions/auth\_currency|String|Валюта операции|
|cards/transactions/card\_sign|String|Знак операции, принимает значения 'Debit' или 'Credit'|
|cards/transactions/transaction\_amount|Number|Сумма операции в валюте карты|
|cards/transactions/transaction\_currency|String|Валюта карты|
|cards/transactions/mcc|Number|Код идентифицирующий категорию (бизнес-код) точки сервиса|
|cards/transactions/place|String|Параметры пункта обслуживания (название и город).|
|cards/transactions/oper\_type|String|Тип операции, IBN - оплата услуг в ИБ, POS - POS, ATM - наличка, BAL - платный запрос баланса карты, P2P - P2P, KOM - комиссия, KAS - наличка, ACC - перевод по счету.|
|cards/transactions/operation\_status|String|Статус операции|
**Пример запроса/ответа:**
## **Основной сценарий (MC)**
1. Сервис инициирует получение актуального списка транзакций по выбранной карте от АБС AllAboutCards с параметром Cardid;
1. По событию получения ответа сервис сохраняет транзакции в БД в таблицу card\_transactions, согласно маппингу:

|**Поле в таблице card\_transactions**|**Поле в ответе AllAboutCards**|
| --- | --- |
|id|Генерируемое автоматически значение|
|Cardid|Текущий ИД клиента в ДБО|
|transaction\_date|Значение transaction\_date в ответе AllAboutCards в формате yyyy-mm-dd|
|auth\_date|Значение auth\_date в ответе AllAboutCards в формате yyyy-mm-dd|
|description|Значение description в ответе AllAboutCards|
|authorization\_code|Значение authorization\_code в ответе AllAboutCards|
|auth\_amount|Значение auth\_amount в ответе AllAboutCards|
|auth\_currency|<p>Значение ₽ если в auth\_currency в ответе AllAboutCards получено значение ‘RUB’</p><p>Значение ‘**$**’ если в auth\_currency в ответе AllAboutCards получено значение ‘USD’</p><p>Значение ‘**€**’ если в auth\_currency в ответе AllAboutCards получено значение ‘EUR’</p>|
|card\_sign|Значение card\_sign в ответе AllAboutCards|
|transaction\_amount|Значение transaction\_amount в ответе AllAboutCards|
|transaction\_currency|Значение transaction\_currency в ответе AllAboutCards|
|mcc|Значение mcc в ответе AllAboutCards|
|place|Значение place в ответе AllAboutCards|
|oper\_type|<p>Значение «оплата услуг в ИБ», если в oper\_type в ответе AllAboutCards получено значение «IBN»</p><p>Значение «POS», если в oper\_type в ответе AllAboutCards получено значение «POS»</p><p>Значение «наличка», если в oper\_type в ответе AllAboutCards получено значение «ATM»</p><p>Значение «платный запрос баланса карты», если в oper\_type в ответе AllAboutCards получено значение «BAL»</p><p>Значение «P2P», если в oper\_type в ответе AllAboutCards получено значение «P2P»</p><p>Значение «KOM», если в oper\_type в ответе AllAboutCards получено значение «комиссия»</p><p>Значение «KAS», если в oper\_type в ответе AllAboutCards получено значение «наличка»</p><p>Значение «перевод по счету», если в oper\_type в ответе AllAboutCards получено значение «ACC» </p>|
|operation\_status|Значение operation\_status в ответе AllAboutCards|
1. Сервис выполняет поиск транзакций для выбранной карты в БД в таблице cards\_transactions используя входные параметры begindate и enddate.
1. Если в таблице cards\_transactions для текущей карты отсутствуют транзакции по заданным параметрам, то выполняется переход к сценарию EC-1 Отсутствие транзакций.
1. Сервис сортируеттранзакции в порядке уменьшения значения transaction\_date.
1. Сервис формирует ответ согласно маппингу:

|**Наименование поля**|**Описание**|
| --- | --- |
|cards/transactions/transaction\_date|Значение card\_transactions.transaction\_date  в формате yyyy-mm-dd|
|cards/transactions/auth\_date|Значение card\_transactions.auth\_date  в формате yyyy-mm-dd|
|cards/transactions/description|Значение card\_transactions.description |
|cards/transactions/authorization\_code|Значение card\_transactions.authorization\_code |
|cards/transactions/auth\_amount|Значение card\_transactions.auth\_amount |
|cards/transactions/auth\_currency|<p>Значение «₽», если в card\_transactions.auth\_currency  получено значение «RUB»</p><p>Значение «**$**», если в card\_transactions.auth\_currency получено значение «USD»</p><p>Значение «**€»** если в card\_transactions.auth\_currency  получено значение «EUR»</p>|
|cards/transactions/card\_sign|Значение card\_transactions.card\_sign |
|cards/transactions/transaction\_amount|Значение card\_transactions.transaction\_amount |
|cards/transactions/transaction\_currency|Значение card\_transactions.transaction\_currency |
|cards/transactions/mcc|Значение card\_transactions.mcc |
|cards/transactions/place|Значение card\_transactions.place |
|cards/transactions/oper\_type|<p>Значение «оплата услуг в ИБ», если в card\_transactions.oper\_type  получено значение «IBN»</p><p>Значение «POS», если в card\_transactions.oper\_type  получено значение «POS»</p><p>Значение «наличка», если в card\_transactions.oper\_type  получено значение «ATM»</p><p>Значение «платный запрос баланса карты», если в card\_transactions.oper\_type  получено значение «BAL»</p><p>Значение «P2P», если в card\_transactions.oper\_type  получено значение «P2P»</p><p>Значение «KOM», если в card\_transactions.oper\_type  получено значение «комиссия»</p><p>Значение «KAS», если в card\_transactions.oper\_type  получено значение «наличка»</p><p>Значение «перевод по счету», если в card\_transactions.oper\_type  получено значение «ACC»</p>|
|cards/transactions/operation\_status|Значение card\_transactions.operation\_status |
1. Сервис возвращает ответ вызывающей системе.
## **Исключительные сценарии (EC)**
EC-1 Отсутствие транзакций

1. Сервис формирует ответ, согласно маппингу.

|**Параметр ответа**|**Правило формирования**|
| --- | --- |
|data/error/code|Код ошибки|
|data/error/text|"За указанный период не найдено ни одной транзакции. Попробуйте изменить период поиска или выбрать другие карты"|
1. API возвращает ответ.
## **Конфигурационные параметры**

|**Название конфига**|**Значение**|**Описание**|
| --- | --- | --- |
| | | |



