# [SQL-A] 4. (09/09) DDL, DQL, DML - Review Session Guide

> **Pedagogical Guide for Mentors and Reviewers** [1]  
> **Course:** SQL for QA Automation / QA Engineer [1]  
> **Session Topic:** SQL Architecture (DDL, DQL, DML), Creating and Dropping Databases/Tables, Data Types, Constraints (Primary Key, Foreign Key), Data Manipulation, and Practical Work with `MyWork`, `UniversityDB`, and `ClassicModels` Databases [1, 26, 32].

---

## 🧭 1. Perspective: 10-Lecture Curriculum Map

Remind students of their current position on the course curriculum map [2]. This helps reduce anxiety regarding the volume of material and demonstrates how fundamental data creation and modification operations establish the groundwork for test environment design and handling complex relationships [2, 26].

* **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary and Foreign Keys, Data Integrity [2].
* **Lecture 2:** Simple Queries on a Single Table — Data Selection, Filtering, Sorting, Result Limiting, and Metadata [2].
* **Lecture 3/4 (Current):** SQL Components — Syntax & Practice of DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation) [2, 26, 32].
* **Lecture 5:** Table Joins (Inner Join, Left Join, Self Join) [2].
* **Lecture 6:** Grouping, Unions, and Subqueries (Group By, Union/Union All, Subqueries) [2].
* **Lecture 7:** Built-in Functions (String, Numeric, Date & Time Functions) [2].
* **Lecture 8:** Temporary Tables and Views (Views vs. Temp Tables) [2].
* **Lecture 9:** Data Warehouse Concepts (DWH, Fact and Dimension Tables) [2].
* **Lecture 10:** Advanced Topics (Triggers, Stored Functions) and Interview Preparation [2].

---

## 🕵️‍♂️ 2. Session Concept: Building DB "From Scratch" & Test Environment Management

In previous sessions, students worked with pre-existing, read-only databases [2, 4]. In real-world QA engineering (both manual and automated), software testers frequently encounter test environment management tasks [3, 26]:
* **Deploying Test Databases "From Scratch":** Creating schemas and tables for isolated automated test runs in CI/CD pipelines [26, 38].
* **Test Data Seeding:** Populating tables with baseline data sets (`INSERT`) and updating records (`UPDATE`) to reproduce bug scenarios [32, 34, 35].
* **Environment Teardown & Reset:** Rapidly clearing database state after test suite execution (`DELETE`, `TRUNCATE`, `DROP`) [35, 53].
* **Schema Integrity & Migration Validation:** Validating column constraints (`NOT NULL`, `PRIMARY KEY`, `FOREIGN KEY`) and schema changes (`ALTER TABLE`) during new service releases [28, 29, 30, 48].

Mastering DDL, DML, and DQL commands enables QA engineers to go beyond querying data, giving them full control over the data lifecycle throughout backend testing [26, 32, 38].

---

## 🧱 3. Theoretical Deep Dive: DDL, DQL, DML in QA Context

### 1. DDL (Data Definition Language)
DDL governs the **structure** of database schemas and objects (databases, tables, columns, indexes) [26, 32].

#### Database Management (`CREATE` & `DROP`) [26, 36, 37]:
* `CREATE DATABASE database_name;` (or `CREATE SCHEMA`) — creates a new empty database [26].
* `DROP DATABASE database_name;` — permanently deletes the database along with all its tables [26].
* **Safe Execution Guards (`IF EXISTS` / `IF NOT EXISTS`) [36, 37, 38]:**  
  ```sql
  DROP DATABASE IF EXISTS mywork;
  CREATE DATABASE IF NOT EXISTS mywork;
  ```  
  *💡 **QA Context (Test Automation):** Using `IF EXISTS` / `IF NOT EXISTS` is vital in automated test setup scripts. If a script attempts to drop a non-existent database or table without `IF EXISTS`, the RDBMS returns a fatal Error Code, causing the CI/CD pipeline execution to crash [37, 38]. The `IF EXISTS` clause converts a fatal error into a manageable Warning, allowing automated teardown scripts to run idempotently [37, 38].*

