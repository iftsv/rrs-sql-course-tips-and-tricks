# [SQL-A] 7. (10/07) Date, String, Numeric Functions, Window Functions & ETL — Review Session Guide

**Pedagogical Guide for Mentors and Reviewers**  
**Course:** SQL for QA Automation / QA Engineer  
**Session Topic:** Built-in Functions (String, Numeric, Date & Time), Conditional Logic `CASE`, Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG/LEAD`, `FIRST_VALUE/LAST_VALUE`), ETL Process (Extract, Transform, Load), Bulk Data Import via `LOAD DATA LOCAL INFILE`, and Homework Review for Lesson 6 (ClassicModels 10–15 & RealEstateDB).

---

## 🧭 1. Perspective: 10-Lecture Curriculum Map

Remind students of their current position on the 10-lecture course curriculum map. This helps summarize knowledge and illustrates how mastering data transformation functions, window analytics, and pipeline ETL processes establishes the foundation for automated backend testing, test data generation, data cleansing, and validating End-to-End Data Pipelines.

* **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary and Foreign Keys, Data Integrity.
* **Lecture 2:** Simple Queries on a Single Table — Data Selection, Filtering, Sorting, Result Limiting (`LIMIT`), and Metadata.
* **Lecture 3/4:** SQL Components — Syntax & Practice of DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation).
* **Lecture 5:** Table Joins (`INNER JOIN`, `LEFT JOIN`, `SELF JOIN`, `FULL JOIN`).
* **Lecture 6:** Grouping, Set Operations & Subqueries (`GROUP BY`, `HAVING`, `UNION / UNION ALL`, Subqueries, `CTE / WITH`).
* **Lecture 7 (Current):** Built-in Functions (Date, String, Numeric), `CASE` Logic, Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG/LEAD`, `FIRST_VALUE/LAST_VALUE`), and ETL Pipeline (`LOAD DATA LOCAL INFILE`).
* **Lecture 8:** Temporary Tables and Views (Views vs Temp Tables).
* **Lecture 9:** Data Warehouse Concepts (DWH, Fact and Dimension Tables, Star/Snowflake Schemas).
* **Lecture 10:** Advanced Topics (Triggers, Stored Functions, Indexes) and Interview Preparation.

---

## 🕵️‍♂️ 2. Session Concept & QA Testing Context

In Lesson 6, students learned data grouping and set unification. In Lesson 7, we advance to **data transformation, cleansing, time-series analysis, and bulk data loading**. In daily QA practice, string/date functions, window analytics, and ETL scripts serve as primary tools for test automation and test environment setup.

### Core QA Objectives when Testing Functions, Windowing & ETL:
1. **Data Cleansing & Sanitization:** Applying `TRIM()`, `LOWER()`, `UPPER()`, and `REPLACE()` to eliminate invisible trailing spaces, control characters, and case mismatches between API responses and DB records.
2. **Business SLA & Date Interval Validation:** Calculating delivery windows, SLAs, and object lifecycles using `DATEDIFF()`, `ADDDATE()`, and `DATE_FORMAT()`.
3. **Advanced Window Analytics & Anomaly Detection:** Customer ranking (`DENSE_RANK`), duplicate identification (`ROW_NUMBER`), user segmentation (`NTILE`), and state transition checks (`LAG / LEAD`).
4. **ETL Pipeline & Bulk Import Validation:** Setting up reliable CSV imports via `LOAD DATA LOCAL INFILE` in contrast to unreliable UI wizards (`Table Data Import Wizard`) that drop rows during import.
5. **Schema & Container Profiling:** Calculating maximum string lengths via `MAX(LENGTH())` to optimize database column data types when building test containers.

---

## 🧱 3. Theoretical Deep Dive: Lesson 6 Homework, Built-in Functions, Window Functions & ETL

### 1. Homework Review (Lesson 6 Homework)

