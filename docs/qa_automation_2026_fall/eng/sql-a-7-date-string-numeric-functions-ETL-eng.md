# [SQL-A] 6. (09/30) Group By, Union, Sub Queries — Review Session Guide
**Instructional Guide for Mentors and Reviewers**  
**Course:** SQL for QA Automation / QA Engineer  
**Session Topic:** Data Grouping (GROUP BY, HAVING), Set Operations (UNION / UNION ALL), Subqueries, Common Table Expressions (CTE / WITH), Homework Review (ClassicModels 10–15, RealEstateDB), and Integration with Built-in Functions & ETL Processes.

---

## 🧭 1. Perspective: 10-Lecture Curriculum Map

Remind students of their current position on the 10-lecture curriculum map. This helps synthesize knowledge and illustrates how mastering data aggregation, set unification, subqueries, and built-in functions creates the foundation for testing analytical dashboards, financial reconciliation, ETL pipelines, and complex database validations.

*   **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary & Foreign Keys, Data Integrity.
*   **Lecture 2:** Simple Queries on a Single Table — Data Selection, Filtering (WHERE), Sorting (ORDER BY), Result Limiting (LIMIT).
*   **Lecture 3/4:** SQL Components — Syntax & Practice of DDL (Data Definition), DQL (Data Query), DML (Data Manipulation).
*   **Lecture 5:** Table Joins (INNER JOIN, LEFT JOIN, SELF JOIN, FULL JOIN).
*   **Lecture 6 (Current Review Session):** Grouping, Unions, and Subqueries (GROUP BY, HAVING, UNION / UNION ALL, Subqueries, CTE).
*   **Lecture 7 (Covered Theory):** Built-in Functions (String, Numeric, Date/Time, CASE, Window Functions) & ETL Processes (`LOAD DATA LOCAL INFILE`).
*   **Lecture 8:** Temporary Tables and Views (Views vs. Temp Tables).
*   **Lecture 9:** Data Warehouse Concepts (DWH, Fact & Dimension Tables, Star/Snowflake schemas).
*   **Lecture 10:** Advanced Topics (Triggers, Stored Functions) and QA Interview Preparation.

---

## 🕵️‍♂️ 2. Session Concept & QA Testing Context

In previous sessions, students learned how to combine tables using `JOIN`. In Lessons 6 and 7, we progress to analyzing, aggregating, cleansing, and transforming large volumes of data. In daily QA practice, groupings, subqueries, built-in functions, and ETL scripts serve as primary tools for automated and manual Data Reconciliation.

### Core QA Objectives when Testing GROUP BY, UNION, Subqueries, Functions & ETL:
1.  **Business Metric & Analytics Validation (KPI & Report Testing):** Verifying backend aggregated indicators (total revenue, average order value, active users per region, profit margins) against UI dashboards and analytical exports.
2.  **Data Reconciliation during Mergers & Migrations:** Utilizing `UNION` and `UNION ALL` to validate data migrations, identify lost or duplicate records, and compile unified customer directories across multi-brand databases.
3.  **Threshold & Boundary Testing (HAVING & Aggregates):** Identifying anomalous customer segments or transactions (e.g., customers with cumulative payments > $70,000) to test edge-case loyalty business logic.
4.  **Validating In-Database Transformations (Data Cleansing & Formatting):** Using `TRIM`, `CONCAT`, `CASE WHEN`, and `DATEDIFF` to verify address formatting, customer age calculations, and reporting tier categories.
5.  **ETL & Ingestion Performance Testing:** Comparing UI import wizards (`Table Data Import Wizard`) with high-throughput batch loading via `LOAD DATA LOCAL INFILE` for high-volume database testing.

---

## 🧱 3. Theoretical Deep Dive & Homework Review

Before introducing new exercises, the mentor conducts a quick review of the Lecture 6 homework solutions, incorporating insights from Lecture 7.

### Part 1: Homework Review — ClassicModels Database (Queries 10–15)

#### 1. Query 10: Count Vendors, Product Lines, and Products
*   **SQL Query:**
    ```sql
    SELECT 
        COUNT(DISTINCT productVendor) AS total_vendors,
        COUNT(DISTINCT productLine) AS total_lines,
        COUNT(DISTINCT productCode) AS total_products
    FROM products;
    ```
