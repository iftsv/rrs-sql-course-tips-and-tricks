# [SQL-A] 6. (09/30) Group By, Union, Sub Queries — Руководство по проведению ревью-сессии
**Методическое руководство для менторов и ревьюеров**  
**Курс:** SQL for QA Automation / QA Engineer  
**Тема сессии:** Группировка данных (GROUP BY, HAVING), операции над множествами (UNION / UNION ALL), подзапросы (Subqueries), табличные выражения (CTE / WITH), разбор домашнего задания (ClassicModels 10–15, RealEstateDB) и интеграция с функциями / ETL-процессами.

---

## 🧭 1. Перспектива: Карта курса из 10 лекций

Напомните студентам их текущую позицию на карте курса. Это помогает систематизировать знания и демонстрирует, как освоение агрегации данных, объединения множеств, подзапросов и встроенных функций создает фундамент для автоматизации тестирования аналитических дашбордов, финансовой сверки (Data Reconciliation), процессов ETL и сложных проверок целостности баз данных.

*   **Лекция 1:** Введение в реляционные БД (RDBMS), таблицы, строки, колонки, Primary & Foreign Keys, целостность данных.
*   **Лекция 2:** Простые запросы к одной таблице — выборка, фильтрация (WHERE), сортировка (ORDER BY), ограничение (LIMIT).
*   **Лекция 3/4:** Компоненты SQL — синтаксис и практика DDL (Data Definition), DQL (Data Query), DML (Data Manipulation).
*   **Лекция 5:** Соединение таблиц (INNER JOIN, LEFT JOIN, SELF JOIN, FULL JOIN).
*   **Лекция 6 (Текущая ревью-сессия):** Группировка, объединения и подзапросы (GROUP BY, HAVING, UNION / UNION ALL, Subqueries, CTE).
*   **Лекция 7 (Прослушанная теория):** Встроенные функции (строковые, числовые, даты и времени, CASE, окно) и процессы ETL (`LOAD DATA LOCAL INFILE`).
*   **Лекция 8:** Временные таблицы и представления (Views vs Temp Tables).
*   **Лекция 9:** Концепции хранилищ данных (DWH, таблицы фактов и измерений, Star/Snowflake schema).
*   **Лекция 10:** Продвинутые темы (триггеры, хранимые функции) и подготовка к собеседованиям QA.

---

## 🕵️‍♂️ 2. Концепция сессии и QA-контекст

На прошлых занятиях студенты научились объединять таблицы через `JOIN`. В рамках 6-го и 7-го блоков мы переходим к анализу, агрегации, очистке и трансформации больших объемов данных. В ежедневной работе QA инженер использует группировки, подзапросы, встроенные функции и ETL-скрипты как базовые инструменты автоматизированной и ручной сверки данных (Data Reconciliation).

### Ключевые цели QA при тестировании GROUP BY, UNION, Subqueries, функций и ETL:
1.  **Валидация бизнес-метрик и KPI (Analytics & Reporting Testing):** Проверка агрегированных показателей на бэкенде (общая выручка, средний чек, количество активных клиентов по регионам, маржинальность) против показателей UI-дашбордов и выгрузок.
2.  **Сверка данных при миграциях и межсервисном объединении (Data Merging & Migration Validation):** Использование `UNION` и `UNION ALL` для валидации миграций данных, поиска потерянных записей и формирования дедуплицированных списков при объединении баз нескольких брендов/сервисов.
3.  **Тестирование пороговых и граничных значений (HAVING & Aggregates):** Выявление аномальных сегментов клиентов или транзакций (например, клиентов с суммарными платежами > $70,000) для проверки бизнес-логики программы лояльности.
4.  **Валидация встроенной трансформации данных (Data Cleansing & Transformation):** Использование функций `TRIM`, `CONCAT`, `CASE WHEN`, `DATEDIFF` для проверки корректности форматирования адресов, расчета возрастов и категорий клиентов в отчетах.
5.  **Тестирование надежности ETL-процессов (ETL & Ingestion Testing):** Сравнение встроенных мастеров импорта (`Table Data Import Wizard`) с высокопроизводительной загрузкой через `LOAD DATA LOCAL INFILE` при объемном тестировании (Performance & Load Testing) баз данных.