#### Part 1: Multiple Query Pathways
Using the problem *"Find customer names whose sales representatives work in the San Francisco office"*, the mentor demonstrates that SQL problems can be solved through multiple equivalent technical approaches:
1. **Pathway 1: Nested Subqueries:**
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
2. **Pathway 2: Subquery with JOIN:**
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
3. **Pathway 3: Direct 3-Table INNER JOIN:**
   ```sql
   SELECT DISTINCT c.customerName 
   FROM customers c
   JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber
   JOIN offices o ON e.officeCode = o.officeCode
   WHERE o.city = 'San Francisco';
   ```
*💡 QA Takeaway:* All three queries yield exactly **12 records**. Different SQL constructs achieve identical results, an essential insight for writing verification assertions and refactoring test suites.

#### Part 2: ClassicModels Database (Queries 10–15)
* **Query 10 (Counting Unique Entities):**
  `SELECT COUNT(DISTINCT productVendor), COUNT(DISTINCT productLine), COUNT(DISTINCT productCode) FROM products;` *(Result: 7 product lines, 110 products)*.
* **Query 11 (Average Buy Price per Vendor):**
  `SELECT productVendor, AVG(buyPrice) AS avg_buy_price FROM products GROUP BY productVendor;`
  *ANSI SQL Rule:* Any non-aggregated column in `SELECT` must be declared in `GROUP BY`.
* **Query 12 (Average Customer Order Value):**
  Joining `orders`, `orderdetails`, and `products` with aggregate calculations.
* **Query 13 (Top Sold Product):**
  Clarifying requirements: top sold by order count (`COUNT(productCode)`) or by volume (`SUM(quantityOrdered)`)? Using `ORDER BY ... DESC LIMIT 1`.
* **Query 14 (Margin Calculation MSRP vs Buy Price):**
  `SELECT productCode, SUM((MSRP - buyPrice) * quantityOrdered) AS total_profit FROM orderdetails JOIN products USING (productCode) GROUP BY productCode ORDER BY total_profit DESC LIMIT 1;`

#### Part 3: RealEstateDB
* **Listings Query:** Multi-table JOIN (`listings`, `properties`, `agents`) with name formatting via `CONCAT(firstName, ' ', lastName)`.
* **Showings & Transactions Query:** Joining 4–5 tables (`showings`, `offers`, `transactions`, `clients`, `agents`) for end-to-end property deal verification.

---

### 2. Built-in SQL Functions (String, Numeric, Date Functions)

#### A. Text / String Functions
* `CONCAT(str1, ' ', str2)` — String concatenation (e.g. assembling Full Name).
* `SUBSTRING(str, pos, len)` — Extracting sub-string from specified position.
* `LEFT(str, n)` / `RIGHT(str, n)` — Extracting characters from left/right side.
* `TRIM(str)` — **Essential data cleansing function**. Removes leading/trailing whitespace. Critical when ingesting external CSVs.
* `LENGTH(str)` — Returns string length in characters/bytes.
* `LOWER(str)` / `UPPER(str)` — Case conversion.

#### B. Numeric Functions
* `ROUND(val, decimals)` — Rounding numbers to specified decimal places.
* `FLOOR(val)` / `CEIL(val)` — Rounding down / up to whole integers.
* `ABS(val)` — Absolute value.
* `RAND()` — Random number generation between 0 and 1 (useful for test data generation).

#### C. Date & Time Functions
* `ADDDATE(date, INTERVAL n DAY)` — Standard date addition (preferred over non-standard `date + 5`).
* `DATEDIFF(date1, date2)` — Difference in days between two dates (`DATEDIFF(shippedDate, orderDate)`).
* `DAY(date)`, `MONTH(date)`, `YEAR(date)` — Extracting date components.
* `NOW()`, `CURDATE()` — Retrieving current server timestamp and date.