*   **Result:** 7 vendors, 7 product lines, 110 products.
*   **QA Tip:** Highlight the critical use of `COUNT(DISTINCT ...)`. Standard `COUNT(...)` counts raw rows with duplicates, leading to false positives when verifying distinct entity counts.

#### 2. Query 11: Average Buy Price per Vendor
*   **SQL Query:**
    ```sql
    SELECT 
        productVendor, 
        AVG(buyPrice) AS avg_buy_price
    FROM products
    GROUP BY productVendor;
    ```
*   **💡 Critical Error Review from Lecture:** On the lecture, an example was discussed where `productName` was selected while grouping by `productCode`. Remind students of the **ANSI SQL Golden Rule**: *Every non-aggregated column in SELECT MUST be explicitly listed in GROUP BY*. Selecting non-grouped columns produces non-deterministic results and breaks automated test assertions.

#### 3. Query 12: Average Payment Check per Customer
*   **SQL Query:**
    ```sql
    SELECT 
        c.customerNumber,
        c.customerName,
        AVG(p.amount) AS avg_payment_amount
    FROM customers c
    JOIN payments p ON c.customerNumber = p.customerNumber
    GROUP BY c.customerNumber, c.customerName;
    ```
*   **QA Context:** Joins `customers` and `payments` to validate backend customer spending averages.

#### 4. Query 13: Product Sold the Most
*   **Clarification Question:** Does "sold the most" mean highest number of distinct orders (`COUNT(orderNumber)`) or highest total volume of units (`SUM(quantityOrdered)`)?
*   **Query by Order Line Frequency:**
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
*   **QA Tip:** QA engineers must always clarify ambiguous business definitions ("sold the most") with Product Owners or Analysts before writing test automation scripts.

#### 5. Query 14: Margin / Revenue Calculation (MSRP vs. Buy Price)
*   **SQL Query:**
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
*   **QA Context:** Validates financial calculations on backend APIs against database formula `(MSRP - BuyPrice) * Quantity`.

---

### Part 2: Homework Review — RealEstateDB Database

1.  **All Listings with Property Address, Price, Agent Name, and Status:**
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
2.  **Showings with Client Name and Property Info:**
    *   Joins 4 tables: `showings` + `listings` + `properties` + `clients`.
    *   Uses `CONCAT(c.first_name, ' ', c.last_name)` for client full name formatting.
3.  **Completed Property Transactions:**
    *   Combines 5 tables (`transactions`, `offers`, `listings`, `properties`, `clients`) for end-to-end purchasing workflow validation.

---

### Part 3: Subqueries — Multiple Solution Paths for One QA Problem

The lecture thoroughly analyzed the problem: *"Show all customer names whose sales representatives (employees) work in the San Francisco office."*

*   **Path 1: Nested Subqueries (2-Level Subquery)**
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
*   **Path 2: Subquery with Internal JOIN**
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
*   **Path 3: Direct INNER JOIN without Subqueries**
    ```sql
    SELECT DISTINCT c.customerName 
    FROM customers c
    JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber
    JOIN offices o ON e.officeCode = o.officeCode
    WHERE o.city = 'San Francisco';
    ```
*   **💡 QA Mentor Takeaway:** All three approaches yield exactly **12 records**. This demonstrates that SQL offers multiple syntax options for testing. However, Paths 2 and 3 are preferred for automated test suites due to better readability and maintainability.

---

## 🔮 4. QA Superpower & Student Questions from Lecture

Key student questions and common misconceptions captured in the Lecture 7 audio recording:

### ❓ Question 1: “Why did the UNION query return 101 rows when 100 were expected?”
*   **Mentor Explanation:** `UNION` removes duplicate rows by comparing **all selected columns across the entire row**. If a synthetic string literal is added (e.g., `'ClassicModels' AS DB` vs. `'MyWork' AS DB`), a city name like `San Francisco` paired with different database labels becomes two UNIQUE rows. The 101st record is caused by the database source tag rendering otherwise duplicate rows distinct!

