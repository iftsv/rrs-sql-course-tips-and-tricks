# [SQL-A] 5. (09/23) Table Joins — Review Session Guide

**Pedagogical Guide for Mentors and Reviewers**
**Course:** SQL for QA Automation / QA Engineer
**Session Topic:** Table Joins (INNER JOIN, LEFT JOIN, SELF JOIN, FULL JOIN), Homework Review (MyWork & UniversityDB), Practical Work with ClassicModels and UniversityDB Databases.

---

## 🧭 1. Perspective: 10-Lecture Curriculum Map

Remind students of their current position on the course curriculum map. This helps summarize knowledge and illustrates how mastering table joins prepares the foundation for complex analytical validations, data aggregation, and end-to-end (E2E) backend testing.

*   **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary and Foreign Keys, Data Integrity.
*   **Lecture 2:** Simple Queries on a Single Table — Data Selection, Filtering, Sorting, Result Limiting, and Metadata.
*   **Lecture 3/4:** SQL Components — Syntax & Practice of DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation).
*   **Lecture 5 (Current):** Table Joins (INNER JOIN, LEFT JOIN, SELF JOIN, FULL JOIN).
*   **Lecture 6:** Grouping, Unions, and Subqueries (GROUP BY, UNION / UNION ALL, Subqueries).
*   **Lecture 7:** Built-in Functions (String, Numeric, Date & Time Functions).
*   **Lecture 8:** Temporary Tables and Views (Views vs. Temp Tables).
*   **Lecture 9:** Data Warehouse Concepts (DWH, Fact and Dimension Tables).
*   **Lecture 10:** Advanced Topics (Triggers, Stored Functions) and Interview Preparation.

---

## 🕵️‍♂️ 2. Session Concept: Combining Data Across Tables & QA Testing Context

In previous sessions, students learned single-table querying and schema modification. In real-world software development, data is normalized across dozens of interconnected tables.

**Core QA Objectives when Testing JOIN Queries:**
*   **End-to-End Business Flow Validation (E2E Integration Testing):** Verifying relations across orders, customers, payments, and products (e.g., verifying if a payment is correctly linked to the corresponding customer and order).
*   **Orphaned Data & Integrity Checks:** Identifying entities lacking required foreign key relationships (e.g., customers without an assigned sales representative or products without a category).
*   **Boundary & NULL Testing:** Utilizing `LEFT JOIN` to uncover missing foreign key values or unexpected NULL records.
*   **Hierarchy Unfolding:** Leveraging `SELF JOIN` to test role permission models and reporting structures ("employee — manager").

---

## 🧱 3. Theoretical Deep Dive: Table Joins in QA Context

### 1. Homework Review (Lecture 4)

Before diving into Joins, the mentor conducts a quick review of the Lecture 4 homework solution across two parts:

#### Part 1: `MyWork` Database
1. **Adding the `country` Column:**
   ```sql
   ALTER TABLE department ADD COLUMN country VARCHAR(50);
   ```
2. **Renaming `location` to `city`:**
   ```sql
   ALTER TABLE department RENAME COLUMN location TO city;
   ```
3. **Batch Insert of New Departments:**
   ```sql
   INSERT INTO department (deptno, dname, city) 
   VALUES 
   (5, 'HR', 'Chicago'), 
   (6, 'Engineering', 'Boston'), 
   (7, 'Marketing', 'Dallas');
   ```
   *💡 QA Tip:* Emphasize strict column-to-value positional and data type alignment in the `VALUES` clause.