#### D. Conditional Logic: `CASE` Statements
Enables creation of dynamic computed columns:
```sql
SELECT customerName, country,
  CASE 
    WHEN country = 'USA' THEN 'US Customer'
    ELSE 'Non USA'
  END AS customer_type
FROM customers;
```
*Alternative via state length:* `CASE WHEN LENGTH(state) = 2 THEN 'USA' ELSE 'Non USA' END`.

---

### 3. Window Functions Deep Dive

Window functions perform calculations across a set of table rows while preserving individual row identity (unlike `GROUP BY`, which collapses rows).

#### Core Window Functions and Behaviors:
1. **`ROW_NUMBER() OVER()`**: Sequential row numbering from 1 to N (1 to 122 in `customers`).
2. **`RANK() OVER(ORDER BY creditLimit DESC)` vs `DENSE_RANK() OVER(ORDER BY creditLimit DESC)`**:
   * **`RANK()`**: Assigns equal rank to tied values and **skips** subsequent rank numbers (e.g., 20, 20, 22).
   * **`DENSE_RANK()`**: Assigns equal rank to tied values and **does NOT skip** rank numbers (e.g., 20, 20, 21).
3. **`NTILE(n) OVER()`**: Divides dataset into $N$ equal partitions/quartiles.
   * `NTILE(4)` — Splits 122 customers into 4 quartiles (Tier 1 top valuable to Tier 4 lowest limit).
   * `NTILE(10)` — Decile partitioning into 10 groups.
4. **`LAG(col) OVER()` / `LEAD(col) OVER()`**:
   * `LAG()` — Accesses value from the **previous** row (useful for calculating delta between consecutive transactions).
   * `LEAD()` — Accesses value from the **next** row.
5. **`CUME_DIST() OVER()`**: Cumulative percentage distribution calculation.
6. **`FIRST_VALUE(col)` / `LAST_VALUE(col)` and Partitioning (`PARTITION BY`)**:
   * Retrieves boundary values per group (`PARTITION BY country`) alongside window aggregates (`AVG`, `MIN`, `MAX`) without collapsing rows:
   ```sql
   SELECT customerName, country, creditLimit,
     FIRST_VALUE(creditLimit) OVER(PARTITION BY country ORDER BY creditLimit DESC) AS max_in_country,
     AVG(creditLimit) OVER(PARTITION BY country) AS avg_in_country,
     DENSE_RANK() OVER(PARTITION BY country ORDER BY creditLimit DESC) AS rank_in_country
   FROM customers;
   ```

---

### 4. Pipeline ETL Process & Bulk Data Load

#### The `Table Data Import Wizard` (MySQL Workbench) Problem:
When attempting to import a CSV dataset (`SF Film Locations`, 1,976 records) via the standard GUI `Table Data Import Wizard`, only **171 rows** are imported. Conclusion: UI wizards are unreliable for QA and Data Engineering due to execution timeouts and string parsing buffer limits.

#### Robust Engineering Approach: `LOAD DATA LOCAL INFILE`
1. **Preparing Table Schema (Container):**
   Profiling column lengths via `MAX(LENGTH(col))` and creating the target DDL table `film_locations`.
2. **Executing Bulk Load Query:**
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
3. **Environment Configuration:**
   * Executing in MySQL: `SET GLOBAL local_infile = 1;`
   * Configuring Workbench Connection: `Edit Connection -> Advanced -> Others -> OPT_LOCAL_INFILE=1`.

---

## 🔮 4. QA Superpower & Student Questions from Lecture

### ❓ Question 1: "What is the difference between `RANK()` and `DENSE_RANK()`, and when do we use which in QA?"
* **Mentor Answer:** `RANK()` skips rank numbers on ties (20, 20, 22), whereas `DENSE_RANK()` produces continuous numbers (20, 20, 21). In QA testing, `DENSE_RANK()` is used when validating top-N entity tiers without rank gaps. `RANK()` is used when rank position represents explicit leaderboards.

