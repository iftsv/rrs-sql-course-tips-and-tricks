# [SQL-A] 6. (09/30) Group By, Union, Sub Queries — Review Session Guide

**Pedagogical Guide for Mentors and Reviewers**  
**Course:** SQL for QA Automation / QA Engineer  
**Session Topic:** Data Grouping (GROUP BY, HAVING, Aggregate Functions), Set Operations (UNION / UNION ALL), Subqueries & Common Table Expressions (CTE / WITH), Homework Review (ClassicModels & UniversityDB).

---

#### 🧭 1. Perspective: 10-Lecture Curriculum Map
Remind students of their current position on the course curriculum map. This helps summarize knowledge and illustrates how mastering data aggregation, set unification, and nested queries prepares the foundation for testing analytical dashboards, financial reconciliation, integration workflows, and complex database validations.

* **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary and Foreign Keys, Data Integrity.
* **Lecture 2:** Simple Queries on a Single Table — Data Selection, Filtering, Sorting, Result Limiting, and Metadata.
* **Lecture 3/4:** SQL Components — Syntax & Practice of DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation).
* **Lecture 5:** Table Joins (INNER JOIN, LEFT JOIN, SELF JOIN, FULL JOIN).
* **Lecture 6 (Current):** Grouping, Unions, and Subqueries (GROUP BY, HAVING, UNION / UNION ALL, Subqueries, CTE).
* **Lecture 7:** Built-in Functions (String, Numeric, Date & Time Functions).
* **Lecture 8:** Temporary Tables and Views (Views vs. Temp Tables).
* **Lecture 9:** Data Warehouse Concepts (DWH, Fact and Dimension Tables).
* **Lecture 10:** Advanced Topics (Triggers, Stored Functions) and Interview Preparation.

---

#### 🕵️‍♂️ 2. Session Concept: Data Aggregation, Set Operations & QA Testing Context
In the previous session, students learned how to join tables. In Lesson 6, we move to analyzing and summarizing large volumes of data. In daily QA practice, groupings, subqueries, and set operations are fundamental tools for automated and manual Data Reconciliation.

**Core QA Objectives when Testing GROUP BY, UNION & Subqueries:**
* **Business Metric & Analytics Validation (KPI & Report Testing):** Verifying backend aggregated indicators (total revenue, average checks, active user counts by region) against UI dashboards and analytics exports.
* **Multi-Tenant & Data Merging Validation:** Utilizing `UNION` and `UNION ALL` to validate data migrations, generate deduplicated mailing lists, and identify lost records during multi-brand DB mergers.
* **Threshold & Boundary Testing (HAVING Clause):** Identifying anomalous customer segments or transactions (e.g., customers with total spending > $70,000) to test edge-case business logic.
* **Isolated Data Selection (Subqueries & CTEs for Bug Hunting):** Extracting ID lists via subqueries to pass into primary queries for boundary testing and regression verification.

---

#### 🧱 3. Theoretical Deep Dive: Group By, Union, Subqueries & CTEs in QA Context

##### 1. Homework Review (Lecture 5 — ClassicModels 1–9 & UniversityDB)
Before introducing new theory, the mentor conducts a quick review of the Lecture 5 homework solutions across two parts:

###### Part 1: ClassicModels Database (Queries 1–9)
1. **Locating Vendor for 1966 Shelby Cobra:**
   ```sql
   SELECT * FROM products WHERE productName LIKE '%1966 Shelby Cobra%';
   ```
2. **Most and Least Expensive Product (MSRP):**
   * *Most Expensive:* `SELECT * FROM products ORDER BY MSRP DESC LIMIT 1;`
   * *Least Expensive:* `SELECT * FROM products ORDER BY MSRP ASC LIMIT 1;`
   * *💡 QA Tip:* Mention that `MIN(MSRP)` / `MAX(MSRP)` aggregate functions can also achieve this result.
3. **Product with Highest Quantity in Stock:**
   ```sql
   SELECT * FROM products ORDER BY quantityInStock DESC LIMIT 1;
   ```
   *(Result: Suzuki Xreo, ~10,000 in stock)*.
4. **Products with Quantity in Stock Less Than 20:**
   ```sql
   SELECT * FROM products WHERE quantityInStock < 20;
   ```
   *(Result: 1 product with 15 units)*.
