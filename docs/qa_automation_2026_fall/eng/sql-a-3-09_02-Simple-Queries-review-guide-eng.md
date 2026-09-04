# [SQL-A] 3. (09/02) Simple Queries - Review Session Guide

This methodological guide is designed for mentors and reviewers. Its purpose is to help you lead an interactive review session to reinforce the material from the second lecture, which focuses on **simple SQL queries (Simple Queries)**. This guide includes the course lecture roadmap, a breakdown of key syntactic structures with QA-specific context, techniques for exploring databases "blindly" using system metadata, common student technical issues, and a step-by-step interactive session script.

---

### 🧭 1. Perspective: 10-Lecture Course Map
Remind students of their current position on the educational map. This helps reduce anxiety about the volume of material and demonstrates how simple queries lay the groundwork for complex database operations down the line [12].

*   **Lecture 1:** Introduction to Relational Databases (RDBMS), tables, rows, columns, Primary and Foreign Keys, and data integrity [12].
*   **Lecture 2 (Current):** **Simple Queries on a Single Table** — data retrieval, filtering, sorting, limiting outputs, and working with database metadata [12].
*   **Lecture 3:** SQL Components (syntax breakdown of DDL, DML, DQL) [12].
*   **Lecture 4:** Joining Tables (Inner Join, Left Join, Self Join) [12].
*   **Lecture 5:** Grouping, Unions, and Subqueries (Group By, Union/Union All, Subqueries) [12].
*   **Lecture 6:** Built-in Functions (Text, Numeric, Date/Time functions) [12].
*   **Lecture 7:** Temporary Tables and Views (Views vs. Temp Tables) [12].
*   **Lecture 8:** Data Warehousing Concepts (Data Warehouse, Dimension vs. Fact tables) [12].
*   **Lecture 9:** SQL in Reporting [12].
*   **Lecture 10:** Advanced Topics (Triggers, Functions) and Interview Preparation [12].

---

### 🕵️‍♂️ 2. Session Concept: Database as a "Black Box"
While the first session decomposed database structures using the familiar analogy of a retail store receipt [13], the core focus of this second session shifts to **exploring an unfamiliar system**.

In real-world projects, QA Engineers frequently face "black box" scenarios:
*   Database documentation is completely missing or outdated.
*   Access to visual ER diagram generation tools in the production/staging environment is restricted [46, 55].
*   Only a terminal or console client is available to execute raw SQL queries.

The ability to write simple data retrieval queries (DQL) and interact with system tables is a tester's "flashlight." It allows you to rapidly navigate an unfamiliar database, reverse-engineer its structure, and begin testing the backend effectively from day one on the project [46].

---

### 🔢 3. Counting Records and Limits: Why It Matters for QA

#### Three Ways to Count Records in MySQL Workbench [3, 43]:
1.  **Using `SELECT COUNT(*)` (The Most Reliable Method)** [3, 43]:
    ```sql
    SELECT COUNT(*) FROM classicmodels.customers;
    ```
    *   **Why it's a QA superpower:** It instantly yields an exact count of records without requiring manual scrolling [43]. Unlike GUI tools, this query returns a single integer, which is easy to parse and verify in automation test scripts.
    *   **💡 QA Use Case (Migration and Integration):** When testing data migration from an old legacy CRM to a new database, a QA engineer must reconcile the record counts. If the source system contained 122 customers, `SELECT COUNT(*)` in the target database must return exactly 122 [2, 3]. A discrepancy of even one record points to data loss during the ETL process.

2.  **Analyzing Action Output (The Log Panel)** [29, 30]:
    *   Executing a standard `SELECT * FROM table;` displays the count of returned rows in the lower Workbench log panel (e.g., *122 rows returned*) [29].
    *   *QA Pitfall:* If Workbench has a forced limit configured in its global settings (e.g., *Limit to 1000 rows*) and the table contains a million rows, the log panel will only show the limit, which can easily mislead a tester into thinking the migration was incomplete [42].

3.  **The Metadata Information Icon (Hover over "i" icon)** [3, 66]:
    *   In the Schemas sidebar, hovering over a table reveals an "i" icon. Clicking it shows table metadata, including the estimated row count (`Table rows`) [3, 66].
    *   *QA Pitfall:* This number is cached by the storage engine (especially in InnoDB) and is frequently a rough estimate. For strict validation checks, always rely on `COUNT(*)` [3].