---

## 🧱 3. Теоретический разбор и разбор домашнего задания

Перед введением новых концепций ментор проводит разбор домашнего задания 6-го занятия, опираясь на разбор из 7-й лекции.

### Часть 1: Разбор домашнего задания ClassicModels (Запросы 10–15)

#### 1. Запрос 10: Подсчет количества вендоров, продуктовых линеек и продуктов
*   **Запрос:**
    ```sql
    SELECT 
        COUNT(DISTINCT productVendor) AS total_vendors,
        COUNT(DISTINCT productLine) AS total_lines,
        COUNT(DISTINCT productCode) AS total_products
    FROM products;
    ```
*   **Результат:** 7 вендоров, 7 продуктовых линеек, 110 продуктов.
*   **QA Tip:** Обратите внимание студентов на использование `COUNT(DISTINCT ...)`. Обычный `COUNT(...)` посчитает общие строки с дубликатами, что приведёт к ошибке при проверке уникальных сущностей.

#### 2. Запрос 11: Средняя цена закупка (Buy Price) по вендорам
*   **Запрос:**
    ```sql
    SELECT 
        productVendor, 
        AVG(buyPrice) AS avg_buy_price
    FROM products
    GROUP BY productVendor;
    ```
*   **💡 Критический разбор ошибки с лекции:** На лекции рассматривался пример, где в `SELECT` стоял `productName`, а в `GROUP BY` — `productCode`. Напомните студентам **золотое правило ANSI SQL**: *Любая неагрегированная колонка в SELECT должна обязательно присутствовать в GROUP BY*. Использование разных колонок может давать случайную строку из группы и ломать автотесты!

#### 3. Запрос 12: Средний чек/цена на клиента (Average Price per Customer)
*   **Запрос:**
    ```sql
    SELECT 
        c.customerNumber,
        c.customerName,
        AVG(p.amount) AS avg_payment_amount
    FROM customers c
    JOIN payments p ON c.customerNumber = p.customerNumber
    GROUP BY c.customerNumber, c.customerName;
    ```
*   **QA Context:** Требуется соединение таблиц `customers` и `payments` для оценки среднего чека оплаты каждого клиента.

#### 4. Запрос 13: Самый продаваемый продукт (Product Sold the Most)
*   **Вопрос с подвохом:** Что значит "продавался больше всего"? По количеству уникальных заказов (`COUNT(orderNumber)`) или по суммарному количеству штук (`SUM(quantityOrdered)`)?
*   **Вариант по количеству позиций в заказах:**
    ```sql
    SELECT 
        p.productCode,
        p.productName,
        COUNT(od.orderNumber) AS times_ordered
    FROM products p
    JOIN orderdetails od ON p.productCode = od.productCode
    GROUP BY p.productCode, p.productName
    ORDER BY times_ordered DESC
    LIMIT 1;
    ```
*   **QA Tip:** В реальных ТЗ QA обязан уточнять у аналитика/Product Owner бизнес-смысл фразы "sold the most".

#### 5. Запрос 14: Расчет прибыли/маржи (MSRP vs Buy Price)
*   **Запрос:**
    ```sql
    SELECT 
        p.productCode,
        p.productName,
        SUM((p.MSRP - p.buyPrice) * od.quantityOrdered) AS total_potential_margin
    FROM products p
    JOIN orderdetails od ON p.productCode = od.productCode
    GROUP BY p.productCode, p.productName
    ORDER BY total_potential_margin DESC
    LIMIT 1;
    ```
*   **QA Context:** Валидация расчетных финансовых полей бэкенда на соответствие формуле `(MSRP - BuyPrice) * Quantity`.

---

### Часть 2: Разбор домашнего задания RealEstateDB

1.  **Все листинги с адресом, ценой, агентом и статусом:**
    ```sql
    SELECT 
        l.listing_id,
        p.address,
        l.price,
        CONCAT(a.first_name, ' ', a.last_name) AS agent_full_name,
        l.status
    FROM listings l
    JOIN properties p ON l.property_id = p.property_id
    JOIN agents a ON l.agent_id = a.agent_id;
    ```