### ❓ Question 2: "Why is `TRIM()` critical during ETL data ingestion?"
* **Mentor Answer:** External CSV files or API strings often contain non-printable control characters (``, `
`) or trailing spaces. The query `WHERE status = 'Active'` evaluates to `FALSE` for `'Active '`. Applying `TRIM()` during ETL transformations prevents phantom automation bugs.

### ❓ Question 3: "Why did the UI Import Wizard load only 171 rows out of 1976?"
* **Mentor Answer:** The GUI wizard parses CSVs through Workbench script wrappers, which fail on embedded quotes and line breaks inside text fields (`fun_facts`). The SQL command `LOAD DATA LOCAL INFILE` operates at the database engine level, correctly handling string encapsulation (`ENCLOSED BY '"'`).

### ❓ Question 4: "How can window functions identify database duplicates?"
* **Mentor Answer:** By combining `ROW_NUMBER() OVER(PARTITION BY email ORDER BY id)` inside a CTE. Any record where `row_num > 1` represents a duplicate row.

---

## 💻 5. Practical Script: Live Queries & Hands-on Tasks

Practical SQL query script for live review session demonstration:

| # | Database / Table | SQL Statement / Operation | Target Concept | 💡 QA Testing Context |
|---|---|---|---|---|
| **1** | ClassicModels | `SELECT customerName, CONCAT(contactFirstName, ' ', contactLastName) AS full_name, LENGTH(customerName) FROM customers;` | String Functions | Verifying name formatting and field length limits for UI fields. |
| **2** | ClassicModels | `SELECT DISTINCT TRIM(city) FROM customers;` | Data Cleansing (`TRIM`) | Sanitizing city names from trailing spaces before verification. |
| **3** | ClassicModels | `SELECT orderNumber, DATEDIFF(shippedDate, orderDate) AS delivery_days FROM orders WHERE status = 'Shipped';` | Date Functions (`DATEDIFF`) | Validating order delivery SLA compliance. |
| **4** | ClassicModels | `SELECT orderNumber, ADDDATE(orderDate, INTERVAL 5 DAY) AS expected_ship FROM orders;` | Date Arithmetic (`ADDDATE`) | Calculating expected shipping date under "+5 days" business rule. |
| **5** | ClassicModels | `SELECT customerName, country, CASE WHEN country = 'USA' THEN 'US Customer' ELSE 'Non USA' END AS market FROM customers;` | Conditional Logic (`CASE`) | Testing dynamic user categorization by geographical market. |
| **6** | ClassicModels | `SELECT customerName, creditLimit, ROW_NUMBER() OVER(ORDER BY creditLimit DESC) AS row_num FROM customers;` | Window (`ROW_NUMBER`) | Sequential dataset numbering for pagination tests. |
| **7** | ClassicModels | `SELECT customerName, creditLimit, RANK() OVER(ORDER BY creditLimit DESC) AS rnk, DENSE_RANK() OVER(ORDER BY creditLimit DESC) AS drnk FROM customers;` | Window (`RANK` vs `DENSE_RANK`) | Comparing rank behaviors under tied credit limit values. |
| **8** | ClassicModels | `SELECT customerName, creditLimit, NTILE(4) OVER(ORDER BY creditLimit DESC) AS quartile FROM customers;` | Window (`NTILE`) | Segmenting customer base into 4 revenue quartiles for targeted marketing. |
| **9** | ClassicModels | `SELECT customerName, creditLimit, LAG(creditLimit) OVER(ORDER BY creditLimit DESC) AS prev_limit, LEAD(creditLimit) OVER(ORDER BY creditLimit DESC) AS next_limit FROM customers;` | Window (`LAG` / `LEAD`) | Analyzing credit limit gaps between adjacent ranked customers. |
| **10** | ClassicModels | `SELECT customerName, country, creditLimit, FIRST_VALUE(creditLimit) OVER(PARTITION BY country ORDER BY creditLimit DESC) AS max_country_limit FROM customers;` | Window (`PARTITION BY`) | Finding top limit per country without collapsing rows. |
| **11** | Film / SFGov | `SELECT MAX(LENGTH(title)), MAX(LENGTH(locations)), MAX(LENGTH(director)) FROM film_locations;` | Schema Profiling | Determining optimal `VARCHAR` column lengths for DDL schema design. |
| **12** | Film / SFGov | `LOAD DATA LOCAL INFILE '...' INTO TABLE film_locations FIELDS TERMINATED BY ',' ENCLOSED BY '"' IGNORE 1 ROWS;` | Bulk ETL Load | Ingesting 1,976 raw CSV records completely without data loss. |