#### Restricting Query Outputs (LIMIT) [2, 38]:
Different RDBMS use distinct syntax structures to restrict the number of returned rows [2, 38]:
*   **MySQL:** `LIMIT 10;` [2, 38]
*   **SQL Server:** `TOP 10` [2, 38]
*   **Oracle:** `WHERE rownum = 10` or `FETCH FIRST 10 ROWS ONLY` [2, 38]

*   **💡 QA Use Case (Pagination and Performance):** 
    When testing web-page pagination (such as displaying 10 products per page), a QA engineer must verify that the backend does not fetch the entire database at once. A query with `LIMIT 10` (coupled with `OFFSET` in production projects) ensures fast page rendering. During API testing, we verify that the backend sends an optimized database query with a limit to avoid system-wide performance degradation and server memory leaks.

---

### 📋 4. Query Structure: Anatomy of a Simple Select Statement

The strict evaluation and writing order of SQL instructions for a single table is [45]:
```sql
SELECT column1, column2  -- WHAT to retrieve (columns)
FROM table_name          -- WHERE to retrieve from (table)
WHERE condition          -- HOW to filter (filtering clause)
ORDER BY column_name     -- HOW to sort (ordering clause)
LIMIT N;                 -- HOW MANY rows to return (limiting clause) [45]
```

