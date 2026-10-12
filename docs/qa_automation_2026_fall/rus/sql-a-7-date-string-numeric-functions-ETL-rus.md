# [SQL-A] 7. (10/07) Date, String, Numeric Functions, Window Functions & ETL — Руководство по проведению ревью-сессии

**Методическое руководство для менторов и ревьюеров**  
**Курс:** SQL for QA Automation / QA Engineer  
**Тема занятия:** Встроенные функции (строковые, числовые, даты и времени), условная логика `CASE`, Оконные функции (Window Functions), процесс ETL (Extract, Transform, Load) и массовый импорт данных через `LOAD DATA LOCAL INFILE`, а также разбор домашнего задания по 6-му занятию (ClassicModels 10–15 & RealEstateDB).

---

## 🧭 1. Перспектива: Карта курса из 10 лекций

Напомните студентам их текущую позицию на карте курса. Это помогает систематизировать знания и показывает, как освоение функций трансформации данных, оконной аналитики и магистральных процессов ETL готовит фундамент для автоматизированного тестирования бэкенда, подготовки тестовых датасетов, очистки данных и валидации сквозных пайплайнов (End-to-End Data Pipelines).

* **Лекция 1:** Введение в реляционные БД (СУБД), таблицы, строки, столбцы, первичные и внешние ключи, целостность данных.
* **Лекция 2:** Простые запросы к одной таблице — выборка, фильтрация, сортировка, ограничение результатов (`LIMIT`) и метаданные.
* **Лекция 3/4:** Компоненты SQL — синтаксис и практика DDL (определение данных), DQL (запросы данных) и DML (манипулирование данными).
* **Лекция 5:** Объединение таблиц (`INNER JOIN`, `LEFT JOIN`, `SELF JOIN`, `FULL JOIN`).
* **Лекция 6:** Группировка, объединение множеств и подзапросы (`GROUP BY`, `HAVING`, `UNION / UNION ALL`, Subqueries, `CTE / WITH`).
* **Лекция 7 (Текущая):** Встроенные функции (Date, String, Numeric), логика `CASE`, Оконные функции (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG/LEAD`, `FIRST_VALUE/LAST_VALUE`) и ETL-процессы (`LOAD DATA LOCAL INFILE`).
* **Лекция 8:** Временные таблицы и представления (Views vs Temp Tables).
* **Лекция 9:** Концепции хранилищ данных (DWH, таблицы фактов и измерений, схемы Star/Snowflake).
* **Лекция 10:** Продвинутые темы (триггеры, хранимые функции, индексы) и подготовка к собеседованиям.

---

## 🕵️‍♂️ 2. Концепция сессии и QA-контекст тестирования

На 6-м занятии студенты научились группировать и соединять данные. На 7-м занятии мы переходим к **преобразованию, очистке, анализу временных рядов и загрузке данных**. В ежедневной практике QA-инженера функции работы со строками, датами, оконная аналитика и ETL-скрипты являются фундаментальными инструментами автоматизации тестирования и подготовки тестового окружения.

### Ключевые QA-цели при тестировании функций и ETL:
1. **Data Cleansing & Sanitization (Очистка и нормализация данных):** Применение `TRIM()`, `LOWER()`, `UPPER()`, `REPLACE()` для удаления невидимых пробелов, спецсимволов и некорректного регистра при сверке API и БД.
2. **Валидация бизнес-логики и интервалов (Date & Time Validation):** Расчет SLA, сроков доставки и жизненного цикла объектов с помощью `DATEDIFF()`, `ADDDATE()`, `DATE_FORMAT()`.
3. **Сложная оконная аналитика и поиск аномалий (Window Functions in QA):** Ранжирование клиентов (`DENSE_RANK`), поиск дубликатов (`ROW_NUMBER`), сегментация пользователей (`NTILE`), сравнение предыдущих и текущих состояний транзакций (`LAG / LEAD`).
4. **Валидация ETL-пайплайнов и массовой загрузки (Bulk Load Verification):** Настройка надежного импорта CSV-файлов через `LOAD DATA LOCAL INFILE` в противоположность ненадежным UI-мастерам (`Table Data Import Wizard`), теряющим строки при загрузке.
5. **Профилирование схемы и контейнеров (Schema Profiling):** Расчет максимальной длины текстовых полей через `MAX(LENGTH())` для оптимизации типов данных при создании тестовых таблиц и контейнеров.

---

## 🧱 3. Теоретический разбор: Домашнее задание 6-го занятия, встроенные функции, оконная аналитика и ETL

### 1. Разбор домашнего задания 6-го занятия

#### Часть 1: Множественные пути решения задач (Multiple Pathways)
На примере запроса *"Показать имена всех клиентов, чьи менеджеры работают в офисе San Francisco"*, ментор демонстрирует, что в SQL одна и та же задача тестирования может быть решена несколькими эквивалентными путями:
1. **Способ 1: Вложенные подзапросы (Nested Subqueries):**
   ```sql
   SELECT customerName 
   FROM customers 
   WHERE salesRepEmployeeNumber IN (
       SELECT employeeNumber 
       FROM employees 
       WHERE officeCode IN (
           SELECT officeCode FROM offices WHERE city = 'San Francisco'
       )
   );
   ```
2. **Способ 2: Подзапрос с JOIN:**
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
3. **Способ 3: Прямой INNER JOIN трех таблиц:**
   ```sql
   SELECT DISTINCT c.customerName 
   FROM customers c
   JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber
   JOIN offices o ON e.officeCode = o.officeCode
   WHERE o.city = 'San Francisco';
   ```
*💡 Урок для QA:* Все три пути возвращают ровно **12 записей**. Разные конструкции SQL приводят к одинаковому результату, что важно использовать при написании тестовых проверок и рефакторинге автотестов.

#### Часть 2: ClassicModels Database (Запросы 10–15)
* **Запрос 10 (Подсчет уникальных поставщиков и товаров):**
  `SELECT COUNT(DISTINCT productVendor), COUNT(DISTINCT productLine), COUNT(DISTINCT productCode) FROM products;` *(Результат: 7 продуктовых линеек, 110 товаров)*.
* **Запрос 11 (Средняя цена закупки по поставщику):**
  `SELECT productVendor, AVG(buyPrice) AS avg_buy_price FROM products GROUP BY productVendor;`
  *Правило ANSI SQL:* Любая неагрегированная колонка в `SELECT` обязана быть в `GROUP BY`.
* **Запрос 12 (Средний чек по клиенту):**
  Соединение `orders`, `orderdetails` и `products` с расчетом средних значений.
* **Запрос 13 (Самый продаваемый товар):**
  Уточнение бизнес-требований: продаваемый по количеству заказов (`COUNT(productCode)`) или по суммарному объему (`SUM(quantityOrdered)`)? Использование `ORDER BY ... DESC LIMIT 1`.
* **Запрос 14 (Расчет маржинальности MSRP vs BuyPrice):**
  `SELECT productCode, SUM((MSRP - buyPrice) * quantityOrdered) AS total_profit FROM orderdetails JOIN products USING (productCode) GROUP BY productCode ORDER BY total_profit DESC LIMIT 1;`

#### Часть 3: RealEstateDB
* **Запрос объявлений:** JOIN таблиц `listings`, `properties`, `agents` с объединением имени через `CONCAT(firstName, ' ', lastName)`.
* **Запрос показов и предложений:** JOIN 4–5 таблиц (`showings`, `offers`, `transactions`, `clients`, `agents`) для полной проверки статусов сделок с недвижимостью.

---

### 2. Встроенные функции SQL (String, Numeric, Date Functions)

#### A. Текстовые функции (String Functions)
* `CONCAT(str1, ' ', str2)` — объединение строк (например, сборка Full Name).
* `SUBSTRING(str, pos, len)` — извлечение подстроки с указанной позиции.
* `LEFT(str, n)` / `RIGHT(str, n)` — извлечение символов слева/справа.
* `TRIM(str)` — **фундаментальная функция очистки**. Удаляет пробелы по краям. Обязательна при обработке внешних CSV и подготовке данных.
* `LENGTH(str)` — возвращает длину строки в байтах/символах.
* `LOWER(str)` / `UPPER(str)` — приведение к нижнему/верхнему регистру.

#### B. Числовые функции (Numeric Functions)
* `ROUND(val, decimals)` — округление чисел до заданного знака.
* `FLOOR(val)` / `CEIL(val)` — округление вниз / вверх до целого.
* `ABS(val)` — абсолютное значение числа.
* `RAND()` — генерация случайного числа от 0 до 1 (для создания тестовых выборок).

#### C. Функции даты и времени (Date & Time Functions)
* `ADDDATE(date, INTERVAL n DAY)` — корректное добавление дней к дате (в отличие от синтаксиса `date + 5`, который может давать сбои в разных СУБД).
* `DATEDIFF(date1, date2)` — расчет разницы в днях между двумя датами (например, `DATEDIFF(shippedDate, orderDate)`).
* `DAY(date)`, `MONTH(date)`, `YEAR(date)` — извлечение компонентов даты.
* `NOW()`, `CURDATE()` — получение текущего времени и даты сервера.

#### D. Условная логика: выражения `CASE`
Позволяет создавать вычисляемые динамические колонки на лету:
```sql
SELECT customerName, country,
  CASE 
    WHEN country = 'USA' THEN 'US Customer'
    ELSE 'Non USA'
  END AS customer_type
FROM customers;
```
*Альтернатива через длину штата:* `CASE WHEN LENGTH(state) = 2 THEN 'USA' ELSE 'Non USA' END`.

---

### 3. Оконные функции (Window Functions) в деталях

Оконные функции выполняют вычисления по набору строк (окну), сохраняя при этом индивидуальную детализацию каждой строки (в отличие от `GROUP BY`, который схлопывает строки).

#### Ключевые оконные функции и их поведение:
1. **`ROW_NUMBER() OVER()`**: Простая сквозная нумерация строк от 1 до N (например, от 1 до 122 в таблице `customers`).
2. **`RANK() OVER(ORDER BY creditLimit DESC)` vs `DENSE_RANK() OVER(ORDER BY creditLimit DESC)`**:
   * **`RANK()`**: При совпадении значений присваивает одинаковый ранг, но **пропускает** последующие ранги (например: 20, 20, 22).
   * **`DENSE_RANK()`**: При совпадении значений присваивает одинаковый ранг и **НЕ пропускает** значения (например: 20, 20, 21).
3. **`NTILE(n) OVER()`**: Разбивает датасет на $N$ равных групп/квартилей.
   * `NTILE(4)` — разбиение 122 клиентов на 4 группы (Tier 1 — самые высокодоходные, Tier 4 — с наименьшим кредитным лимитом).
   * `NTILE(10)` — децильное разбиение на 10 частей.
4. **`LAG(col) OVER()` / `LEAD(col) OVER()`**:
   * `LAG()` — обращение к значению **предыдущей** строки (полезно для расчета разницы платежей или времени между транзакциями).
   * `LEAD()` — обращение к значению **следующей** строки.
5. **`CUME_DIST() OVER()`**: Расчет кумулятивного процентного распределения значений.
6. **`FIRST_VALUE(col)` / `LAST_VALUE(col)` и секционирование (`PARTITION BY`)**:
   * Позволяют получить первое и последнее значение внутри группы (например, `PARTITION BY country`), а также посчитать агрегаты (`AVG`, `MIN`, `MAX`) без сжатия строк:
   ```sql
   SELECT customerName, country, creditLimit,
     FIRST_VALUE(creditLimit) OVER(PARTITION BY country ORDER BY creditLimit DESC) AS max_in_country,
     AVG(creditLimit) OVER(PARTITION BY country) AS avg_in_country,
     DENSE_RANK() OVER(PARTITION BY country ORDER BY creditLimit DESC) AS rank_in_country
   FROM customers;
   ```

---

### 4. Магистральный ETL-процесс и загрузка данных

#### Проблема UI-инструмента `Table Data Import Wizard` (MySQL Workbench):
При попытке загрузить файл CSV с данными о съемках фильмов в Сан-Франциско (`SF Film Locations`, 1976 записей) через стандартный UI-мастер `Table Data Import Wizard`, импортируется всего **171 строка**. Вывод: UI-мастер ненадёжен для QA и Data Engineering из-за внутренних таймаутов и ошибок парсинга.

#### Надежный инженерный подход: `LOAD DATA LOCAL INFILE`
1. **Подготовка структуры таблицы (контейнера):**
   Предварительное профилирование длины столбцов `MAX(LENGTH(col))` и создание DDL-таблицы `film_locations`.
2. **Исполнение SQL-команды загрузки:**
   ```sql
   LOAD DATA LOCAL INFILE '/path/to/Film_Locations_in_San_Francisco.csv'
   INTO TABLE film_locations
   FIELDS TERMINATED BY ',' 
   ENCLOSED BY '"'
   LINES TERMINATED BY '
'
   IGNORE 1 ROWS
   (title, release_year, locations, fun_facts, production_company, distributor, director, writer, actor_1, actor_2, actor_3);
   ```
3. **Предварительная настройка окружения:**
   * Выполнение в MySQL: `SET GLOBAL local_infile = 1;`
   * Настройка подключения в MySQL Workbench: `Edit Connection -> Advanced -> Others -> OPT_LOCAL_INFILE=1`.

---

## 🔮 4. Суперсила QA и разбор вопросов студентов с лекции

### ❓ Вопрос 1: "В чем разница между `RANK()` и `DENSE_RANK()`, и когда что применять в QA?"
* **Ответ ментора:** `RANK()` пропускает номера при совпадении дублей (20, 20, 22), а `DENSE_RANK()` идет подряд (20, 20, 21). В QA-тестировании `DENSE_RANK()` применяют, когда нужно определить "Топ-3 категории пользователей" без дыр в нумерации. `RANK()` используют, когда ранг отражает реальную позицию в спортивном/конкурентном рейтинге.

### ❓ Вопрос 2: "Зачем нужен `TRIM()` при импорте данных через ETL?"
* **Ответ ментора:** При загрузке файлов CSV или работе с текстовыми полями API в строки часто попадают скрытые символы переноса строк (``, `
`) или неразрывные пробелы. Запрос `WHERE status = 'Active'` вернет `FALSE` для `'Active '`. Наложение `TRIM()` на этапе трансформации (ETL) предотвращает фантомные баги.

### ❓ Вопрос 3: "Почему UI Import Wizard загрузил 171 строку вместо 1976?"
* **Ответ ментора:** Графический мастер парсит CSV через интерпретатор Python/JS в Workbench, который спотыкается о кавычки, переносы строк внутри текстовых полей (`fun_facts`) и спецсимволы UTF-8. Прямой SQL-оператор `LOAD DATA LOCAL INFILE` работает на уровне ядра СУБД и корректно обрабатывает экранирование `ENCLOSED BY '"'`.

### ❓ Вопрос 4: "Как с помощью оконных функций найти дубликаты в базе данных?"
* **Ответ ментора:** С помощью комбинации `ROW_NUMBER() OVER(PARTITION BY email ORDER BY id)` в CTE. Если `row_num > 1`, значит перед нами дубликат записи.

---

## 💻 5. Практический скрипт: Live Queries & Hands-on Tasks

Таблица практических SQL-запросов для демонстрации на ревью-сессии:

| # | БД / Таблица | SQL Запрос / Операция | Целевая концепция | 💡 QA-контекст и сценарий проверки |
|---|---|---|---|---|
| **1** | ClassicModels | `SELECT customerName, CONCAT(contactFirstName, ' ', contactLastName) AS full_name, LENGTH(customerName) FROM customers;` | String Functions | Проверка форматирования ФИО и длины наименования для UI-полей. |
| **2** | ClassicModels | `SELECT DISTINCT TRIM(city) FROM customers;` | Data Cleansing (`TRIM`) | Санитаризация наименований городов от невидимых пробелов перед сверкой. |
| **3** | ClassicModels | `SELECT orderNumber, DATEDIFF(shippedDate, orderDate) AS delivery_days FROM orders WHERE status = 'Shipped';` | Date Functions (`DATEDIFF`) | Валидация соблюдения SLA по срокам доставки заказов. |
| **4** | ClassicModels | `SELECT orderNumber, ADDDATE(orderDate, INTERVAL 5 DAY) AS expected_ship FROM orders;` | Date Arithmetic (`ADDDATE`) | Расчет ожидаемой даты отгрузки по бизнес-правилу "+5 дней". |
| **5** | ClassicModels | `SELECT customerName, country, CASE WHEN country = 'USA' THEN 'US Customer' ELSE 'Non USA' END AS market FROM customers;` | Conditional Logic (`CASE`) | Тестирование динамической группировки пользователей по географическому рынку. |
| **6** | ClassicModels | `SELECT customerName, creditLimit, ROW_NUMBER() OVER(ORDER BY creditLimit DESC) AS row_num FROM customers;` | Window (`ROW_NUMBER`) | Простая сквозная нумерация датасета клиентов для пейджинга. |
| **7** | ClassicModels | `SELECT customerName, creditLimit, RANK() OVER(ORDER BY creditLimit DESC) AS rnk, DENSE_RANK() OVER(ORDER BY creditLimit DESC) AS drnk FROM customers;` | Window (`RANK` vs `DENSE_RANK`) | Сравнительный анализ поведения ранжирования при совпадении лимитов. |
| **8** | ClassicModels | `SELECT customerName, creditLimit, NTILE(4) OVER(ORDER BY creditLimit DESC) AS quartile FROM customers;` | Window (`NTILE`) | Сегментация клиентов на 4 квартили по доходности для таргетированных рассылок. |
| **9** | ClassicModels | `SELECT customerName, creditLimit, LAG(creditLimit) OVER(ORDER BY creditLimit DESC) AS prev_limit, LEAD(creditLimit) OVER(ORDER BY creditLimit DESC) AS next_limit FROM customers;` | Window (`LAG` / `LEAD`) | Анализ кредитных интервалов между соседними клиентами в рейтинге. |
| **10** | ClassicModels | `SELECT customerName, country, creditLimit, FIRST_VALUE(creditLimit) OVER(PARTITION BY country ORDER BY creditLimit DESC) AS max_country_limit FROM customers;` | Window (`PARTITION BY`) | Расчет максимального лимита внутри каждой страны без сжатия строк. |
| **11** | Film / SFGov | `SELECT MAX(LENGTH(title)), MAX(LENGTH(locations)), MAX(LENGTH(director)) FROM film_locations;` | Schema Profiling | Определение оптимальной длины `VARCHAR` при проектировании таблиц. |
| **12** | Film / SFGov | `LOAD DATA LOCAL INFILE '...' INTO TABLE film_locations FIELDS TERMINATED BY ',' ENCLOSED BY '"' IGNORE 1 ROWS;` | Bulk ETL Load | Полная и неискаженная загрузка 1976 строк CSV для интеграционного теста. |

---

## ⚠ 6. Типичные ошибки студентов и Live Troubleshooting

| Проблема / Симптом | Корень проблемы | Инструкция ментора по устранению |
|---|---|---|
| **Ошибка: `Loading local data is disabled; this must be enabled on both the client and server sides`** | Переменная `local_infile` выключена в MySQL сервере или клиенте. | Выполнить `SET GLOBAL local_infile = 1;` в SQL-клиенте и добавить `OPT_LOCAL_INFILE=1` в настройки соединения Workbench. |
| **Некорректная работа `LAST_VALUE()` (возвращает значение текущей строки)** | По умолчанию рамка окна (window frame) определена как `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. | Объяснить явное задание рамки: `LAST_VALUE(col) OVER(PARTITION BY ... ORDER BY ... RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)`. |
| **Потеря строк при использовании UI Import Wizard (171 строка из 1976)** | Сбой графического мастера из-за запятых и кавычек внутри полей CSV. | Продемонстрировать переход на скриптовый метод `LOAD DATA LOCAL INFILE`. |
| **Сбой функции `ADDDATE()` при передаче некорректной строки** | Несоответствие формата текстовой даты системному стандарту `YYYY-MM-DD`. | Применить `STR_TO_DATE(date_string, '%m/%d/%Y')` перед добавлением интервала. |

---

## ⏱ 7. Пошаговый таймплан ревью-сессии (50 минут)

### 📌 Фаза 1: Вводная часть и карта курса (5 мин)
* Приветствие, позиционирование Лекции 7 на карте 10 лекций курса.
* Обзор трех столпов занятия: встроенные функции, оконная аналитика, ETL-загрузка.

### 📌 Фаза 2: Разбор ДЗ занятия №6 (10 мин)
* Разбор трех способов решения задачи поиска клиентов в San Francisco (Subqueries vs JOINs).
* Проверка ключевых запросов ClassicModels 10–15 и структуры RealEstateDB.

### 📌 Фаза 3: Теоретический deep-dive и вопросы студентов (15 мин)
* Разбор работы `TRIM()`, `DATEDIFF()`, условной логики `CASE`.
* Наглядная демонстрация разницы `RANK()` vs `DENSE_RANK()`, `NTILE(4)`, `LAG/LEAD`.
* Объяснение причин сбоя `Table Data Import Wizard` и демонстрация `LOAD DATA LOCAL INFILE`.

### 📌 Фаза 4: Live Coding & Практика в MySQL Workbench (15 мин)
* Выполнение запросов профилирования схемы `MAX(LENGTH())`.
* Запуск комплексного запроса с оконными функциями по таблице `customers`.
* Выполнение импорта датасета `SF Film Locations` через `LOAD DATA LOCAL INFILE`.

### 📌 Фаза 5: Подведение итогов и анонс (5 мин)
* Резюме лучших практик подготовки данных для QA.
* Анонс Лекции 8: Временные таблицы (`Temp Tables`) и представления (`Views`).

---

## 🎯 8. Чек-лист ментора и Definition of Done (DoD)

По итогам ревью-сессии студент должен уметь:
* [ ] Применять `TRIM()`, `LOWER()`, `CONCAT()` для очистки и нормализации текстовых полей в тестах.
* [ ] Вычислять временные интервалы и сроки SLA с помощью `DATEDIFF()` и `ADDDATE()`.
* [ ] Понимать различие между `RANK()` и `DENSE_RANK()` и применять оконные функции для ранжирования.
* [ ] Строить запросы с `LAG()` / `LEAD()` для анализа изменений состояний данных во времени.
* [ ] Обосновать неприменимость UI-мастеров импорта для больших датасетов и конфигурировать `LOAD DATA LOCAL INFILE`.
* [ ] Самостоятельно составлять выражения `CASE` для формирования вычисляемых тестовых колонок.