5. **Customers with Highest and Lowest Credit Limit:**
   * *Highest:* `SELECT * FROM customers ORDER BY creditLimit DESC LIMIT 1;` *(Customer #141, limit $227,600)*.
   * *Lowest:* `SELECT * FROM customers ORDER BY creditLimit ASC LIMIT 1;` *(Limit $0)*.
6. **Highest Single Payment Check:**
   ```sql
   SELECT c.customerNumber, c.customerName, p.amount 
   FROM customers c 
   JOIN payments p ON c.customerNumber = p.customerNumber 
   ORDER BY p.amount DESC LIMIT 1;
   ```
   *(Result: $120,000 check)*.
7. **Best Customer by Payment:**
   Demonstrating the same `INNER JOIN`, emphasizing that swapping left/right table positions yields identical results.
8. **Customers Without Payments (Show all customers without payment):**
   ```sql
   SELECT c.customerNumber, c.customerName, p.amount 
   FROM customers c 
   LEFT JOIN payments p ON c.customerNumber = p.customerNumber 
   WHERE p.amount IS NULL;
   ```
   *(Result: 24 unlinked records — identifying orphaned data)*.
9. **Full Name Formatting via Concatenation:**
   ```sql
   SELECT CONCAT(firstName, ' ', lastName) AS full_name 
   FROM employees 
   ORDER BY full_name ASC;
   ```

###### Part 2: UniversityDB Database
1. **Students and Major Departments:** Joining `students` and `departments` on `department_id` with full name concatenation.
2. **Class Schedule Info:** `INNER JOIN` across 4 tables (`classes`, `courses`, `instructors`, `semesters`).
3. **Enrollment Details:** 5-table join including `enrollments` and enrollment dates.
4. **Student Grades per Course:** Comprehensive 5-table query displaying student course grades.

---

##### 2. Core Concepts of Lecture 6

###### A. Baseline Data Profiling (General Column Analysis)
The lecturer recommends performing a **full aggregate table audit** before performing targeted groupings to establish a baseline mental map:
* For Text/Categorical Fields: `COUNT(*)`, `COUNT(DISTINCT column)`.
* For Numeric Fields: `MIN()`, `MAX()`, `AVG()`, `SUM()`.

*Example for `customers` table:*
```sql
SELECT 
    COUNT(*) AS total_customers,
    COUNT(DISTINCT country) AS distinct_countries,
    COUNT(DISTINCT city) AS distinct_cities,
    MAX(creditLimit) AS max_credit,
    MIN(creditLimit) AS min_credit,
    AVG(creditLimit) AS avg_credit
FROM customers;
```
*(Result: 122 customers, 27 countries, 95 cities, max limit $227,600, min $0, avg $67,659)*.

###### B. Data Grouping (`GROUP BY`)
Applies aggregate functions to distinct sub-groups. Rule: Any non-aggregated column selected in `SELECT` **must** be listed in the `GROUP BY` clause.

*Example Grouping by Country (`customers`):*
```sql
SELECT 
    country, 
    COUNT(*) AS customer_count, 
    COUNT(DISTINCT city) AS distinct_cities, 
    MAX(creditLimit) AS max_credit, 
    AVG(creditLimit) AS avg_credit
FROM customers
GROUP BY country;
```
*(Result: 27 rows — aggregated metrics for each country)*.

###### C. Filtering Aggregated Results (`HAVING` vs `WHERE`)
* `WHERE` filters **raw rows** *before* grouping occurs.
* `HAVING` filters **grouped aggregate results** *after* `GROUP BY` execution.

*Execution Syntax Order:*
`SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT ...`

###### D. Set Unification (`UNION` vs `UNION ALL`)
* `UNION ALL`: Merges result sets **preserving all duplicate rows**. Faster execution.
* `UNION`: Merges result sets and **removes duplicates** (returns distinct rows).
* *Requirement:* Both queries must select the exact same number of columns with compatible data types.

*Example adding a custom Database Source Tag:*
```sql
SELECT city, 'ClassicModels' AS DB FROM customers
UNION ALL
SELECT city, 'MyWork' AS DB FROM department;
```
*(129 rows with `UNION ALL` vs 102 distinct rows with `UNION`)*.

###### E. Subqueries & Common Table Expressions (`WITH ... AS`)
* **Subquery in `WHERE`:** Generates a single-column list of values for matching with `IN`.
  ```sql
  SELECT customerNumber, customerName, city 
  FROM customers 
  WHERE customerNumber IN (
      SELECT customerNumber 
      FROM payments 
      GROUP BY customerNumber 
      HAVING SUM(amount) > 70000
  );
  ```
* **Common Table Expressions (CTE):** Temporary named result set created using the `WITH` keyword. Enhances readability and modular structure in test automation.
  ```sql
  WITH CTE_HighSpenders AS (
      SELECT customerNumber, SUM(amount) AS total_spent
      FROM payments
      GROUP BY customerNumber
      HAVING SUM(amount) > 70000
  )
  SELECT c.customerNumber, c.customerName, c.city, cte.total_spent
  FROM customers c
  JOIN CTE_HighSpenders cte ON c.customerNumber = cte.customerNumber;
  ```

---

#### 🔮 4. QA Superpower & Student Questions from Lecture
Key student questions raised during Lecture 6 that mentors should address:

##### ❓ Question 1: “Why can't we write `WHERE SUM(amount) > 70000`?”
* **Mentor Explanation:** SQL query execution order is: `FROM` ➔ `WHERE` ➔ `GROUP BY` ➔ `HAVING` ➔ `SELECT`. At the `WHERE` stage, rows have not been grouped yet, so the engine cannot evaluate totals. Aggregate functions (`SUM`, `COUNT`, `AVG`) require the `HAVING` clause, which executes *after* grouping.

##### ❓ Question 2: “What is the difference between `UNION` and `UNION ALL` for QA testing?”
* **Mentor Explanation:** 
  * `UNION ALL` retains all records including duplicates. When validating total record migration or transaction counts, `UNION ALL` reflects true raw row volume.
  * `UNION` performs deduplication. Useful when compiling unified customer mailing lists or master directories where duplicate entries cause errors.

##### ❓ Question 3: “Why did `UNION` yield 102 rows while `SELECT DISTINCT` yielded 101 rows?”
* **Mentor Explanation:** `UNION` evaluates uniqueness across **all selected columns** (including literal source tags like `'ClassicModels'` vs `'MyWork'`). If a city exists in both tables, the distinct DB labels make the rows unique for `UNION`. A `SELECT DISTINCT` on just the city name ignores DB tags and collapses them into 1 row.

##### ❓ Question 4: “When should we use a Subquery vs a CTE (`WITH`)?”
* **Mentor Answer:** Subqueries are quick for inline filtering. CTEs are preferred for multi-stage transformations, complex aggregations, or repeated joins because they keep test code clean, modular, and maintainable.

---

#### 💻 5. Practical Script: Live Queries & Hands-on Tasks
Summary breakdown of practical queries analyzed from a QA testing perspective:

| # | Database | SQL Statement / Operation | Target Concept | 💡 QA Testing Context |
|---|---|---|---|---|
| **1** | ClassicModels | `SELECT COUNT(*), COUNT(DISTINCT country), MAX(creditLimit) FROM customers;` | Baseline Profiling | Initial table audit before writing requirement tests. |
| **2** | ClassicModels | `SELECT country, COUNT(*) FROM customers GROUP BY country;` | GROUP BY | Validating entity distribution across geographical regions. |
| **3** | ClassicModels | `SELECT jobTitle, COUNT(*) FROM employees GROUP BY jobTitle;` | GROUP BY | Verifying organizational structure grouping. |
| **4** | ClassicModels | `SELECT country, COUNT(*) FROM offices GROUP BY country;` | GROUP BY | Reconciling physical office branch counts by country. |
| **5** | ClassicModels | `SELECT productLine, COUNT(*), AVG(buyPrice) FROM products GROUP BY productLine;` | GROUP BY + Aggregates | Validating catalog metrics and average product pricing. |
| **6** | ClassicModels | `SELECT city, 'ClassicModels' AS DB FROM customers UNION ALL SELECT city, 'MyWork' AS DB FROM department;` | UNION ALL | Testing full dataset merging across disparate services. |
| **7** | ClassicModels | `SELECT city FROM customers UNION SELECT city FROM department;` | UNION (Deduplicated) | Verifying unified deduplicated lookup table generation. |
| **8** | ClassicModels | `SELECT customerNumber FROM payments GROUP BY customerNumber HAVING SUM(amount) > 70000;` | GROUP BY + HAVING | Pinpointing VIP customers exceeding spending thresholds. |
| **9** | ClassicModels | `SELECT * FROM customers WHERE customerNumber IN (SELECT customerNumber FROM payments GROUP BY customerNumber HAVING SUM(amount) > 70000);` | Subquery in WHERE | Preparing test customer datasets for loyalty E2E validations. |
| **10** | ClassicModels | `WITH CTE_amount AS (...) SELECT c.*, cte.total_amount FROM customers c JOIN CTE_amount cte ON ...;` | CTE (`WITH`) | Elegant and readable test script structure for financial audits. |

---

#### ⚠ 6. Common Student Pitfalls & Live Troubleshooting

| Problem / Symptom | Root Cause | How Mentor Should Explain Fix |
|---|---|---|
| **Error Code: 1111. Invalid use of group function** | Placing aggregate functions (`SUM`, `COUNT`, `AVG`) inside a `WHERE` clause. | Explain that `WHERE` runs before aggregation. Use `HAVING` after `GROUP BY` to filter aggregate values. |
| **Error Code: 1248. Every derived table must have its own alias** | Nested subquery in the `FROM` clause lacks a table alias. | Demonstrate adding an explicit table alias at the end of subquery parentheses: `FROM (SELECT ...) AS sub_table`. |
| **Column count mismatch in UNION (`The used SELECT statements have a different number of columns`)** | Mismatched column counts or incompatible data types across `UNION` queries. | Teach the fundamental rule: every `SELECT` in a `UNION` must have identical column counts and positions. |
| **Ambiguous or incorrect GROUP BY results** | Selecting columns in `SELECT` that are neither aggregated nor listed in `GROUP BY`. | Explain the ANSI SQL rule: all non-aggregated columns in `SELECT` must be explicitly declared in `GROUP BY`. |

---

#### ⏱ 7. Timeboxed Review Session Scenario (50 Minutes)

##### 📌 Phase 1: Warm-up & Curriculum Map (5 mins)
* Welcome students and position Lesson 6 on the 10-lecture curriculum map.
* Highlight the transition to analytical queries, aggregate calculations, and data reconciliation.

##### 📌 Phase 2: Homework Review (10 mins)
* Review key queries from ClassicModels #1–9 (Shelby Cobra vendor, min/max MSRP, best customer, `IS NULL` payment checks).
* Walk through UniversityDB queries (4–5 table joins).

##### 📌 Phase 3: Theory Deep-Dive & Student Questions (15 mins)
* Interactive overview of data profiling (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
* Explain key differences: `WHERE` vs `HAVING`, `UNION` vs `UNION ALL`, Subqueries vs CTEs.
* Re-address student questions from the lecture recording (`WHERE` aggregate error, DB source tagging).

##### 📌 Phase 4: Hands-on Live Coding on ClassicModels (15 mins)
* Demonstrate baseline profiling queries on `customers`, `employees`, and `products`.
* Construct `GROUP BY` with `HAVING SUM(amount) > 70000`.
* Demonstrate `UNION` vs `UNION ALL` with custom DB tags (`'ClassicModels' AS DB`).
* Build equivalent queries using Subquery in `WHERE` and CTE (`WITH`).

##### 📌 Phase 5: Wrap-up & Homework Preview (5 mins)
* Summarize key takeaways: rules for aggregate grouping and subquery usage.
* Announce Lesson 6 Homework: ClassicModels (Queries 10–15) and RealEstateDB preview (4 queries).

---

#### 🎯 8. Mentor Checklist & Definition of Done (DoD)
By the end of the review session, students should be able to:
- [ ] Perform baseline profiling audits on unstudied tables using aggregate functions.
- [ ] Articulate the difference between row filtering (`WHERE`) and aggregate filtering (`HAVING`).
- [ ] Apply `GROUP BY` confidently to compute business KPIs and reconcile analytical reports.
- [ ] Understand the difference between `UNION` and `UNION ALL` and apply them in integration testing.
- [ ] Construct subqueries with the `IN` operator to generate targeted test data sets.
- [ ] Write clean, modular CTE expressions using the `WITH` keyword for automated test scripts.