### ❓ Question 2: “Why is TRIM essential during ETL and data population?”
*   **Mentor Explanation:** When importing external CSV/JSON files, invisible whitespace or trailing newline characters break string comparisons like `WHERE name = 'John'`. The `TRIM()` function is a QA's primary tool for Data Cleansing. In automated tests, wrapping string fields in `TRIM(column)` prevents false failures.

### ❓ Question 3: “What is the difference between `Table Data Import Wizard` and `LOAD DATA LOCAL INFILE`?”
*   **Mentor Explanation:** 
    *   The Workbench `Wizard` is convenient for small ad-hoc tests, but it is slow and **unreliable**: for a 1,976-row file, it may silently load only 171 rows due to parsing errors or quotes.
    *   `LOAD DATA LOCAL INFILE` is an industrial high-performance SQL command that loads hundreds of thousands of records in seconds without loss. QA engineers must use `LOAD DATA LOCAL INFILE` for performance and volume testing.

### ❓ Question 4: “Why should QA engineers care about Window Functions (ROW_NUMBER, RANK, DENSE_RANK, LAG/LEAD)?”
*   **Mentor Explanation:**
    *   `ROW_NUMBER()` / `DENSE_RANK()` allow QA to detect duplicate records or extract top-N items per group without collapsing rows (unlike `GROUP BY`).
    *   `LAG()` / `LEAD()` enable comparing the current row value against preceding or succeeding rows (e.g., verifying status changes or balance deltas over time).

---

## 💻 5. Practical Script: Live Queries & Hands-on Tasks

Summary breakdown of practical queries for the review session:

| # | Database | SQL Query / Operation | Target Concept | 💡 QA Testing Context |
|---|---|---|---|---|
| **1** | ClassicModels | `SELECT COUNT(*), COUNT(DISTINCT country), MAX(creditLimit) FROM customers;` | Baseline Profiling | Initial table audit before writing requirement assertions. |
| **2** | ClassicModels | `SELECT country, COUNT(*) FROM customers GROUP BY country;` | GROUP BY | Verifying entity distribution across geographical regions. |
| **3** | ClassicModels | `SELECT customerNumber FROM payments GROUP BY customerNumber HAVING SUM(amount) > 70000;` | GROUP BY + HAVING | Pinpointing VIP customers exceeding the $70,000 threshold for loyalty tests. |
| **4** | ClassicModels | `SELECT city, 'ClassicModels' AS DB FROM customers UNION ALL SELECT city, 'Department' AS DB FROM department;` | UNION ALL (Preserve Duplicates) | Validating total record unification across two microservices. |
| **5** | ClassicModels | `SELECT city FROM customers UNION SELECT city FROM department;` | UNION (Deduplicated) | Generating a unified deduplicated master directory. |
| **6** | ClassicModels | `SELECT customerName FROM customers WHERE salesRepEmployeeNumber IN (SELECT employeeNumber FROM employees WHERE officeCode IN (SELECT officeCode FROM offices WHERE city = 'San Francisco'));` | Subquery in WHERE | Preparing customer test datasets for end-to-end validation. |
| **7** | ClassicModels | `WITH CTE_VIP AS (SELECT customerNumber, SUM(amount) AS total FROM payments GROUP BY customerNumber HAVING total > 70000) SELECT c.*, v.total FROM customers c JOIN CTE_VIP v ON c.customerNumber = v.customerNumber;` | CTE (WITH ... AS) | Clean, modular test script structure for financial auditing. |
| **8** | ClassicModels | `SELECT customerName, TRIM(addressLine1), UPPER(city), CONCAT(contactFirstName, ' ', contactLastName) FROM customers;` | Text Functions (TRIM, UPPER, CONCAT) | Verifying data cleansing and text formatting in test assertions. |
| **9** | ClassicModels | `SELECT orderNumber, DATEDIFF(shippedDate, orderDate) AS processing_days FROM orders WHERE shippedDate IS NOT NULL;` | Date Functions (DATEDIFF) | Verifying fulfillment SLA delivery times. |
| **10** | ClassicModels | `SELECT customerName, creditLimit, CASE WHEN creditLimit > 100000 THEN 'Platinum' WHEN creditLimit > 50000 THEN 'Gold' ELSE 'Standard' END AS tier FROM customers;` | Advanced CASE Statement | Testing business tier classification logic. |
| **11** | Film / ETL | `LOAD DATA LOCAL INFILE '/path/films.csv' INTO TABLE film_locations FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '
' IGNORE 1 ROWS;` | ETL Direct Ingestion | Mass data loading and validation without record drop-offs. |

