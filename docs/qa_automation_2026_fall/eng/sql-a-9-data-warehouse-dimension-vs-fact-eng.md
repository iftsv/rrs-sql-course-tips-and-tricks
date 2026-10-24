# [SQL-A] 9. (10/21) Data Warehouse, Dimension vs Fact — Review Session Guide

**Methodological Guide for Mentors and Reviewers**  
**Course:** SQL for QA Automation / QA Engineer  
**Session Topic:** Data Warehouse (DWH) Concepts, Dimension Tables vs Fact Tables, Star Schema & Snowflake Schema, OLTP vs OLAP Comparison, Time Series & Historical Snapshots, Data Marts, ETL Data Cleansing Processes, and Review of Lesson 8 Homework (English Dictionary Data Cleansing Project & Tesla DB / World DB / Jeopardy DB).

---

## 🧭 1. Perspective: 10-Lecture Course Map

Remind students of their current position on the course map. In Session 9, students make a fundamental transition from operational database concepts (OLTP) and local storage objects (Tables, Views, Temp Tables) to **Analytical Data Warehouse Architecture (DWH), Star and Snowflake Data Modeling, and ETL Pipeline Organization**. Understanding DWH principles is essential for QA Engineers to test end-to-end business workflows, validate BI reporting, verify ETL pipelines, and ensure big data quality.