#### Table Creation (`CREATE TABLE`) and Data Types [27, 28, 29]:
```sql
CREATE TABLE mywork.employee_test (
    employee_id INT NOT NULL,
    hire_date DATE,
    last_name VARCHAR(75),
    first_name VARCHAR(75),
    phone_number VARCHAR(25),
    salary DECIMAL(10,2),
    PRIMARY KEY (employee_id)
);
```
* **Key Data Types for QA Engineers [27, 28]:**
  * **String Types (`VARCHAR`, `CHAR`):** Variable or fixed length strings [27, 28]. *QA Tip:* When testing fields with unpredictable formats (e.g., phone numbers containing dashes, spaces, or plus signs), using `VARCHAR` prevents unexpected type casting failures [28].
  * **Numeric Types (`INT`, `DECIMAL(p, s)`):** `DECIMAL(10,2)` specifies 10 total digits with 2 decimal places (ideal for monetary amounts and financial salaries, e.g., $10.99) [28, 29].
  * **Date & Time Types (`DATE`, `DATETIME`, `TIMESTAMP`):** Standard MySQL date literals are formatted as `'YYYY-MM-DD'` (e.g., `'2018-01-01'`) [20, 27].
* **Integrity Constraints (`NOT NULL`, `PRIMARY KEY`) [28, 29]:** `PRIMARY KEY` enforces row uniqueness and strictly forbids `NULL` values [28, 29].

#### Table Structure Modification (`ALTER TABLE`) [30, 31, 42, 44, 45]:
* **Adding a Column:** `ALTER TABLE employee_test ADD COLUMN email VARCHAR(50);` (appends the new field to the end of the table) [30].
* **Dropping a Column:** `ALTER TABLE employee_test DROP COLUMN email;` [31, 45].
* **Renaming a Column:** `ALTER TABLE EMP RENAME COLUMN job TO job_title;` [42].

---

### 2. DQL (Data Query Language)
DQL handles reading and retrieving data from tables [31, 32].
* **Core Command:** `SELECT column1, column2 FROM table_name WHERE condition;` [7, 31].
* In this session, DQL serves as a **verification tool** to inspect and validate the state of data after executing DML operations (e.g., verifying inserted, updated, or deleted rows) [34, 35, 52].

---

### 3. DML (Data Manipulation Language)
DML manages **data rows (records)** stored inside database tables [32, 35].

#### Data Insertion (`INSERT INTO`) [32, 33, 34, 42]:
```sql
-- Single-row insert
INSERT INTO employee_test (employee_id, hire_date, last_name, first_name, phone_number, salary)
VALUES (999999, '2018-01-01', 'Smith', 'John', '555-0199', 100000.00);

-- Batch insert (multiple rows in a single query)
INSERT INTO EMP (empno, ename, job_title, mgr, hiredate, sal, comm, deptno)
VALUES 
(7369, 'SMITH', 'CLERK', 7902, '1980-12-17', 800.00, NULL, 20),
(7499, 'ALLEN', 'SALESMAN', 7698, '1981-02-20', 1600.00, 300.00, 30);
```
* ⚠️ **Golden Rule for Students:** The listed column order in the statement must strictly match the position and data types of values in the `VALUES` clause [33].

#### Data Update (`UPDATE`) [34, 35, 43]:
```sql
-- Global table update: giving a 10% raise to all employees
UPDATE employee_test SET salary = salary * 1.1;

-- Targeted update with a WHERE clause
UPDATE EMP SET ename = 'Smith' WHERE ename = 'Polk';
```
* ⚠️ **QA Safety Warning:** Executing an `UPDATE` statement without a `WHERE` clause will modify **every single row** in the table [34, 35]!