---

## ⚠ 6. Common Student Pitfalls & Live Troubleshooting

| Error Code / Symptom | Root Cause | Mentor Solution & Explanation |
|---|---|---|
| **Error Code: 1111. Invalid use of group function** | Placing aggregate functions (`SUM`, `COUNT`, `AVG`) inside `WHERE`. | Explain SQL execution order: `FROM` ➔ `WHERE` ➔ `GROUP BY` ➔ `HAVING` ➔ `SELECT`. At `WHERE`, rows are not aggregated yet. Filter aggregates using `HAVING`. |
| **Error Code: 1248. Every derived table must have its own alias** | A nested subquery in the `FROM` clause lacks a table alias. | Show how to add an explicit table alias at the end of subquery parentheses: `FROM (SELECT ...) AS sub_table`. |
| **Column count mismatch in UNION** | Mismatched column counts or incompatible data types across `UNION` queries. | Emphasize the golden rule: every `SELECT` in a `UNION` must have identical column counts and positions. |
| **Non-aggregated columns in SELECT and GROUP BY** | Selecting columns in `SELECT` that are neither aggregated nor listed in `GROUP BY`. | Explain ANSI SQL rules: all non-aggregated columns in `SELECT` must be declared in `GROUP BY`. |
| **Error Code: 3948. Loading local data is disabled** | Attempting `LOAD DATA LOCAL INFILE` when local infile is disabled on server/client. | Run `SET GLOBAL local_infile = 1;` and configure `OPT_LOCAL_INFILE=1` in Workbench connection settings. |

---

## ⏱ 7. Timeboxed Review Session Scenario (50 Minutes)

### 📌 Phase 1: Warm-up & Curriculum Map (5 mins)
*   Welcome students and position Lessons 6 & 7 on the 10-lecture curriculum map.
*   Highlight the transition to analytical queries, aggregate calculations, and ETL reconciliation.

### 📌 Phase 2: Homework Review (10 mins)
*   Review key queries from ClassicModels #10–15 (vendor counts, margin calculation, `COUNT(DISTINCT)`).
*   Walk through RealEstateDB queries (4–5 table joins for showings and transactions).

### 📌 Phase 3: Theory Deep-Dive & Student Questions (15 mins)
*   Address questions from Lecture 7 recording: why `UNION` yielded 101 records, `WHERE` vs. `HAVING`, why UI wizards drop records.
*   Explain practical QA applications of `CASE WHEN`, `TRIM()`, `DATEDIFF()`, and Window Functions.

### 📌 Phase 4: Hands-on Live Coding (15 mins)
*   Demonstrate `GROUP BY` with `HAVING SUM(amount) > 70000` on `payments`.
*   Build equivalent queries using Subquery in `WHERE` and CTE (`WITH`).
*   Demonstrate data cleansing and formatting using `TRIM`, `DATEDIFF`, and `CASE`.

### 📌 Phase 5: Wrap-up & Next Steps (5 mins)
*   Summarize key takeaways: aggregate grouping rules, subqueries, and batch ingestion.
*   Preview Lecture 8: Temporary Tables and Views (Views vs. Temp Tables).

---

## 🎯 8. Mentor Checklist & Definition of Done (DoD)

By the end of the review session, students should be able to:
*   [ ] Perform baseline profiling audits on unstudied tables using `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
*   [ ] Articulate the difference between row filtering (`WHERE`) and aggregate filtering (`HAVING`).
*   [ ] Apply `GROUP BY` correctly, obeying non-aggregated column declaration rules.
*   [ ] Differentiate between `UNION` (deduplicated) and `UNION ALL` (duplicates preserved) for data migration testing.
*   [ ] Construct subqueries with the `IN` operator and build clean, modular CTE expressions (`WITH`).
*   [ ] Apply built-in functions (`TRIM`, `CONCAT`, `DATEDIFF`, `CASE`) to prepare and assert test data.
*   [ ] Understand the workflow of bulk ETL ingestion via `LOAD DATA LOCAL INFILE` over UI wizards.