2.  **Все показы (Showings) с именем клиента и адресом:**
    *   Соединение 4 таблиц: `showings` + `listings` + `properties` + `clients`.
    *   Использование `CONCAT(c.first_name, ' ', c.last_name)` для формирования ФИО.
3.  **Завершенные транзакции (Complete Transactions):**
    *   Сборка данных из 5 таблиц (`transactions`, `offers`, `listings`, `properties`, `clients`) для полного сквозного E2E-аудита покупки недвижимости.

---

### Часть 3: Подзапросы (Subqueries) — Различные пути решения одной задачи

На лекции подробно разбиралась задача: *"Показать имена всех клиентов, у которых менеджеры (employees) работают в офисе San Francisco"*.

*   **Путь 1: Вложенные подзапросы (2 уровня Subquery)**
    ```sql
    SELECT customerName 
    FROM customers 
    WHERE salesRepEmployeeNumber IN (
        SELECT employeeNumber 
        FROM employees 
        WHERE officeCode IN (
            SELECT officeCode 
            FROM offices 
            WHERE city = 'San Francisco'
        )
    );
    ```
*   **Путь 2: Подзапрос с JOIN внутри**
    ```sql
    SELECT customerName 
    FROM customers 
    WHERE salesRepEmployeeNumber IN (
        SELECT e.employeeNumber 
        FROM employees e
        JOIN offices o ON e.officeCode = o.officeCode
        WHERE o.city = 'San Francisco'
    );
    ```
*   **Путь 3: Прямой INNER JOIN без подзапросов**
    ```sql
    SELECT DISTINCT c.customerName 
    FROM customers c
    JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber
    JOIN offices o ON e.officeCode = o.officeCode
    WHERE o.city = 'San Francisco';
    ```
*   **💡 QA Вывод ментора:** Все три пути дают ровно **12 записей**. Это демонстрирует студентам, что в SQL одну и ту же задачу тестирования можно решить разным синтаксисом. Однако для автотестов более предпочтителен путь 2 или 3 из-за лучшей читаемости и производительности.

---

## 🔮 4. Суперсила QA и разбор вопросов студентов с теории

Ключевые вопросы и ошибки студентов, зафиксированные в аудиозаписи 7-й лекции:

### ❓ Вопрос 1: «Почему при объединении UNION получилась 101 запись, а ожидалось 100?»
*   **Разбор ментора:** `UNION` удаляет дубликаты, сравнивая **абсолютно все колонки** в строке выборки. Если в запрос добавляется синтетическая текстовая метка источника (например, `'ClassicModels' AS DB` vs `'MyWork' AS DB`), одна и та же городская запись `San Francisco` в сочетании с разными метками становится двумя УНИКАЛЬНЫМИ строками. Появление 101-й строки — это результат работы метки БД, превратившей совпавший дубликат в уникальную строку!

### ❓ Вопрос 2: «Зачем использовать TRIM при ETL и заполнении баз данных?»
*   **Разбор ментора:** При загрузке данных из внешних CSV/JSON файлов в поля частые "невидимые" символы пробелов или переносов строк ломают поиска `WHERE name = 'John'`. Функция `TRIM()` — главный инструмент QA при Data Cleansing. При сверке данных в автотестах рекомендуется обворачивать текстовые поля в `TRIM(column)`.

### ❓ Вопрос 3: «В чем разница между `Table Data Import Wizard` и `LOAD DATA LOCAL INFILE`?»
*   **Разбор ментора:** 
    *   `Wizard` (мастер импорта Workbench) удобен для быстрой работы, но крайне медленный и **ненадежный**: из файла на 1976 строк он может молча загрузить только 171 строку из-за ошибок парсинга типы данных или кавычек.
    *   `LOAD DATA LOCAL INFILE` — промышленный SQL-команда прямого импорта, за секунды загружающая сотни тысяч строк. В QA для нагрузочного тестирования и подготовки тестовых стендов следует использовать именно `LOAD DATA LOCAL INFILE`.