---

## ⚠ 6. Common Student Pitfalls & Live Troubleshooting

| Symptom / Error | Root Cause | Mentor Troubleshooting Guide |
|---|---|---|
| **Error: `Loading local data is disabled; this must be enabled on both the client and server sides`** | `local_infile` variable is disabled on client or server. | Execute `SET GLOBAL local_infile = 1;` in SQL client and append `OPT_LOCAL_INFILE=1` to Workbench connection settings. |
| **Unexpected `LAST_VALUE()` result (returns current row value)** | Default window frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. | Explicitly define window frame: `LAST_VALUE(col) OVER(PARTITION BY ... ORDER BY ... RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)`. |
| **Data loss via UI Import Wizard (171 rows out of 1976)** | GUI wizard parser fails on commas and line breaks within CSV text fields. | Guide students to switch to scripted `LOAD DATA LOCAL INFILE`. |
| **`ADDDATE()` execution failure on string input** | Text date format does not match system standard `YYYY-MM-DD`. | Apply `STR_TO_DATE(date_string, '%m/%d/%Y')` prior to interval arithmetic. |

---

## ⏱ 7. Timeboxed Review Session Scenario (50 Minutes)

### 📌 Phase 1: Warm-up & Curriculum Map (5 mins)
* Welcome students and frame Lecture 7 on the 10-lecture curriculum map.
* Overview of session pillars: built-in functions, window analytics, and ETL bulk loading.

### 📌 Phase 2: Homework Review (Lesson 6) (10 mins)
* Review 3 alternative query paths for San Francisco office customers (Subqueries vs JOINs).
* Check key queries for ClassicModels 10–15 and RealEstateDB structure.

### 📌 Phase 3: Theory Deep-Dive & Student Q&A (15 mins)
* Explain `TRIM()`, `DATEDIFF()`, and `CASE` conditional statements.
* Visual comparison of `RANK()` vs `DENSE_RANK()`, `NTILE(4)`, `LAG/LEAD`.
* Address UI Import Wizard limitations and explain `LOAD DATA LOCAL INFILE`.

### 📌 Phase 4: Hands-on Live Coding in Workbench (15 mins)
* Execute schema profiling query `MAX(LENGTH())`.
* Run comprehensive window functions query on `customers` table.
* Perform bulk import of `SF Film Locations` CSV dataset using `LOAD DATA LOCAL INFILE`.

### 📌 Phase 5: Wrap-up & Homework Preview (5 mins)
* Summarize key QA data preparation practices.
* Announce Lecture 8 Preview: Temporary Tables (`Temp Tables`) and Views.

---

## 🎯 8. Mentor Checklist & Definition of Done (DoD)

By the end of the review session, students must be able to:
* [ ] Apply `TRIM()`, `LOWER()`, and `CONCAT()` for string sanitization and formatting in test queries.
* [ ] Calculate date intervals and SLA compliance using `DATEDIFF()` and `ADDDATE()`.
* [ ] Differentiate `RANK()` from `DENSE_RANK()` and apply window functions for ranking.
* [ ] Construct `LAG()` / `LEAD()` queries to analyze state transitions over time.
* [ ] Explain why UI import wizards fail on large datasets and configure `LOAD DATA LOCAL INFILE`.
* [ ] Write dynamic `CASE` expressions to build computed test columns.