4. **Locating Department in Atlanta:**
   ```sql
   SELECT deptno, dname, city FROM department WHERE city = 'Atlanta';
   ```
   *(Result: Dept #3 — Sales)*.

#### Part 2: `UniversityDB` Database
1. **Schedule in Room 306:**
   ```sql
   SELECT * FROM classes WHERE room = '306';
   ```
2. **Credits for Algebra Course:**
   ```sql
   SELECT DISTINCT credits FROM courses WHERE course_name = 'Algebra';
   ```
   *(Result: 3 credits)*.
3. **Building for English Department:**
   ```sql
   SELECT building FROM departments WHERE dept_name = 'English';
   ```
   *(Result: Humanities Hall)*.
4. **Enrollment Records for August 22, 2024:**
   ```sql
   SELECT * FROM enrollments WHERE enroll_date = '2024-08-22';
   ```
5. **Distinct Letter Grades:**
   ```sql
   SELECT DISTINCT grade FROM enrollments;
   ```
   *(Result: 'A' and 'B' grades)*.
6. **Instructor Alla Johnson's Email:**
   ```sql
   SELECT email FROM instructors WHERE first_name = 'Alla' AND last_name = 'Johnson';
   ```
7. **Start and End Dates of Last Semester:**
   ```sql
   SELECT * FROM semesters ORDER BY semester_id DESC;
   ```
8. **Contact Details for Student Michael Jordan:**
   ```sql
   SELECT * FROM students WHERE first_name = 'Michael' AND last_name = 'Jordan';
   ```

---

### 2. Core JOIN Types

#### A. INNER JOIN (or simply JOIN)
*   **Concept:** Returns only rows where the join condition (`ON`) evaluates to TRUE in **both** tables (intersection of sets).
*   **Syntax:**
    ```sql
    SELECT e.empno, e.ename, d.dname
    FROM EMP e
    JOIN dept d ON e.deptno = d.deptno;
    ```
*   **Key Note:** The `INNER` keyword is optional and typically omitted in daily development and test scripts.

#### B. LEFT JOIN (LEFT OUTER JOIN)
*   **Concept:** The table specified first (on the left, after `FROM`) is primary. Returns **all** rows from the left table. If no matching record exists in the right table, right-side columns are filled with `NULL`.
*   **Syntax:**
    ```sql
    SELECT c.customerName, e.firstName, e.lastName
    FROM customers c
    LEFT JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber;
    ```
*   **QA Superpower:** Filtering via `WHERE right_table.key IS NULL` allows testers to identify unlinked entities and data integrity bugs.

#### C. SELF JOIN (Joining a Table to Itself)
*   **Concept:** A table is joined with itself using two distinct aliases. Used to unfold hierarchical relationships stored within a single table.
*   **Syntax:**
    ```sql
    SELECT 
        e.employeeNumber AS emp_id,
        CONCAT(e.firstName, ' ', e.lastName) AS employee_name,
        CONCAT(m.firstName, ' ', m.lastName) AS manager_name
    FROM employees e
    LEFT JOIN employees m ON e.reportsTo = m.employeeNumber;
    ```
*   **Purpose:** Elevates the hierarchical relationship ("employee — manager") onto a single flat row for easy inspection.

#### D. FULL JOIN & CROSS JOIN
*   **FULL OUTER JOIN:** Returns all records from both tables, filling `NULL` for missing matches on either side (emulated in MySQL via `UNION` of `LEFT JOIN` and `RIGHT JOIN`). Rarely used in routine QA practice.
*   **CROSS JOIN:** Cartesian product (combines every row of the first table with every row of the second). Used primarily for generating combinatorial test data sets.

---

## 🔮 4. QA Superpower & Student Questions from Lecture

During the Lecture 5 theory presentation, students asked practical questions. Mentors should re-address them during the review session:

### ❓ Question 1: "Why did INNER JOIN return 100 records when the customers table has 122 records?"
*   **Context:** Joining `customers` (122 rows) and `employees` (23 rows) on `salesRepEmployeeNumber` via `INNER JOIN` yielded 100 rows.
*   **Mentor Explanation:** `INNER JOIN` drops rows where the join condition is not met. 22 customers have `NULL` in `salesRepEmployeeNumber` (no assigned sales representative).
*   **Verification:** Run `LEFT JOIN` with `WHERE e.employeeNumber IS NULL`. The query outputs exactly 22 customers.

### ❓ Question 2: "Can records be excluded from INNER JOIN for reasons other than NULL?"
*   **Mentor Explanation:** Yes! If the left table contains a foreign key value (e.g., `salesRepEmployeeNumber = 1800`), but no record in `employees` has `employeeNumber = 1800` (an invalid/broken foreign key reference).
*   **QA Takeaway:** `INNER JOIN` silently hides these records, whereas `LEFT JOIN` highlights them (showing `1800` on the left and `NULL` for manager fields on the right), revealing a critical data integrity bug.

### ❓ Question 3: "Which table should be placed on the LEFT in a LEFT JOIN?"
*   **Mentor Answer:** Place the **primary target table** on the left (first) — the table whose records you must preserve completely without losing data.

### ❓ Question 4: "Why do we need a SELF JOIN if all data is already inside one table?"
*   **Mentor Answer:** The `employees` table only stores the manager's ID (`reportsTo = 1002`). To retrieve the manager's full name alongside the employee in the same row, we join `employees` to itself, mapping `reportsTo` to `employeeNumber` in the second copy.

---

## 💻 5. Practical Script: Live Queries & Hands-on Tasks

Below is a comprehensive breakdown of practical queries from Lecture 5 and hands-on exercises, analyzed through a QA testing lens.

| # | Database | SQL Statement / Operation | Target Concept | 💡 QA Testing Context |
|---|---|---|---|---|
| **1** | `MyWork` | `ALTER TABLE department ADD COLUMN country VARCHAR(50);` | DDL Schema Update | Validating schema migrations and column additions. |
| **2** | `MyWork` | `ALTER TABLE department RENAME COLUMN location TO city;` | DDL Column Rename | Validating schema refactoring and backward compatibility. |
| **3** | `MyWork` | `INSERT INTO department VALUES (5, 'HR', 'Chicago', 'USA'), ...;` | DML Batch Insert | Seeding baseline test data (Data Seeding) before test runs. |
| **4** | `MyWork` | `SELECT deptno, dname, city FROM department WHERE city = 'Atlanta';` | DQL Single Table | Validating filtered entity attributes. |
| **5** | `UniversityDB` | `SELECT DISTINCT credits FROM courses WHERE course_name = 'Algebra';` | DQL Aggregation/Distinct | Validating unique business rules for course configurations. |
| **6** | `UniversityDB` | `SELECT * FROM semesters ORDER BY semester_id DESC;` | DQL Sorting | Validating sorting and retrieving current/active semester records. |
| **7** | `ClassicModels` | `SELECT * FROM employees e JOIN offices o ON e.officeCode = o.officeCode;` | INNER JOIN (100% Match) | Validating 100% referential integrity: all 23 employees link to valid offices. |
| **8** | `ClassicModels` | `SELECT * FROM customers c JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber;` | INNER JOIN (Partial Match) | Yields 100 rows out of 122. Uncovers customers without assigned sales reps. |
| **9** | `ClassicModels` | `SELECT * FROM customers c LEFT JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber;` | LEFT JOIN (Full Coverage) | Returns all 122 customers, guaranteeing no primary entity data loss. |
| **10** | `ClassicModels` | `SELECT * FROM customers c LEFT JOIN employees e ON c.salesRepEmployeeNumber = e.employeeNumber WHERE e.employeeNumber IS NULL;` | LEFT JOIN + IS NULL Filter | **QA Superpower:** Pinpointing the 22 unassigned customers to file a bug report. |
| **11** | `ClassicModels` | `SELECT * FROM orders o JOIN customers c ON o.customerNumber = c.customerNumber;` | INNER JOIN (Transaction Match) | Yields 326 orders, verifying that every order links to a valid customer. |
| **12** | `ClassicModels` | `SELECT * FROM orders o JOIN orderdetails od ON o.orderNumber = od.orderNumber;` | INNER JOIN (Order Items) | Yields 2996 line items, validating shopping cart contents and receipts. |
| **13** | `ClassicModels` | `SELECT * FROM products p JOIN productlines pl ON p.productLine = pl.productLine;` | INNER JOIN (Catalog Match) | Yields 110 products, verifying catalog taxonomy integrity. |
| **14** | `ClassicModels` | `SELECT * FROM customers c JOIN payments p ON c.customerNumber = p.customerNumber;` | INNER JOIN (Financial Match) | Yields 273 payment transactions, validating billing logs and account statements. |
| **15** | `ClassicModels` | `SELECT e.employeeNumber, e.lastName, m.lastName AS manager FROM employees e LEFT JOIN employees m ON e.reportsTo = m.employeeNumber;` | SELF JOIN (Hierarchy) | Unfolding manager-employee hierarchy (President has `manager IS NULL`). |

---

## ⚠️ 6. Common Student Pitfalls & Live Troubleshooting

| Problem / Symptom | Root Cause | How Mentor Should Explain Fix |
|---|---|---|
| **Error Code: 1052. Column '...' in field list is ambiguous** | Selecting a column (e.g., `employeeNumber`) present in both tables without specifying a table alias. | Explain the requirement to explicitly qualify columns with aliases: `c.customerNumber` or `e.employeeNumber`. |
| **Missing customer records when joining customers & employees** | Using `INNER JOIN` instead of `LEFT JOIN` when NULL foreign keys are present. | Demonstrate the difference between strict intersection (`INNER`) and preserving left table integrity (`LEFT`). |
| **Swapped table positions in LEFT JOIN** | Placing the secondary lookup table first after `FROM`. | Teach the golden rule: the left table represents the **primary entity that must never be dropped**. |
| **Confusing ON vs WHERE clauses** | Placing business logic filter predicates inside `ON` instead of `WHERE`. | Explain separation of concerns: `ON` defines table linking criteria, while `WHERE` filters the joined dataset. |

---

## ⏱ 7. Timeboxed Review Session Scenario (50 Minutes)

### 📌 Phase 1: Warm-up & Curriculum Context (5 mins)
*   Welcome students and position Lesson 5 on the 10-lecture curriculum map.
*   Highlight the shift from single-table queries to integration testing across normalized tables.

### 📌 Phase 2: Homework Review (10 mins)
*   Quick walk-through of `MyWork` homework (ALTER, RENAME, INSERT, SELECT).
*   Review `UniversityDB` queries (schedules, credits, grades, semesters).

### 📌 Phase 3: Theory Deep-Dive & Student Questions (15 mins)
*   Interactive review of JOIN types: `INNER JOIN` vs `LEFT JOIN` vs `SELF JOIN`.
*   Address lecture questions (100 vs 122 rows, finding `NULL` records, `SELF JOIN` logic).

### 📌 Phase 4: Hands-on Live Coding on ClassicModels (15 mins)
*   Demonstrate `customers JOIN employees` (100 rows) vs `customers LEFT JOIN employees` (122 rows).
*   Locate 22 unassigned customers via `WHERE e.employeeNumber IS NULL`.
*   Construct `SELF JOIN` on `employees` table to inspect organizational managers.

### 📌 Phase 5: Wrap-up & Next Topic Preview (5 mins)
*   Summarize key takeaways: JOIN selection criteria and orphan detection.
*   Announce Lesson 6 topic: Grouping (`GROUP BY`), aggregate functions, `UNION`, and subqueries.

---

## 🎯 8. Mentor Checklist & Definition of Done (DoD)

By the end of the review session, students should be able to:
*   [ ] Articulate differences between `INNER JOIN`, `LEFT JOIN`, and `SELF JOIN`.
*   [ ] Explain why `INNER JOIN` filters out rows with `NULL` or invalid foreign keys.
*   [ ] Use `LEFT JOIN ... WHERE right_key IS NULL` to detect orphaned data and bugs.
*   [ ] Apply table aliases correctly to avoid `Column is ambiguous` errors.
*   [ ] Construct `SELF JOIN` queries to flatten hierarchical structures.
*   [ ] Write SQL queries for E2E validation of multi-table database workflows.