### ❓ Вопрос 4: «Зачем нужны оконные функции (ROW_NUMBER, RANK, DENSE_RANK, LAG/LEAD) QA-инженеру?»
*   **Разбор ментора:**
    *   `ROW_NUMBER()` / `DENSE_RANK()` позволяют быстро найти дубликаты записей или выявить N самых дорогих заказов без схлопывания строк (в отличие от `GROUP BY`).
    *   `LAG()` / `LEAD()` позволяют сравнивать значение текущей строки с предыдущей или следующей (например, проверка изменения баланса или статуса транзакции во времени).

---

## 💻 5. Практический скрипт: Live Queries & Hands-on Tasks

Таблица практических SQL-запросов для демонстрации на ревью-сессии:

| # | БД | SQL Запрос / Операция | Целевая концепция | 💡 QA Контекст / Сценарий тестирования |
|---|---|---|---|---|
| **1** | ClassicModels | `SELECT COUNT(*), COUNT(DISTINCT country), MAX(creditLimit) FROM customers;` | Baseline Data Profiling | Первичный аудит таблицы перед написанием автотестов. |
| **2** | ClassicModels | `SELECT country, COUNT(*) FROM customers GROUP BY country;` | GROUP BY | Проверка распределения пользователей по регионам. |
| **3** | ClassicModels | `SELECT customerNumber FROM payments GROUP BY customerNumber HAVING SUM(amount) > 70000;` | GROUP BY + HAVING | Поиск VIP-клиентов, превысивших порог $70,000 для программы лояльности. |
| **4** | ClassicModels | `SELECT city, 'ClassicModels' AS DB FROM customers UNION ALL SELECT city, 'Department' AS DB FROM department;` | UNION ALL (с дубликатами) | Валидация полного объединения записей из двух микросервисов. |
| **5** | ClassicModels | `SELECT city FROM customers UNION SELECT city FROM department;` | UNION (дедупликация) | Формирование единого справочника городов без дублей. |
| **6** | ClassicModels | `SELECT customerName FROM customers WHERE salesRepEmployeeNumber IN (SELECT employeeNumber FROM employees WHERE officeCode IN (SELECT officeCode FROM offices WHERE city = 'San Francisco'));` | Subquery в WHERE | Выборка тестового датасета клиентов для E2E сценария. |
| **7** | ClassicModels | `WITH CTE_VIP AS (SELECT customerNumber, SUM(amount) AS total FROM payments GROUP BY customerNumber HAVING total > 70000) SELECT c.*, v.total FROM customers c JOIN CTE_VIP v ON c.customerNumber = v.customerNumber;` | CTE (WITH ... AS) | Модульный и читаемый запрос для автоматизации аудита VIP-клиентов. |
| **8** | ClassicModels | `SELECT customerName, TRIM(addressLine1), UPPER(city), CONCAT(contactFirstName, ' ', contactLastName) FROM customers;` | Text Functions (TRIM, UPPER, CONCAT) | Валидация форматирования и очистки текстовых полей при репортинг-тестах. |
| **9** | ClassicModels | `SELECT orderNumber, DATEDIFF(shippedDate, orderDate) AS processing_days FROM orders WHERE shippedDate IS NOT NULL;` | Date Functions (DATEDIFF) | Проверка соблюдения SLA по срокам доставки заказов. |
| **10** | ClassicModels | `SELECT customerName, creditLimit, CASE WHEN creditLimit > 100000 THEN 'Platinum' WHEN creditLimit > 50000 THEN 'Gold' ELSE 'Standard' END AS tier FROM customers;` | Advanced CASE Statement | Тестирование бизнес-логики категоризации клиентов. |
| **11** | Film / ETL | `LOAD DATA LOCAL INFILE '/path/films.csv' INTO TABLE film_locations FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '
' IGNORE 1 ROWS;` | ETL Direct Ingestion | Загрузка и проверка массового тестового датасета без потерь записей. |

---

## ⚠ 6. Типичные ошибки студентов и Live Troubleshooting