* **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary & Foreign Keys, Data Integrity.
* **Lecture 2:** Simple Queries on a Single Table — Filtering, Sorting, Result Limiting (`LIMIT`), and Metadata Profiling.
* **Lecture 3/4:** SQL Components — Syntax and Practice for DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation).
* **Lecture 5:** Table Joins (`INNER JOIN`, `LEFT JOIN`, `SELF JOIN`, `FULL JOIN`).
* **Lecture 6:** Grouping, Set Operations, and Subqueries (`GROUP BY`, `HAVING`, `UNION` / `UNION ALL`, Subqueries, CTE / `WITH`).
* **Lecture 7:** Built-in Functions (Date, String, Numeric), Conditional Logic (`CASE`), Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG`/`LEAD`), and ETL Processes (`LOAD DATA LOCAL INFILE`).
* **Lecture 8:** Tables, Views, and Temporary Tables (`Tables` vs `Views` vs `Temp Tables`), Grand Master Query, RDBMS Architecture (5 Components), ACID Properties, SQL vs NoSQL, CAP Theorem, BASE Model, and Vector DBs.
* **Lecture 9 (Current): Data Warehouse Concepts (DWH), Fact and Dimension Tables, Star / Snowflake / Galaxy Schemas, OLTP vs OLAP, Time Series / Historical Snapshots, Data Marts, and ETL Data Cleansing Projects.**
* **Lecture 10:** Advanced Topics (Triggers, Stored Functions, Indexes) and Technical Interview Preparation.

---

## 🕵️‍♂️ 2. Session Concept and QA Testing Context

In Session 9, students master **the design and testing of Analytical Data Warehouses (DWH)**. In daily QA Automation and Manual QA practice, working with DWH allows engineers to verify cross-system integration, validate business intelligence reporting, and prevent data loss or distortion during ETL pipeline execution.

### Key QA Objectives in Studying DWH, Fact/Dimension, and ETL:
1. **ETL Pipeline Testing:** Verifying data completeness, accuracy, and transformation logic during data transfer from transactional databases (OLTP) to analytical tables (DWH). Validating data cleansing rules, `NULL` handling, surrogate key generation, and duplicate elimination.
2. **Business Intelligence & Reporting Validation:** Reconciling metrics in final dashboards and reports against raw transactional facts. Understanding Star and Snowflake schemas enables QA to build efficient validation SQL queries for aggregations (`SUM`, `COUNT`, `AVG`) without redundant heavy `JOIN` operations.
3. **Time Series & Historical Snapshot Testing:** Verifying periodic snapshot generation ("as of end of month / week") and confirming historical data immutability for financial audits and regulatory compliance.
4. **Data Marts Validation:** Testing subject-specific warehouse subsets created for specific business departments (Sales, Marketing, Finance) and ensuring data consistency between core DWH and departmental Data Marts.
5. **Surrogate Keys & Performance Testing:** Validating `AUTO_INCREMENT` behavior, verifying key uniqueness, and checking index optimization on fact tables containing millions of records.

---

## 🧱 3. Theoretical Deep-Dive: Homework Review, Data Warehouse, OLTP vs OLAP, Dimension vs Fact, Star & Snowflake Schemas, Time Series & Data Marts

### 1. Review of Homework #8

#### Part 1: English Dictionary & Data Cleansing Project
* **Project Scope:** Cleansing and transforming a large text dataset from two CSV files: `English Master` (~176,000 rows) and `Most Common Words`.
* **ETL & Data Cleansing Workflow:** Data loading, duplicate handling, column renaming, and populating the `is_common` flag ('Yes' / 'No') to mark frequently used English words.
* **Analytical DQL Findings:**
  * **Word Frequency Distribution:** Out of 176k words, only **4,035 words** are classified as common (`common words`), while over **107,000 words** fall into the uncommon category (`uncommon words`).
  * **Parts of Speech (`word_type`):** Classification into Nouns (`N`), Adjectives (`A`), Verbs (`V`), Prepositions (`Prep`), Pronouns (`Pron`), etc.
  * **Semantic Filtering:** Queries searching for color-related definitions (`WHERE definition LIKE '%color%'`).
  * **Randomized Learning Dataset:** Generating random word selections via `ORDER BY RAND() LIMIT 4` filtered by `is_common = 'Yes'`.
  * **Combining Sets via `UNION`:** Selecting 4 random nouns and 4 random verbs for daily language study.
  * **String Character Breakdown:** Parsing fixed-length 7-character words (e.g., `qualify`) into separate columns `letter1`..`letter7` using string functions (`SUBSTRING` / `MID`).

#### Part 2: Additional Practice Databases (Tesla DB, World DB, Jeopardy DB, Hellman DB)
* **Tesla DB:** Answering 7 quiz queries regarding operational stats and financial metrics.
* **World DB:** Solving 8 queries on global geographic and demographic totals.
* **Jeopardy DB:** Ingesting real Jeopardy trivia dataset (CSV/JSON from Reddit/SQL materials) and solving 11 analytical questions.
* **Hellman DB:** Running 5 validation SQL queries.

---

### 2. Comparative Analysis: OLTP vs OLAP

| Feature | Operational DB (OLTP) | Data Warehouse (OLAP) |
| :--- | :--- | :--- |
| **Full Name** | Online Transaction Processing | Online Analytical Processing |
| **Primary Goal** | Recording day-to-day business operations (orders, payments, signups) | In-depth business analysis, trend detection, BI reporting |
| **Data Model** | Highly Normalized (3NF) to eliminate anomalies | Denormalized (Star / Snowflake Schemas) for fast reads |
| **Orientation** | Application-oriented | Subject-oriented |
| **Data Granularity** | Detailed real-time operational data | Aggregated historical snapshots (Batches / Snapshots) |
| **Operation Types** | Frequent, short `INSERT`, `UPDATE`, `DELETE`, simple `SELECT` | Massive read-heavy `SELECT` with aggregations (`SUM`, `AVG`, `GROUP BY`), batch loads |
| **Storage & Scale** | GB / TB scale per application/service | TB / PB enterprise-wide centralized repository |

---

### 3. Data Warehouse (DWH) Concepts & ETL Pipeline
* **What is DWH:** A centralized repository integrating data from disparate source systems (operational DBs, CRM, ERP, external APIs, CSV files) into a standardized schema.
* **ETL Workflow (Extract, Transform, Load):**
  1. **Extract:** Fetching raw data from operational OLTP sources.
  2. **Transform (Data Cleansing):** Cleaning data, standardizing names, handling missing values (`NULL`), generating surrogate keys, removing duplicates, and enforcing naming conventions (`t_`, `v_`, `tnp_`).
  3. **Load:** Populating clean transformed datasets into DWH tables.

---

### 4. Fact Tables vs Dimension Tables

* **Fact Table:**
  * Central table in DWH schemas.
  * Stores numeric business metrics/measures (sales revenue, quantity sold, session duration, discount amount).
  * Contains surrogate Foreign Keys referencing corresponding Dimension tables.
  * Characterized by rapid row growth and high volume (millions to billions of records).
* **Dimension Table:**
  * Stores descriptive attributes, categories, and contextual data (customer details, product specs, store locations, calendar dates).
  * Contains a surrogate Primary Key passed to Fact tables.
  * Changes infrequently and contains relatively small row counts.

---

### 5. Warehouse Schemas: Star Schema, Snowflake Schema, Galaxy Schema

```
        +------------------+
        |  dim_customer    |
        +------------------+
                 |
                 v