#### Row Deletion (`DELETE`) [35, 43, 44]:
```sql
DELETE FROM EMP WHERE ename = 'Roosevelt';
```

---

### 4. Comparison of Table Cleanup Operations: `DROP` vs `DELETE` vs `TRUNCATE`

| Operation | Language Category | What it Deletes | Performance on Large Tables (1M+ rows) | Preserves Table Structure | QA Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`DROP TABLE`** [26, 31, 35] | DDL [31] | Entire table (schema definition + all data) [31, 35] | Instant [35] | ❌ No (table is removed from DB catalog) [31, 35] | Destroying temporary test schemas or teardown after automated test runs [31, 35, 36]. |
| **`DELETE FROM`** [35, 44, 53] | DML [32] | Specific rows matching `WHERE` or all rows [35, 53] | Slow (deletes and logs row-by-row) [53] | ✅ Yes (structure remains intact) [53] | Targeted removal of test data generated by automated test cases (`WHERE test_id = 123`) [35, 43, 44]. |
| **`TRUNCATE TABLE`** [53, 54] | DDL [53] | All rows in table (complete data purge) [53, 54] | Ultra-Fast (deallocates data pages without row logging) [53] | ✅ Yes (structure & indexes preserved) [53, 54] | Fast test suite teardown/reset of large lookup tables or event logs before a test run [53, 54]. |

---

### 5. Table Relationships (Primary & Foreign Keys) and Reverse Engineering
Linking tables like `EMP` (employees) and `dept` (departments) via the `deptno` column establishes a Primary Key — Foreign Key relationship [46, 48].

```sql
-- Adding a Foreign Key constraint
ALTER TABLE EMP 
ADD CONSTRAINT fk_emp_dept 
FOREIGN KEY (deptno) REFERENCES dept(deptno);
```

* ⚠️ **Technical Nuance from Lecture:** Attempting to add a `FOREIGN KEY` constraint to `EMP` when the parent table `dept` is empty or missing referenced key values will trigger an RDBMS referential integrity constraint error [47]. Always populate parent tables before establishing FK constraints or adding child records [47, 48]!
* **Visualization in MySQL Workbench (Reverse Engineer):** Navigating to `Database -> Reverse Engineer` automatically generates an Entity-Relationship (ER) Diagram, enabling visual validation of `PRIMARY KEY` (key icon) and `FOREIGN KEY` connections [49].

---

### 6. Auxiliary Administrative Functions & DBA Shortcuts [55]
* `SELECT NOW();` — returns the current server timestamp [55].
* `SHOW VARIABLES LIKE 'version';` — displays the installed MySQL database version [55].
* `SELECT CURRENT_USER();` — displays the current session user and host (`local_host`) [55].
* `ANALYZE TABLE table_name;` — analyzes and optimizes table key distribution stats when query execution slows down [55].

---

## 💻 4. Practical Script: Live Queries & Hands-on Tasks

Reviewers should walk through these queries with students, highlighting both SQL execution and QA testing context.