| Ошибка / Симптом | Корень проблемы | Как ментор объясняет решение |
|---|---|---|
| **Error Code: 1111. Invalid use of group function** | Использование агрегатных функций (`SUM`, `COUNT`, `AVG`) внутри `WHERE`. | Объяснить порядок выполнения SQL: `FROM` ➔ `WHERE` ➔ `GROUP BY` ➔ `HAVING` ➔ `SELECT`. В момент `WHERE` строки ещё не сгруппированы. Фильтр агрегатов делать через `HAVING`. |
| **Error Code: 1248. Every derived table must have its own alias** | Вложенный подзапрос в блоке `FROM` не имеет псевдонима (алиаса). | Показать обязательное добавление алиаса в конце скобок подзапроса: `FROM (SELECT ...) AS sub_table`. |
| **Column count mismatch in UNION** | Разное количество колонок или несопоставимые типы данных в блоках `UNION`. | Напомнить базовое правило `UNION`: количество и порядок столбцов во всех `SELECT` обязаны совпадать. |
| **Несоответствие неагрегированных колонок в SELECT и GROUP BY** | Выбор столбцов в `SELECT`, которые отсутствуют в `GROUP BY` и не обернуты в агрегат. | Разъяснить правило ANSI SQL: любые неагрегированные столбцы из `SELECT` должны быть явно указаны в `GROUP BY`. |
| **Error Code: 3948. Loading local data is disabled** | Попытка выполнить `LOAD DATA LOCAL INFILE`, когда параметр отключен на сервере/клиенте. | Выполнить команду `SET GLOBAL local_infile = 1;` и прописать `OPT_LOCAL_INFILE=1` в настройках подключения Workbench. |

---

## ⏱ 7. Пошаговый таймплан ревью-сессии (50 минут)

### 📌 Фаза 1: Вводная часть и Карта курса (5 мин)
*   Приветствие, фиксация положения Занятий 6 и 7 на карте курса из 10 лекций.
*   Подчеркнуть важность агрегаций, подзапросов и ETL для автоматизации тестирования и сверки данных.

### 📌 Фаза 2: Разбор домашнего задания (10 мин)
*   Разбор ключевых задач из ClassicModels (10–15): ошибки группировки по продуктам, расчет маржи, `COUNT(DISTINCT)`.
*   Быстрый обзор решений RealEstateDB (соединение 4–5 таблиц для проверок показов и транзакций).

### 📌 Фаза 3: Теоретический deep-dive и вопросы студентов (15 мин)
*   Разбор вопросов из 7-й лекции: почему `UNION` вернул 101 запись, разница `WHERE` vs `HAVING`, почему `Table Data Import Wizard` потерял данные.
*   Объяснение применения `CASE WHEN`, `TRIM()`, `DATEDIFF()` и оконных функций в повседневных задачах QA.

### 📌 Фаза 4: Практический Live Coding (15 мин)
*   Демонстрация `GROUP BY` + `HAVING SUM(amount) > 70000` на таблице `payments`.
*   Построение эквивалентных запросов через Subquery в `WHERE` и CTE (`WITH`).
*   Демонстрация фильтрации текстовых и временных полей с помощью `TRIM`, `DATEDIFF` и `CASE`.

### 📌 Фаза 5: Wrap-up и анонс следующего шага (5 мин)
*   Подведение итогов: закрепление правил группировки, подзапросов и загрузки данных.
*   Preview Лекции 8: Временные таблицы и представления (Views vs Temp Tables).

---

## 🎯 8. Чек-лист ментора и Definition of Done (DoD)

К концу ревью-сессии студенты должны уметь:
*   [ ] Проводить первичный профилирующий аудит незнакомых таблиц с помощью `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
*   [ ] Понимать и объяснять различие между фильтрацией строк (`WHERE`) и фильтрацией агрегатов (`HAVING`).
*   [ ] Грамотно применять `GROUP BY`, соблюдая правило совпадения столбцов из `SELECT`.
*   [ ] Отличать поведение `UNION` (с дедупликацией) от `UNION ALL` (сохранение всех строк) в контексте миграционного тестирования.
*   [ ] Конструировать подзапросы с оператором `IN` и создавать читаемые модульные CTE-выражения (`WITH`).
*   [ ] Применять встроенные функции (`TRIM`, `CONCAT`, `DATEDIFF`, `CASE`) для подготовки и проверки тестовых данных.
*   [ ] Понимать принцип работы ETL-импорта через `LOAD DATA LOCAL INFILE` и его преимущества перед стандартным UI-мастером.