+--------------+ +------------------+ +--------------+
|  dim_product |->|    sales_fact    |<-|   dim_store  |
+--------------+ +------------------+ +--------------+
                 ^
                 |
        +------------------+
        |     dim_date     |
        +------------------+
```

1. **Star Schema:**
   * Central Fact table directly connected to denormalized Dimension tables.
   * Advantages: simple structure, fast query execution, minimal `JOIN` operations.
2. **Snowflake Schema:**
   * Extension of Star Schema where Dimension tables are further normalized into sub-dimensions (e.g., `dim_dealer` -> `dim_country` -> `dim_region`).
   * Advantages: reduces storage redundancy. Disadvantages: requires more complex SQL queries with multiple `JOIN`s.
3. **Galaxy Schema (Fact Constellation):**
   * Multiple Fact tables sharing common Dimension tables (e.g., sales facts and inventory facts sharing `dim_product` and `dim_date`).

---

### 6. Time Series / Historical Snapshots & Data Marts
* **Time Series & Snapshots:** Capturing data state at periodic intervals ("as of end of month"). Enables monthly/quarterly trend analysis, pattern recognition, and anomaly detection.
* **Data Marts:** Specialized, departmental subsets of DWH tailored for specific business teams (Marketing, Finance, Sales) allowing safe querying without impacting core warehouse operations.

---

## 🔮 4. QA Superpower and Student Q&A from Lecture

### ❓ Question 1: "Why do we use integer surrogate IDs in Fact tables instead of storing full customer names or product titles directly?"
* **Mentor Answer:** Fact tables contain millions or billions of rows. Repeating long text strings (e.g., "Johnathan Christopher Smith" or "Lenovo ThinkPad T14 Gen 3 Laptop") in every single row causes massive disk storage bloat and drastically slows down table scans. Using compact integer surrogate keys (`INT` / `BIGINT`) keeps Fact tables lean, while descriptive strings are stored once in Dimension tables.

### ❓ Question 2: "What is the key difference between Star and Snowflake schemas from a QA Automation perspective?"
* **Mentor Answer:** In Star schemas, dimensions are denormalized, so validation SQL queries are easier to write, require fewer `JOIN`s, and run faster. In Snowflake schemas, dimensions are normalized across multiple levels (e.g., `Dealer` -> `Country` -> `Region`). When testing Snowflake schemas, QA engineers must validate foreign key chains across sub-dimensions and inspect query Execution Plans (`EXPLAIN`) to confirm proper index usage.

### ❓ Question 3: "Why are DWH updates performed in batches, and how does QA test Time Series / Snapshot data?"
* **Mentor Answer:** DWH updates are executed via batch ETL jobs (nightly, weekly, or monthly) to avoid overloading live production OLTP systems during peak business hours. QA tests snapshot data by filtering on historical dates (`WHERE snapshot_date = '2026-09-30'`), reconciling aggregated DWH totals against raw OLTP transaction totals, and verifying historical immutability.

### ❓ Question 4: "How does `AUTO_INCREMENT` work in Fact tables during mass ETL batch loads, and what should QA verify?"
* **Mentor Answer:** During batch loads, `AUTO_INCREMENT` automatically assigns a unique surrogate Primary Key to each inserted fact row. In ETL scripts, the Primary Key column is omitted (or passed as `NULL`), and only Foreign Keys and numeric measures are provided. QA verifies that key sequences have no gaps or duplicates and confirms ID integrity under concurrent multi-threaded ETL loads.

---

## 💻 5. Practical Script: Live Queries & Hands-on Tasks

| № | DB / Object | SQL Query / Operation | Key Concept | 💡 QA Context & Verification Scenario |
|---|---|---|---|---|
| **1** | English Dict | `UPDATE dictionary_master SET is_common = 'Yes' WHERE word IN (SELECT word FROM most_common_words);` | Data Cleansing / Enrichment | Data enrichment and tagging high-frequency words prior to analytics. |
| **2** | English Dict | `SELECT is_common, COUNT(*) FROM dictionary_master GROUP BY is_common;` | DQL Aggregation | Validating frequency distribution between common (4,035) and uncommon (~107,000) words. |
| **3** | English Dict | `SELECT word, definition FROM dictionary_master WHERE definition LIKE '%color%';` | Text Pattern Search | Searching and validating all color-related word definitions. |
| **4** | English Dict | `SELECT word, word_type FROM dictionary_master WHERE is_common = 'Yes' AND word_type IN ('N', 'A') ORDER BY RAND() LIMIT 4;` | Randomization (`RAND()`) | Generating a random 4-word dataset of common nouns and adjectives for learning apps. |
| **5** | English Dict | `(SELECT word, 'Noun' AS type FROM dictionary_master WHERE word_type = 'N' AND is_common = 'Yes' ORDER BY RAND() LIMIT 4) UNION (SELECT word, 'Verb' AS type FROM dictionary_master WHERE word_type = 'V' AND is_common = 'Yes' ORDER BY RAND() LIMIT 4);` | Set Operations (`UNION`) | Combining two distinct random samples across parts of speech. |
| **6** | English Dict | `SELECT word, SUBSTRING(word, 1, 1) AS letter1, SUBSTRING(word, 2, 1) AS letter2, SUBSTRING(word, 3, 1) AS letter3 FROM dictionary_master WHERE LENGTH(word) = 7 AND is_common = 'Yes' ORDER BY RAND() LIMIT 1;` | String Processing | Character-by-character string splitting of 7-letter words across columns. |
| **7** | Family DWH | `CREATE TABLE dim_members (member_id INT AUTO_INCREMENT PRIMARY KEY, member_name VARCHAR(50), gender CHAR(1), birthdate DATE);` | Dimension Table (`dim_`) | Creating the family members dimension table. |
| **8** | Family DWH | `CREATE TABLE dim_activity (activity_id INT AUTO_INCREMENT PRIMARY KEY, activity_name VARCHAR(100), description TEXT);` | Dimension Table (`dim_`) | Creating the activity dimension table. |
| **9** | Family DWH | `CREATE TABLE fact_schedule (schedule_id INT AUTO_INCREMENT PRIMARY KEY, activity_date DATE, member_id INT, activity_id INT, FOREIGN KEY (member_id) REFERENCES dim_members(member_id), FOREIGN KEY (activity_id) REFERENCES dim_activity(activity_id));` | Fact Table & Foreign Keys | Creating the central schedule fact table with foreign key constraints. |
| **10** | Family DWH | `INSERT INTO fact_schedule (activity_date, member_id, activity_id) VALUES ('2026-10-25', 1, 2), ('2026-10-26', 2, 1);` | Fact Insertion | Inserting schedule fact records referencing surrogate dimension IDs. |
| **11** | Family DWH | `SELECT m.member_name, a.activity_name, f.activity_date, CASE DAYOFWEEK(f.activity_date) WHEN 1 THEN 'Sunday' WHEN 2 THEN 'Monday' WHEN 3 THEN 'Tuesday' WHEN 4 THEN 'Wednesday' WHEN 5 THEN 'Thursday' WHEN 6 THEN 'Friday' WHEN 7 THEN 'Saturday' END AS day_name FROM fact_schedule f JOIN dim_members m ON f.member_id = m.member_id JOIN dim_activity a ON f.activity_id = a.activity_id;` | DWH Joining & Date Logic | Validation reporting query decoding day of week and joining facts with dimensions. |
| **12** | Family DWH | `CREATE INDEX idx_schedule_date ON fact_schedule(activity_date);` | Indexing | Creating a date index on the fact table to accelerate snapshot queries. |
| **13** | StarSchemaDB | `CREATE TABLE sales_fact (sales_id INT AUTO_INCREMENT PRIMARY KEY, customer_id INT, product_id INT, store_id INT, date_id INT, quantity INT, total_amount DECIMAL(10,2), FOREIGN KEY (customer_id) REFERENCES dim_customer(customer_id), FOREIGN KEY (product_id) REFERENCES dim_product(product_id), FOREIGN KEY (store_id) REFERENCES dim_store(store_id), FOREIGN KEY (date_id) REFERENCES dim_date(date_id));` | Star Schema Fact Table | Creating central sales fact table in a Star schema. |
| **14** | StarSchemaDB | `SELECT d.year, p.category, SUM(f.total_amount) AS total_revenue FROM sales_fact f JOIN dim_date d ON f.date_id = d.date_id JOIN dim_product p ON f.product_id = p.product_id GROUP BY d.year, p.category;` | Star Schema Reporting Query | Analytical revenue reporting query by year and product category without heavy joins. |

---

## ⚠️ 6. Common Student Errors & Live Troubleshooting

| Issue / Symptom | Root Cause | Mentor Instruction & Fix |
| :--- | :--- | :--- |
| **Error: `Cannot add or update a child row: a foreign key constraint fails` on fact insert** | Inserting a record into Fact table referencing a non-existent Dimension ID. | Explain DWH load ordering: Dimension tables must be populated before inserting into Fact tables. |
| **Symptom: Aggregated report "lost" sales records after `JOIN`** | Using `INNER JOIN` when some fact records contain missing or unmapped dimension IDs. | Use `LEFT JOIN` to identify orphaned fact rows and map missing dimension IDs to a technical default (e.g., `-1` = 'Unknown'). |
| **Symptom: Slow execution of analytical queries on Fact table** | Missing indexes on Foreign Key columns (`customer_id`, `product_id`, `date_id`) in Fact table. | Create single or composite indexes on Foreign Key columns in Fact tables. |
| **Error: Discrepancy in revenue totals between OLTP DB and DWH for prior month** | Historical transactions in OLTP were updated after the DWH snapshot was taken. | Explain historical snapshot immutability in DWH and configure Change Data Capture (CDC) / incremental batch loads. |

---

## ⏱ 7. Step-by-Step Session Time Plan (50 Minutes)

### 📌 Phase 1: Introduction & Course Map (5 min)
* Welcome students and locate Session 9 position on the 10-lecture course map.
* Overview of core topics: DWH, OLTP vs OLAP, Fact vs Dimension, Star & Snowflake Schemas, Time Series.

### 📌 Phase 2: Review of Homework #8 (10 min)
* Review English Dictionary Data Cleansing Project (176k rows, `is_common` flag, queries, `ORDER BY RAND()`, `UNION`, string breakdown).
* Review Tesla DB, World DB, Jeopardy DB, and Hellman DB results.

### 📌 Phase 3: Theoretical Deep-Dive (15 min)
* OLTP vs OLAP comparative analysis and ETL pipeline stages.
* Fact vs Dimension table anatomy and surrogate keys.
* Star, Snowflake, and Galaxy schemas, Time Series snapshots, and Data Marts.

### 📌 Phase 4: Live Coding & Hands-on Practice in MySQL Workbench (15 min)
* Constructing `Family DWH` (`dim_members`, `dim_activity`, `fact_schedule`).
* Populating facts, decoding days of the week, and creating indexes.
* Demonstrating Star schema (`StarSchemaDB`) and running analytical revenue queries.

### 📌 Phase 5: Wrap-up & Definition of Done (5 min)
* Q&A session, student feedback, and checking DoD criteria.
* Teaser for Lecture 10: Advanced Topics (Triggers, Stored Functions, Indexes) and Technical Interview Prep.

---

## 🎯 8. Mentor Checklist & Definition of Done (DoD)

By the end of the review session, students must be able to:

- [-] Explain the fundamental differences between operational databases (OLTP) and Data Warehouses (OLAP / DWH).
- [-] Distinguish between Fact and Dimension tables and justify the use of surrogate keys.
- [-] Design Star Schema and Snowflake Schema structures and evaluate their trade-offs.
- [-] Construct analytical SQL queries joining Fact tables with Dimension tables.
- [-] Understand the concept of Time Series snapshots ("as of end of month") and Data Marts in QA testing.
- [-] Apply ETL Data Cleansing rules, duplicate handling, and foreign key integrity constraints.
