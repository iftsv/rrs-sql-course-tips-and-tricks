# [SQL-A] 8. (10/14) Table vs View vs Temp Table, SQL vs NoSQL — Review Session Guide

**Methodological Guide for Mentors and Reviewers**

**Course:** SQL for QA Automation / QA Engineer

**Session Topic:** Tables, Views, and Temporary Tables (`Tables`, `Views`, `Temporary Tables`), Building End-to-End Master Views (`Grand Master Query`), Database Architecture (5 Components), ACID Properties, SQL vs NoSQL Comparison, CAP Theorem, BASE Model, and Vector Databases (`Vector DBs`), alongside Homework Review for Lesson #7 (ClassicModels 16–24 & ETL SF Film Locations).

---

## 🧭 1. Perspective: 10-Lecture Course Map

Remind students of their current position on the course map. In Lesson 8, the focus shifts from writing isolated DQL queries to understanding **storage structures, data abstraction layers (Views/Temp Tables), and fundamental database architecture**. This knowledge is essential for designing effective test environments, optimizing regression backend automated testing, and understanding distributed databases in QA.

* **Lecture 1:** Introduction to Relational Databases (RDBMS), tables, rows, columns, primary and foreign keys, data integrity.
* **Lecture 2:** Simple single-table queries — selection, filtering, sorting, result limiting (`LIMIT`), and metadata.
* **Lecture 3/4:** SQL Components — DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation) syntax and practice.
* **Lecture 5:** Joining tables (`INNER JOIN`, `LEFT JOIN`, `SELF JOIN`, `FULL JOIN`).
* **Lecture 6:** Grouping, set operations, and subqueries (`GROUP BY`, `HAVING`, `UNION / UNION ALL`, Subqueries, `CTE / WITH`).
* **Lecture 7:** Built-in functions (Date, String, Numeric), `CASE` conditional logic, Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG/LEAD`), and ETL processes (`LOAD DATA LOCAL INFILE`).
* **Lecture 8 (Current):** Temporary Tables and Views (`Tables` vs `Views` vs `Temp Tables`), End-to-End Master Views (`Grand Master Query`), Database Architecture (5 Components), ACID Properties, SQL vs NoSQL, CAP Theorem, BASE Model, and Vector Databases (`Vector DBs`).
* **Lecture 9:** Data Warehouse concepts (DWH, Fact and Dimension tables, Star/Snowflake schemas).
* **Lecture 10:** Advanced topics (Triggers, Stored Functions, Indexes), and interview preparation.

---

## 🕵️‍♂️ 2. Session Concept & QA Testing Context

In Lesson 8, students master **storage object management and database architectural concepts**. In the daily practice of a QA Automation or Manual QA Engineer, choosing the correct table or view type directly impacts test execution speed, test environment stability, and test coverage depth.

### Key QA Objectives for DB Objects & Architecture:

1. **Building Master Views in Auto-tests & Regression (`Grand Master Query`):** Creating virtual tables that join the entire database schema (`v_classicmodels_detail`, `v_classicmodels_customer`) enables automated tests to query system state parameters in milliseconds without re-writing complex `JOIN` clauses in every test case.
2. **Test Data Isolation via `Temp Tables`:** Utilizing temporary tables (`tnp_`) for intermediate calculations and test dataset setup within a single test execution session prevents database clutter and conflicts during parallel test execution.
3. **Validating Transactional Integrity & Resilience (ACID):** Understanding Atomicity and Durability to design negative backend test scenarios (simulating network disruptions, timeouts, and transaction `ROLLBACK` in financial and e-commerce applications).
4. **Testing Distributed & NoSQL Systems (CAP & BASE Model):** Accounting for replication delay (`Eventual Consistency`) when validating microservices and NoSQL stores (e.g., delay in draft or profile updates appearing across nodes).
5. **DB Component Profiling:** Analyzing Query Processor execution, metadata catalog operations, and index usage to identify bottlenecks and configure optimal timeout thresholds in automated test suites.

---

## 🧱 3. Theoretical Deep-Dive: Homework Review, Tables, Views, Temp Tables, DB Architecture, SQL vs NoSQL

### 1. Homework Review for Lesson #7

#### Part 1: ClassicModels Database (Queries 16–24)

* **Query 16 (Rank customers by credit limit):**
  Using `RANK() OVER(ORDER BY creditLimit DESC)`.
  *Behavior:* When credit limits match (e.g., $105,000), customers receive the same rank (20), while the next rank skips values and becomes 22. The mentor highlights the contrast with `DENSE_RANK()`, which does not skip ranks (20, 20, 21).

* **Query 17 (Most sold product by city):**
  Comparing implementation approaches: CTEs (`WITH`), Subqueries, and direct 4-table joins (`customers`, `orders`, `orderdetails`, `products`).
  *💡 QA Lesson:* Different SQL constructs yield identical results but vary in readability and execution efficiency in auto-tests.

* **Query 18 (Average number of orders per customer):**
  Calculating via total orders divided by total distinct customers `COUNT(orderNumber) / COUNT(DISTINCT customerNumber)` vs. customer-level aggregation.
  *QA Lesson:* Highlighting the necessity of clarifying precise business logic requirements before writing assertion code.

* **Query 19 (Average days between order date and ship date):**
  Using date difference function `DATEDIFF(shippedDate, orderDate)`. Comparing average calculation via `AVG(DATEDIFF(...))` versus manual summation `SUM(...) / COUNT(...)`.

* **Query 20 (Sales revenue by year):**
  Applying year extraction `YEAR(paymentDate)` and grouping via `GROUP BY YEAR(paymentDate)`.

* **Query 21 (Orders not shipped — Business logic catch):**
  *Pitfall:* Filtering via `WHERE status != 'Shipped'` is flawed because it ignores `NULL` statuses and includes intermediate states (`In Process`, `On Hold`, `Disputed`).
  *Correct QA Assertion:* Using `WHERE shippedDate IS NULL`.

* **Query 22 (Sales by employee in 2003):**
  Using a Subquery to pre-filter 2003 data via `YEAR(orderDate) = 2003`, aggregating total revenue `SUM(quantityOrdered * priceEach)`, and formatting full employee names via `CONCAT(firstName, ' ', lastName)`.

* **Query 23 (City with most employees & Office with biggest sales):**
  Joining `employees` and `offices`, ordering `ORDER BY ... DESC LIMIT 1`. Comparing sales metric interpretations (revenue vs. total items sold).

* **Query 24 ("Grand Master Query" — Joining all database tables):**
  * **Detail Level:** Direct `JOIN` across all ClassicModels tables returns **12,015 rows** containing all schema attributes.
  * **Customer Level:** Aggregating data at the customer level yields **2,996 rows**.
  * **Master View Concept:** Wrapping this query into `CREATE VIEW v_classicmodels_detail AS ...` provides QA engineers with a unified data source to execute verification queries in seconds.

#### Part 2: ETL Homework Review (SF Film Locations)

* **UI Import Wizard Issue:** The standard MySQL Workbench UI wizard loads only **171 rows** out of 1,976 due to parsing errors on escaped quotes and newlines inside the `fun_facts` field.
* **Proper ETL Script Result:** Successfully ingests all **1,976 rows**.
* **DQL Verification Results:**
  * Distinct Movie Titles (`DISTINCT title`): **325 movies**.
  * Movies by Production Company: Top companies — *Paramount Pictures* (19) and *Warner Bros* (16).
  * Woody Allen Directed Movies (`director LIKE '%Woody Allen%'`): **2 movies**.
  * Fact Availability Classification (`CASE WHEN fun_facts IS NOT NULL THEN 'Yes' ELSE 'No' END`): **169 movies** with recorded facts.
  * Robin Williams Movies (`actor_1`, `actor_2`, or `actor_3`): **82 location rows**, representing **12 distinct movies**.

---

### 2. Comparative Analysis: Table vs View vs Temp Table

| Feature | Physical Table (`Table`) | View (`View`) | Temporary Table (`Temp Table`) |
| :--- | :--- | :--- | :--- |
| **Naming Naming Prefix** | `t_` (e.g., `t_payments_10k`) | `v_` (e.g., `v_payments_10k`) | `tnp_` (e.g., `tnp_payments_10k`) |
| **Data Storage** | Physically stored on disk | No data stored (only saves DDL query in metadata) | Stored in temporary memory for session duration |
| **Lifecycle** | Permanent (until `DROP TABLE`) | Permanent (until `DROP VIEW`) | Automatically dropped upon session termination |
| **GUI Visibility** | Visible in `Tables` list | Visible in `Views` list | Hidden from standard GUI object browser |
| **Creation Command** | `CREATE TABLE t_name AS SELECT ...;` | `CREATE VIEW v_name AS SELECT ...;` | `CREATE TEMPORARY TABLE tnp_name AS SELECT ...;` |
| **Primary QA Use Case** | Base application tables, staging | Master Views for regression, data masking | Intermediate dataset isolation in auto-tests |

*💡 Key Mentor Note:* Executing `DESCRIBE` returns identical metadata structural information across all three object types.

---

### 3. Database Architecture: 5 Key Components

Every modern RDBMS consists of 5 fundamental components:

1. **Interface Layer:** SQL dialect, TCP/IP networking protocols, and API drivers (JDBC, ODBC) that accept queries from client applications and test suites.
2. **Query Processor ("Database CPU"):**
   * *Parser:* Validates SQL syntax correctness.
   * *Validator:* Checks table existence and security permissions against metadata catalog.
   * *Execution Planner:* Constructs the optimal execution plan leveraging indexes.
   * *Executor:* Reads/writes storage pages, updating indexes and metadata.
3. **Storage System:** Disk blocks or RAM memory holding physical data pages.
4. **Indexes:** Auxiliary data structures (B-Tree, Hash) enabling high-speed record lookup without triggering a `Full Table Scan`.
5. **Metadata (System Catalog):** Schema definitions, table structures, column types, permissions, and compression statistics.

#### OLTP vs OLAP Context:
* **OLTP (Online Transaction Processing):** Operational databases optimized for frequent, low-latency row transactions (INSERT/UPDATE), highly normalized schemas.
* **OLAP (Online Analytical Processing):** Data warehouses (DWH) optimized for complex analytical aggregations across massive historical datasets.

---

### 4. SQL vs NoSQL, ACID Properties, CAP Theorem, BASE, and Vector DBs

#### A. ACID Properties in SQL Databases:
* **Atomicity:** Transactions execute entirely or not at all ("all or nothing"). Any failure triggers a `ROLLBACK`.
* **Consistency:** Transactions transition the database from one valid state to another, maintaining all constraints (`Primary/Foreign Keys`, `CHECK`).
* **Isolation:** Concurrent transactions execute without interfering with one another prior to commitment.
* **Durability:** Once committed (`COMMIT`), changes persist to non-volatile storage and survive system crashes.

#### B. SQL vs NoSQL:
* **SQL:** Highly structured, `Schema-on-Write`, ACID compliant, `Vertical Scaling`. Ideal for financial, healthcare, and transactional applications.
* **NoSQL:** Flexible `Schema-on-Read`, high throughput, `Horizontal Scaling`. Ideal for social networks, e-commerce product catalogs, and Big Data workloads.

#### C. CAP Theorem & BASE Model:
* **CAP Theorem:** Distributed systems can simultaneously guarantee at most **2 of 3** properties: **Consistency**, **Availability**, **Partition Tolerance**.
* **BASE Model (NoSQL):** **Basically Available**, **Soft state**, **Eventually Consistent**.
  * *Audio Example:* Gmail draft synchronization. Edits saved on a mobile phone appear on a desktop browser after a few seconds (replication delay window).

#### D. Vector Databases (`Vector DBs`):
* Databases optimized for storing and querying high-dimensional numerical vectors (vector embeddings). Fundamental for artificial intelligence, LLM applications, and semantic search systems.

---

## 🔮 4. QA Superpowers & Student Q&A Breakdown

### ❓ Question 1: "Why create a Master View over the entire DB (`v_classicmodels_detail`) if joining all tables produces duplicate column names?"
* **Mentor Answer:** A Master View provides a single data abstraction layer, saving QA engineers from repeatedly writing complex `JOIN` logic. However, joining tables with identical column names (e.g., `status` in `orders` and `status` in `payments`) causes naming collisions. In QA practice, engineers must explicitly define unique column aliases during View creation: `SELECT o.status AS order_status, p.status AS payment_status`.

### ❓ Question 2: "When should auto-tests use a Temp Table versus a View?"
* **Mentor Answer:** A View is used when the query logic is required repeatedly across multiple test suites or team members (e.g., a reusable view for active user filtering). A Temp Table is used when a test suite needs to temporarily isolate intermediate data (e.g., a list of generated order IDs) strictly for the duration of a single execution run without polluting the main database.

### ❓ Question 3: "How do you test Atomicity (from ACID) during API and DB integration testing?"
* **Mentor Answer:** By simulating an intentional failure during a multi-step transaction (e.g., passing an invalid payload on step 2 of 3). The QA engineer verifies that the DBMS triggers a `ROLLBACK` and that zero partial state records persist in the database.

### ❓ Question 4: "How do you properly automate testing for Eventually Consistent NoSQL systems?"
* **Mentor Answer:** In NoSQL databases, executing an immediate `assert` right after a write call will fail due to replication lag. QA Automation engineers implement polling wait mechanisms (`Polling / Awaitility`) that retry assertions until the system reaches an eventually consistent state within the expected SLA timeframe.

---

## 💻 5. Practical Script: Live Queries & Hands-on Tasks

| # | Database / Object | SQL Query / Operation | Core Concept | 💡 QA Context & Verification Scenario |
|---|---|---|---|---|
| **1** | ClassicModels | `CREATE TABLE t_payments_10k AS SELECT * FROM payments WHERE amount < 10000;` | Physical Table (`t_`) | Physically creating a sample table to isolate a test dataset. |
| **2** | ClassicModels | `CREATE VIEW v_payments_10k AS SELECT * FROM payments WHERE amount < 10000;` | View (`v_`) | Creating a lightweight view without consuming additional disk space. |
| **3** | ClassicModels | `CREATE TEMPORARY TABLE tnp_payments_10k AS SELECT * FROM payments WHERE amount < 10000;` | Temp Table (`tnp_`) | Generating a temporary table for an isolated test session. |
| **4** | ClassicModels | `DESCRIBE t_payments_10k; DESCRIBE v_payments_10k; DESCRIBE tnp_payments_10k;` | Structure Profiling | Demonstrating identical metadata structure output across all object types. |
| **5** | ClassicModels | `CREATE VIEW v_classicmodels_detail AS SELECT c.customerNumber, c.customerName, o.orderNumber, o.status AS order_status, p.productName, pm.amount FROM customers c JOIN orders o USING (customerNumber) JOIN orderdetails od USING (orderNumber) JOIN products p USING (productCode) JOIN payments pm USING (customerNumber);` | Grand Master View | Building a schema-wide view for rapid test validation. |
| **6** | ClassicModels | `SELECT customerName, creditLimit, RANK() OVER(ORDER BY creditLimit DESC) AS rnk FROM customers;` | Window (`RANK`) | Validating rank skipping behavior on identical credit limit values. |
| **7** | ClassicModels | `SELECT c.city, p.productName, SUM(od.quantityOrdered) AS total_sold FROM customers c JOIN orders o USING (customerNumber) JOIN orderdetails od USING (orderNumber) JOIN products p USING (productCode) GROUP BY c.city, p.productName;` | Multi-table Aggregation | Verifying top-selling product quantities grouped by customer city. |
| **8** | ClassicModels | `SELECT orderNumber, DATEDIFF(shippedDate, orderDate) AS days_to_ship FROM orders WHERE shippedDate IS NOT NULL;` | Date Functions | Checking shipping intervals to validate SLA compliance. |
| **9** | ClassicModels | `SELECT YEAR(paymentDate) AS sales_year, SUM(amount) AS total_revenue FROM payments GROUP BY YEAR(paymentDate);` | Date Grouping | Aggregating annual financial revenue figures. |
| **10** | ClassicModels | `SELECT COUNT(*) AS non_shipped_count FROM orders WHERE shippedDate IS NULL;` | Logic Fix (`IS NULL`) | Correctly counting non-shipped orders handling `NULL` values. |
| **11** | SF Film / ETL | `SELECT title, CASE WHEN fun_facts IS NOT NULL AND fun_facts != '' THEN 'Yes' ELSE 'No' END AS has_facts FROM film_locations;` | Conditional Logic | Testing dynamic translation of fact presence indicators. |
| **12** | System Schema | `SELECT table_name, table_type FROM information_schema.tables WHERE table_schema = 'ClassicModels';` | Metadata Catalog | Inspecting object catalog metadata in system tables. |

---

## ⚠ 6. Common Student Pitfalls & Live Troubleshooting

| Problem / Symptom | Root Cause | Mentor Troubleshooting Step |
|---|---|---|
| **Error: `Duplicate column name 'status'` when calling `CREATE VIEW`** | Using `SELECT *` across tables sharing identical column names. | Explicitly list columns in `SELECT` and assign unique aliases (`o.status AS order_status`). |
| **Symptom: Temporary table `tnp_` disappeared or is inaccessible in another Workbench tab** | Query executed from a different DB connection. Temp tables are strictly session-scoped. | Explain session isolation concepts to students and execute queries within the same connection tab. |
| **Error: DML changes are invisible to concurrent automated test runs** | `autocommit` is disabled in the session, and no explicit `COMMIT` was executed. | Check autocommit mode via `SELECT @@autocommit;` or execute an explicit `COMMIT;`. |
| **NoSQL Test Failure: `AssertionError` checking a newly created record** | Test reads from a secondary replica node before asynchronous replication completes (`Eventual Consistency`). | Implement a retry polling mechanism (`polling`) with timeout in the test automation script. |

---

## ⏱ 7. Step-by-Step Review Session Timeline (50 Minutes)

### 📌 Phase 1: Introduction & Course Map (5 min)
* Welcome, positioning Lesson 8 on the 10-lecture course map.
* Overview of core topics: `Tables` vs `Views` vs `Temp Tables`, DB Architecture, ACID, and NoSQL.

### 📌 Phase 2: Homework Review for Lesson #7 (10 min)
* Review ClassicModels 16–24, `RANK()` behavior, non-shipped orders pitfall (`shippedDate IS NULL`).
* Analyze SF Film Locations ETL results (1,976 rows) and UI Import Wizard limitations.

### 📌 Phase 3: Theoretical Deep-Dive (15 min)
* Comparative breakdown of `Tables`, `Views`, and `Temp Tables`. Master View creation walkthrough.
* 5 DB Architecture components, ACID properties, CAP theorem, BASE model, and `Eventual Consistency`.

### 📌 Phase 4: Live Coding & MySQL Workbench Practice (15 min)
* Hands-on creation of `t_`, `v_`, and `tnp_` objects. Comparing `DESCRIBE` outputs.
* Querying Master Views and profiling metadata in `information_schema`.

### 📌 Phase 5: Wrap-up & DoD Check (5 min)
* Student Q&A, verifying completion against Definition of Done checklist.
* Preview of Lesson 9: Data Warehouse concepts (DWH, Facts and Dimensions, Star/Snowflake schemas).

---

## 🎯 8. Mentor Checklist & Definition of Done (DoD)

By the end of the review session, the student should be able to:

* [ ] Understand fundamental differences between `Tables`, `Views`, and `Temporary Tables` and apply them appropriately.
* [ ] Design and leverage Master Views (`v_`) to accelerate test assertion query execution.
* [ ] Use Temporary Tables (`tnp_`) for isolated test data preparation during automated test runs.
* [ ] Explain the 5 core database architecture components and the role of the Query Processor.
* [ ] Define ACID properties and justify their criticality when testing backend transactional operations.
* [ ] Explain CAP theorem trade-offs, NoSQL `Eventual Consistency`, and strategies for automating QA checks in distributed systems.