| # | Database | Task / Query | Target Concept | QA Testing Context |
| :-: | :--- | :--- | :--- | :--- |
| **1** | `sys` / Global | `CREATE DATABASE IF NOT EXISTS mywork;`<br>`DROP DATABASE IF EXISTS mywork;` | DDL: Safe Database Management [26, 36, 37] | Ensuring idempotent setup and teardown scripts in CI/CD pipeline automation [37, 38]. |
| **2** | `mywork` | `CREATE TABLE employee_test (<columns...>);` | DDL: Data Types & Primary Keys [27, 28, 29] | Validating table schema specifications against technical software design docs [28]. |
| **3** | `mywork` | `ALTER TABLE employee_test ADD COLUMN email VARCHAR(50);` | DDL: Schema Migration [30, 42] | Verifying database migrations during feature releases (backward compatibility) [30]. |
| **4** | `mywork` | `INSERT INTO employee_test VALUES (999999, '2018-01-01', 'Smith', 'John', ...);` | DML: Single Record Insertion [32, 33] | Data seeding for edge-case bug reproduction in test environments [32, 34]. |
| **5** | `mywork` | `UPDATE employee_test SET salary = salary * 1.1;` | DML: Mass Data Update [34, 35] | Testing batch processing features and global data recalculation scripts [34]. |
| **6** | `UniversityDB` | `INSERT INTO EMP (empno, ename, job_title, ...) VALUES (...), (...);` | DML: Batch Insertion [33, 42] | Testing API endpoints that process bulk data payloads [33]. |
| **7** | `UniversityDB` | `UPDATE EMP SET ename = 'Smith' WHERE ename = 'Polk';` | DML: Targeted Record Update [35, 43] | Verifying user profile modification features via UI/API [35]. |
| **8** | `UniversityDB` | `ALTER TABLE EMP RENAME COLUMN job TO job_title;` | DDL: Column Renaming [42] | Testing database refactoring impact on backend services [42]. |
| **9** | `UniversityDB` | `DELETE FROM EMP WHERE ename = 'Roosevelt';` | DML: Conditional Row Deletion [35, 44] | Testing entity deletion endpoints (e.g., account removal) [35, 44]. |
| **10** | `UniversityDB` | `ALTER TABLE EMP DROP COLUMN comm;` | DDL: Dropping Columns [31, 45] | Verifying system behavior when legacy database fields are deprecated [45]. |
| **11** | `UniversityDB` | `ALTER TABLE EMP ADD CONSTRAINT fk_emp_dept FOREIGN KEY (deptno) REFERENCES dept(deptno);` | DDL: Foreign Key Constraints [46, 48] | Verifying database integrity enforcement when trying to insert orphaned records [47, 48]. |
| **12** | `ClassicModels` | Reverse Engineer ER Diagram (`Database -> Reverse Engineer`) | Metadata & Schema Visualization [49, 50] | Reverse engineering complex DB structures without documentation [49]. |
| **13** | `ClassicModels` | `SELECT * FROM information_schema.tables WHERE table_schema = 'classicmodels';` | Information Schema Metadata [51, 52] | Automated verification of table presence across test environments [51, 52]. |
| **14** | Any | `TRUNCATE TABLE table_name;` vs `DELETE FROM table_name;` | DDL vs DML Teardown [53, 54] | Optimizing test cleanup execution time in large automated test suites [53, 54]. |

---

## ⚠️ 5. Common Student Pitfalls & Live Troubleshooting

When reviewing homework or answering questions during the session, mentors should focus on these frequent student mistakes:

| Problem / Error Message | Root Cause | How Mentor Should Explain |
| :--- | :--- | :--- |
| **`Error Code: 1046. No database selected`** [11, 36] | Executing `CREATE TABLE` or `SELECT` without setting an active database context [11, 36]. | Point out the double-click action in MySQL Workbench schemas list or explain explicit qualification (`mywork.employee_test`) and `USE database_name;` [11, 36]. |
| **`Error Code: 1050. Table '...' already exists`** [38] | Running a `CREATE TABLE` script repeatedly without drop guards [38]. | Explain idempotency in QA test automation and show `CREATE TABLE IF NOT EXISTS` [38]. |
| **`Error Code: 1366. Incorrect integer value...`** [33] | Mismatch between listed columns and values in `INSERT INTO` (e.g., passing string into `INT`) [33]. | Teach students to align column lists vertically with `VALUES(...)` blocks to verify strict type parity [33]. |
| **Unintended Mass Update/Delete (`UPDATE` updated all 100 rows)** [34, 35] | Forgetting the `WHERE` clause in `UPDATE` or `DELETE` statements [34, 35]. | **QA Golden Rule:** Always run a `SELECT ... WHERE condition` *before* executing `UPDATE` or `DELETE` to preview affected rows [34, 35, 52]. |
| **`Error Code: 1215 / 1452. Cannot add foreign key constraint`** [47] | Trying to create a Foreign Key linking to a non-existent parent key or empty parent table [47]. | Explain referential integrity: parent primary keys must exist *before* child foreign keys can reference them [47, 48]. |
| **`TRUNCATE` fails on table with Foreign Keys** [53, 54] | FK constraints prevent fast truncation to protect child records [53, 54]. | Show how DDL integrity checks guard against data corruption, requiring dropping FK constraints or clearing child tables first [53, 54]. |