#### Key Syntax Rules for Students [4, 40, 41]:
*   **Case Insensitivity:** SQL is case-insensitive (`SELECT` and `select` execute identically), but writing reserved SQL keywords in uppercase is highly recommended for readability [40].
*   **Reserved Keywords:** These are highlighted in blue by the query editor (e.g., `SELECT`, `FROM`, `STATUS`) [40, 41].
*   **Backticks (`` ` ``):** If a table or column name clashes with an SQL reserved word (e.g., a table named `` `select` `` or a column named `` `status` ``), you must wrap the identifier in backticks (tilde `~` key on English keyboards). This instructs the DBMS that it is an identifier, not a functional command [41, 62].
*   **Semicolon (`;`):** Acts as a query separator (the Commit command for Workbench) [40]. It signals the DBMS that the current execution block is complete and it can safely proceed to the next [40].
*   **Quotes for Literals:** Text values and date strings inside the `WHERE` condition must always be wrapped in **single quotes** (`country = 'Norway'`), whereas numbers must remain **unquoted** (`creditLimit >= 50000`) [4, 59].
*   **Session Shortcut:** The `USE database_name;` command sets the active database schema for the session [4, 44]. This allows students to omit the database prefix in their queries (`customers` instead of `classicmodels.customers`) [4, 44].

---

### 🔮 5. QA Superpower: Working "Blindly" (INFORMATION_SCHEMA and DESC)

When a tester has no physical access to visual ER diagrams, they extract structure directly from metadata [46, 47].

#### 1. Inspecting Table Schema via `DESCRIBE` [6, 52]:
Executing `DESC table_name;` (or the full `DESCRIBE`) returns the exact structure of a table: fields, data types (VARCHAR, INT, DATE), nullability, indexes (Primary Key as PRI, Multiple Key as MUL), and default values [6, 52].
```sql
DESC classicmodels.customers;
```
*   **Why it's a QA superpower:** It instantly reveals field constraints (e.g., the maximum string length of a VARCHAR column). This allows a QA engineer to design Boundary Value Analysis (BVA) test cases without waiting for API documentation or consulting developers.

#### 2. Querying the System Catalog `INFORMATION_SCHEMA` [4, 5, 47]:
`INFORMATION_SCHEMA` is a built-in virtual database containing metadata about all objects across the database server [4, 47].

*   **Retrieve all tables and their exact row counts in a single query [5, 49]:**
    ```sql
    SELECT TABLE_NAME, TABLE_ROWS 
    FROM INFORMATION_SCHEMA.tables 
    WHERE TABLE_SCHEMA = 'classicmodels';
    ```
    *This eliminates the need to write separate `COUNT(*)` queries for each of the 8+ individual tables [49].*

*   **Extract all 59 database columns along with their exact data types [5, 49]:**
    ```sql
    SELECT TABLE_NAME, COLUMN_NAME, ORDINAL_POSITION, COLUMN_TYPE, COLUMN_KEY 
    FROM INFORMATION_SCHEMA.columns 
    WHERE TABLE_SCHEMA = 'classicmodels' 
    ORDER BY TABLE_NAME, ORDINAL_POSITION;
    ```
    *   **💡 QA Use Case:** Using the output of this query, a QA engineer can copy-paste the metadata into Excel and fully reconstruct the relational schema of the database, even if the "Reverse Engineer" visualization tool is locked down by permissions [50, 51, 55]!

*   **Verify Primary and Foreign Key Constraints [5, 47]:**
    ```sql
    SELECT * \n    FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE \n    WHERE TABLE_SCHEMA = 'classicmodels' AND TABLE_NAME = 'customers';
    ```
    *The output clearly shows that `customerNumber` is the Primary Key (PRI) and that the table links back to employees via the Foreign Key mapped to `salesRepEmployeeNumber` [48].*

#### ⚠️ Interview Trap: The Dual Meaning of `DESC`
Explain to students that the keyword `DESC` in MySQL has two completely different meanings depending on its context:
1.  **DESC as DESCRIBE (Table Structure):** Placed *before* a table name. Example: `DESC customers;` — outputs the table schema (columns, types) [6, 52].
2.  **DESC as DESCENDING (Sorting Order):** Placed *after* a column name inside the sorting block. Example: `ORDER BY score DESC;` — sorts values from highest to lowest (or Z to A) [2, 39].

---

### 🎯 6. Practical Trainer: Query Analysis (ClassicModels)

Below is a curated breakdown of core educational queries from Lecture 2, analyzed through the lens of QA testing tasks [6, 7, 8].

| SQL Query | What It Does | 💡 QA Context: Why It Matters for Testers |
| :--- | :--- | :--- |
| `SELECT * FROM customers WHERE country = 'Norway';` [6] | Retrieves all customers located in Norway [6]. | **Localization and Tax Validation:** Verifies that delivery fees, localized shipping terms, and Norwegian VAT (MVA) are computed accurately by the backend system. |
| `SELECT * FROM customers WHERE creditLimit BETWEEN 50000 AND 60000;` [6] | Selects customers with a credit limit from $50k to $60k [6]. | **Boundary Value Analysis (BVA):** Identifies exact test accounts located precisely on equivalence class boundaries ($50,000, $60,000, and inside the range) to validate discount logic. |
| `SELECT * FROM employees WHERE jobTitle LIKE '%VP%';` [7] | Retrieves all employees with "VP" in their job title [7, 58]. | **UI Search Feature Testing:** Tests the application's partial text search engine. The `%` wildcards represent any characters preceding and following "VP" [58, 59]. |
| `SELECT city, phone FROM offices WHERE city IN ('San Francisco', 'Boston');` [7] | Returns only the city and phone number for SF and Boston offices [7, 59]. | **Multi-select Filter Testing:** When a user selects multiple checkbox options in a UI filter, the backend translates this input into an `IN (...)` operator rather than a series of cumbersome `OR` statements [59, 60]. |
| `SELECT * FROM offices WHERE state IS NULL;` [7] | Finds offices where the state field is empty [7, 60]. | **Optional Field Validation:** While US offices require the `state` field, international offices (e.g., Japan, France) leave it empty (`NULL`) [60]. QA verifies that the backend handles missing data gracefully without throwing a `NullPointerException`. |
| `SELECT orderNumber, quantityOrdered FROM orderdetails ORDER BY quantityOrdered DESC;` [7] | Returns order details sorted by quantity ordered, from largest to smallest [7]. | **UI Sorting Validation:** Verifies product sorting order in an admin console. The tester matches the sequence of items on the UI screen with the database order retrieved using descending sort [7, 61]. |
| `SELECT DISTINCT status FROM orders;` [8] | Displays all unique order statuses in the system [8, 62]. | **State Transition Testing:** To test lifecycle transitions (e.g., In Process -> Shipped -> Resolved), a QA must first query `DISTINCT status` to discover every status state currently defined in the DB [61, 62]. |

---

### 🔍 7. Troubleshooting Typical Student Errors (Review Case!)

Students often run into these common pitfalls when working on homework assignments. Address these proactively during the review [68].

#### 1. Pitfall: "I ran the `universitydb.sql` database creation script, the log showed success, but the database is missing from the Schemas list!" [65, 68]
*   **Root Cause:** MySQL Workbench does not auto-refresh the Schemas tree sidebar in real-time when schemas are created via external scripts [68].
*   **Mentor's Guide:** Demonstrate that they need to right-click anywhere in the empty white space of the Schemas sidebar on the left and select **`Refresh All`** [68]. The `universitydb` schema will instantly appear in the list [65, 68].

#### 2. Pitfall: Confusing the Workbench UI Row Limit with the SQL `LIMIT` Clause [38, 42]
*   **Root Cause:** The upper toolbar in Workbench has a dropdown limit option (e.g., "Limit to 1000 rows") [42]. Some students assume that because of this setting, the DBMS always restricts outputs to that row count [42].
*   **Mentor's Guide:** Explain that the Workbench UI limit is purely local. It prevents your local PC memory from crashing when rendering large datasets [42]. However, in test automation code or real backend services, no such "Workbench Limit" exists. If a query does not explicitly specify a `LIMIT` clause, the database will attempt to return millions of rows, potentially causing an Out Of Memory (OOM) error. Reinforce writing `LIMIT N` directly in the query code [38]!

#### 3. Pitfall: Syntax Errors due to Incorrect Clause Ordering [45]
*   Students frequently write `LIMIT` before `ORDER BY`, or place the `WHERE` filter before the `FROM` clause.
*   **Mentor's Guide:** Introduce a simple mnemonic rule to memorize keyword ordering: **S**ingle **C**ats **F**ly **T**o **W**est **C**oast (**S**elect, **C**olumn, **F**rom, **T**able, **W**here, **C**ondition) [4]. The strict order of clauses must always be: `SELECT` -> `FROM` -> `WHERE` -> `ORDER BY` -> `LIMIT` [45]. Any violation yields a standard syntax error.

#### 4. Pitfall: Using `=` Instead of `LIKE` with Wildcards `%` [4, 58]
*   A query containing `WHERE jobTitle = '%VP%'` returns zero rows.
*   **Mentor's Guide:** Clarify that the `=` operator searches for a literal, exact match (i.e., a string literally containing a percent symbol, a V, a P, and another percent symbol). To utilize wildcard pattern matching (`%`), they must use the `LIKE` operator [58].

---

### 🎓 Reviewer Checklist: How to Conduct an Active Session (50 minutes)

*   **[ ] Icebreaker & Technical Health Check (5 minutes):**
    *   Confirm if everyone successfully imported the `classicmodels` database [29].
    *   Ask: "Did anyone encounter the 'disappearing' `universitydb` database after running the script, and how did you resolve it?" (Tests their knowledge of `Refresh All`) [65, 68].
*   **[ ] Interactive Game: "The Blind Tester" (10 minutes) [46, 52]:**
    *   *Scenario:* Show an empty SQL console. Tell the students: "Imagine you've just joined a project. There's no documentation, no ER diagram, and you have no visual schema tools [46, 55]. We need to test the `customers` table. How do we find out what fields exist, what their data types are, and how many records are in this table?"
    *   *Expected Answer:* Use `DESC customers;` to inspect fields and nullability [6, 52], run `SELECT COUNT(*) FROM customers;` to count rows [3, 43], or query `INFORMATION_SCHEMA.columns` / `tables` for comprehensive metadata [5].
*   **[ ] Spotlight Quiz: Dual Meanings of DESC (5 minutes):**
    *   Display two distinct queries:
        1. `DESC employees;` [6, 52]
        2. `SELECT * FROM employees ORDER BY employeeNumber DESC;` [2, 39]
    *   Have students explain the functional difference of the keyword `DESC` in each query (Table structure vs. Descending sort) [39, 52].
*   **[ ] Live Homework Blitz (20 minutes) [9]:**
    *   Have students write queries in the shared chat or on a shared screen, calling on them one by one:
        *   *Task 1:* Retrieve all customers located in Australia (conceptually identical to Norway in the lecture) [9, 57, 65].
        *   *Task 2:* Locate the company's President (tests filtering using `WHERE jobTitle = 'President'` or `LIKE`) [9, 58].
        *   *Task 3:* Calculate the total number of Sales Representatives employed by the company [9] (expects `SELECT COUNT(*)` coupled with a job title filter [3, 43]).
        *   *Task 4:* List all unique product lines offered in the shop, sorted alphabetically [8] (expects `SELECT DISTINCT ProductLine FROM products ORDER BY ProductLine;` [8]).
*   **[ ] UniversityDB Exploration & Q&A (10 minutes) [10, 65, 66]:**
    *   Review the second part of the homework assignment: importing the University database [10, 65].
    *   Show how to count rows across all tables in a single sweep using `INFORMATION_SCHEMA` instead of executing eight separate `COUNT(*)` queries [5, 49].
    *   Answer any remaining syntax or administrative questions before wrapping up.