---

## ⏱️ 6. Timeboxed Review Session Scenario (50 Minutes)

### 📌 Phase 1: Warm-up & Curriculum Context (5 mins)
* Welcome students and locate Lesson 4 on the 10-lecture curriculum map [2].
* Emphasize the shift from read-only data querying to environment management (DDL/DML) [26, 32].

### 📌 Phase 2: Theory Check & Student Questions (15 mins)
* Address questions submitted from the lecture recording or homework [1, 26].
* Discuss key differences between `DROP`, `DELETE`, and `TRUNCATE` (use the comparison table) [31, 35, 53].
* Explain the role of `IF EXISTS` / `IF NOT EXISTS` in QA test automation [37, 38].

### 📌 Phase 3: Hands-on Live Coding & DB Construction (20 mins)
* **Step 1:** Create `mywork` database and `employee_test` table live with data types and primary key [26, 28, 29].
* **Step 2:** Execute `ALTER TABLE` (add email column, drop column, rename column) [30, 42, 45].
* **Step 3:** Perform single and batch `INSERT` into `UniversityDB.EMP` [33, 42].
* **Step 4:** Demonstrate safe `UPDATE` with `WHERE` and show how missing `WHERE` alters all records [34, 35].
* **Step 5:** Link `EMP` and `dept` via `FOREIGN KEY` constraint and demonstrate dependency errors [46, 47, 48].

### 📌 Phase 4: Schema Reverse Engineering & Metadata (5 mins)
* Open `ClassicModels` in MySQL Workbench and run `Reverse Engineer` to inspect ER diagrams [49].
* Query `information_schema.tables` to show metadata exploration [51, 52].

### 📌 Phase 5: Wrap-up & QA Mindset Reinforcement (5 mins)
* Summarize key takeaways: idempotency, transaction safety, parent-child record dependencies [37, 47].
* Announce Lesson 5 topic: Table Joins (`INNER JOIN`, `LEFT JOIN`) [2].

---

## 🎯 7. Mentor Checklist & Definition of Done (DoD)

By the end of the review session, ensure that students can:
- [ ] Articulate the differences between DDL (structure), DQL (retrieval), and DML (data rows) [26, 31, 32].
- [ ] Safely write `CREATE DATABASE` and `DROP TABLE` statements using `IF EXISTS` / `IF NOT EXISTS` [36, 37, 38].
- [ ] Construct `CREATE TABLE` queries with appropriate data types (`INT`, `VARCHAR`, `DECIMAL`, `DATE`) and Primary Keys [27, 28, 29].
- [ ] Modify existing tables using `ALTER TABLE` (`ADD COLUMN`, `DROP COLUMN`, `RENAME COLUMN`) [30, 42, 45].
- [ ] Perform single and batch `INSERT` queries adhering to strict column-value alignment [33, 42].
- [ ] Safely execute `UPDATE` and `DELETE` queries with mandatory `WHERE` clause checks [34, 35, 43].
- [ ] Contrast `DROP`, `DELETE`, and `TRUNCATE` in performance, language category, and QA testing scenarios [31, 35, 53].
- [ ] Explain Primary/Foreign key constraints and the dependency order when seeding parent/child tables [46, 47, 48].
- [ ] Generate ER diagrams via MySQL Workbench Reverse Engineering [49].
