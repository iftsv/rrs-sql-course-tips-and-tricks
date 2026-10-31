# [SQL-A] 10. (10/28) SQL in Reporting — Review Session Guide
**Methodological Guide for Mentors and Reviewers**  
**Course:** SQL for QA Automation / QA Engineer  
**Session Topic:** SQL in Data Visualization & BI Reporting, BI Tool Ecosystem (Tableau, Power BI, Google Looker Studio), Master Queries vs Database Views, Real-World Analytics Case Studies (San Francisco SFPD Crime Dataset & HKSpace Non-Profit Multi-Source Pipeline), and Review of Homework #9 (World DB, Jeopardy DB, Hotel DB).

---

## 🧭 1. Perspective: 10-Lecture Course Map
Remind students of their current position on the course map. In Session 10, students reach the final core theoretical domain of the course: **SQL in Reporting and Data Visualization**. Having mastered relational modeling, complex multi-table joins, analytical functions, windowing, and Data Warehouse (DWH) concepts in previous sessions, students now focus on how raw database records are aggregated, structured, and presented to end-users and decision-makers via Business Intelligence (BI) dashboards. Understanding BI integration and reporting architecture is crucial for QA Engineers to validate end-to-end data pipelines, verify reporting accuracy, reconcile dashboard metrics against database truth, and test data transformations.

* **Lecture 1:** Introduction to Relational Databases (RDBMS), Tables, Rows, Columns, Primary & Foreign Keys, Data Integrity.
* **Lecture 2:** Simple Queries on a Single Table — Filtering, Sorting, Result Limiting (`LIMIT`), and Metadata Profiling.
* **Lecture 3/4:** SQL Components — Syntax and Practice for DDL (Data Definition), DQL (Data Query), and DML (Data Manipulation).
* **Lecture 5:** Table Joins (`INNER JOIN`, `LEFT JOIN`, `SELF JOIN`, `FULL JOIN`).
* **Lecture 6:** Grouping, Set Operations, and Subqueries (`GROUP BY`, `HAVING`, `UNION` / `UNION ALL`, Subqueries, CTE / `WITH`).
* **Lecture 7:** Built-in Functions (Date, String, Numeric), Conditional Logic (`CASE`), Window Functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`, `LAG` / `LEAD`), and ETL Processes (`LOAD DATA LOCAL INFILE`).
* **Lecture 8:** Tables, Views, and Temporary Tables (`Tables` vs `Views` vs `Temp Tables`), Grand Master Query, RDBMS Architecture (5 Components), ACID Properties, SQL vs NoSQL, CAP Theorem, BASE Model, and Vector DBs.
* **Lecture 9:** Data Warehouse Concepts (DWH), Fact and Dimension Tables, Star / Snowflake / Galaxy Schemas, OLTP vs OLAP, Time Series / Historical Snapshots, Data Marts, and ETL Data Cleansing Projects.
* **Lecture 10 (Current): SQL in Reporting & Data Visualization, BI Tools Landscape (Tableau, Power BI, Looker Studio), Master Queries vs Views, Real-World Analytics Dashboards (SF Police Crime Data, HKSpace Non-Profit Pipeline), and Review of Homework #9 (World DB, Jeopardy DB, Hotel DB).**
* **Lecture 11 (Upcoming / Final):** Advanced Topics (Triggers, Stored Functions, Indexes Optimization) and Technical Interview Preparation.

---

## 🕵️‍♂️ 2. Session Concept and QA Testing Context
In Session 10, students master **the role of SQL in Business Intelligence (BI) and Data Visualization testing**. In everyday software quality assurance, BI reporting sits at the end of the data pipeline. QA Engineers must ensure that visual widgets (bar charts, line graphs, heatmaps, KPIs) accurately reflect the underlying SQL database metrics and that ETL transformations do not distort or drop data.

##### Key QA Objectives in Studying SQL in Reporting:
1. **Dashboard Data Reconciliation:** Verifying visual KPI numbers (total sales, average life expectancy, total incident counts, support call totals) by executing direct validation SQL queries against the underlying database tables.
2. **Master Query & View Validation:** Testing single comprehensive "Master Queries" and SQL Views designed for BI tools (Tableau, Power BI, Looker) to confirm correct join logic, NULL handling, and column aliasing.
3. **Filter & Parameter Testing:** Validating how BI filters (date ranges, geographic regions, categories, staff names) modify the underlying SQL queries and ensuring edge-case parameter values do not return duplicate or orphaned records.
4. **Multi-Source ETL Pipeline Testing:** Testing data pipelines that extract raw records from heterogeneous source systems (scheduling software, telephony, video call tools), clean and aggregate them in MySQL, and export processed tables to cloud reporting layers (Google Drive / Looker Studio).

---

## 🧱 3. Theoretical Deep-Dive: Homework Review, Data Visualization, BI Tools, Master Query & Real-World Dashboards

### 1. Review of Homework #9 (World DB, Jeopardy DB, Hotel DB)

#### Part 1: World SQL Database (`world` DB)
The `world` database consists of 3 relational tables: `city` (4,079 rows), `country`, and `countrylanguage`.
* **Row Count & Profiling:** Validating baseline record counts (`SELECT COUNT(*) FROM city` yields 4,079 rows).
* **Geographic Breakdown:** `SELECT DISTINCT continent, region, name AS country FROM country;` to inspect global coverage.
* **Language Analysis (Sydney):** Joining `city` with `countrylanguage` on `countrycode` where `city.name = 'Sydney'` to list languages spoken in Sydney.
* **Government Forms:** `SELECT governmentform, COUNT(*) FROM country GROUP BY governmentform ORDER BY COUNT(*) DESC;` identifying Republic as the predominant form of government.
* **Population Ranking:** Ranking countries by population in descending order (`China`, `India`, `United States`, `Indonesia`, `Brazil`).
* **Language Diversity:** Identifying countries with the highest count of spoken languages (multiple countries record up to 12 languages).
* **Life Expectancy Anomalies:** Querying lowest life expectancy (e.g., Zambia at 37 years in historical statistical datasets).
* **English-Speaking Country Flag (`CASE` Statement & Subquery):**
  * Creating a conditional flag: `CASE WHEN language = 'English' THEN 1 ELSE 0 END`.
  * Aggregating via `MAX()` or calculating percentage of English speakers to categorize countries into English-speaking vs non-English-speaking.
* **Population Segmentation (< 1M vs > 1M):**
  * Grouping countries by population size (< 1,000,000 vs >= 1,000,000).
  * Calculating average life expectancy formatted to 2 decimal places using `FORMAT(AVG(lifeexpectancy), 2)`.
  * Results: 85 countries with population < 1M, 154 countries with population > 1M.
* **Full Database Join:** Joining `city`, `country`, and `countrylanguage` together (yielding 4,079 city-level records).

#### Part 2: Jeopardy Trivia DB & Hotel Management DB (`hotel_db`)
* **Jeopardy DB:** Solving 11 analytical DQL trivia queries across categories, scores, and questions.
* **Hotel Management DB (`hotel_db`):**
  * **Schema Profiling & ER Diagram:** Using MySQL Workbench *Database -> Reverse Engineer* to visualize multi-table relationships across `reservations`, `guests`, `rooms`, `services`, `service_transactions`, `payments`, `staff`, and `housekeeping_log`.
  * **Service Transactions Query:** Joining 4 tables (`service_transactions`, `services`, `reservations`, `guests`) to report guest service charges.
  * **Housekeeping Log:** Querying room maintenance records, cleanliness statuses, and operational notes.

---

### 2. Comparative Analysis: BI Tools Ecosystem

| Feature | Tableau | Power BI | Google Looker Studio |
| :--- | :--- | :--- | :--- |
| **Vendor / Developer** | Salesforce | Microsoft | Google |
| **Primary Target Market** | Enterprise Data Analytics & Data Science | Enterprise Corporate Reporting & Office 365 Integration | Web Analytics, SMBs, Non-Profits & Google Workspace Ecosystem |
| **Data Connections** | Direct DB (MySQL, PostgreSQL, Snowflake), CSV, Excel, Cloud | Direct DB, Power Query ETL, Excel, Azure, REST APIs | Google Sheets, BigQuery, MySQL, Google Analytics, CSV |
| **Data Transformation Capability** | High (Calculated Fields, LOD Expressions) | Very High (Power Query / M Language, DAX) | Moderate (Calculated Fields, Blend Data) |
| **Cost & Licensing** | Paid License (Desktop / Creator), Limited Public/Student Tier | Included in enterprise E5 / Paid Pro per user | Free with Google Workspace / Cloud account |

---

### 3. Master Query Architecture vs Database Views for BI Integration

```
+-----------------------------------------------------------------------------------+
|                            RELATIONAL DATABASE (OLTP / DWH)                        |
|  +---------------+    +---------------+    +---------------+    +---------------+ |
|  |  customers    |    |    orders     |    | order_details |    |   products    | |
|  +---------------+    +---------------+    +---------------+    +---------------+ |
+----------------------------------------+------------------------------------------+
                                         |
                       [ SQL Master Query / Database View ]
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                        DENORMALIZED FLAT REPORTING DATASET                        |
| (customer_name, order_date, product_title, category, quantity, total_amount, etc.) |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                            BUSINESS INTELLIGENCE (BI)                             |
|                    [ Tableau / Power BI / Google Looker Studio ]                  |
|                 (Interactive Dashboards, Filtering, Visual Charts)                |
+-----------------------------------------------------------------------------------+
```

#### Why Build a Master Query or View for BI?
1. **Decoupling Logic from Visual Layer:** Complex multi-table `JOIN` logic, date formatting, and text transformations are executed directly in SQL. The BI tool simply connects to a flat, clean dataset.
2. **Performance Optimization:** Offloads heavy relational query processing to the database server.
3. **Single Source of Truth:** Ensures all BI dashboards and QA validation queries pull from the exact same pre-tested SQL definition (`v_master_reporting`).

---

### 4. Real-World Case Study 1: San Francisco Crime Data & Dashboard Analytics
* **Dataset Characteristics:** Public municipal police dataset covering incident logs from 2018 to present.
* **Schema Attributes:** Incident ID, Incident Number, Incident Category (Theft/Larceny, Assault, Fraud), Sub-Category, Description, Resolution Status, Street / Intersection, Police District, Neighborhood, Latitude, Longitude, Date, Time, Day of Week, Hour (0–23).
* **Analytical & Visual Patterns:**
  * **District & Neighborhood Ranking:** Comparing crime volumes across police districts (identifying high-density crime districts vs quiet residential neighborhoods).
  * **24-Hour Incident Distribution:** Peaks occur at **12:00 PM (noon)** and **12:00 AM (midnight)**, while the lowest crime volume occurs at **5:00 AM**.
  * **Dominant Crime Categories:** Theft / Larceny (vehicle break-ins) constitutes the largest single category.
  * **Dashboard Layout:** Combining bar charts (incidents by neighborhood), horizontal histograms (incidents by hour), monthly trendlines, bubble charts/word clouds (crime categories), and geographic map layers.

---

### 5. Real-World Case Study 2: Non-Profit Multi-Source Pipeline (HKSpace / Google Looker Studio)
* **Business Context:** Non-profit organization offering emotional support, life coaching, and career guidance via 3 distinct channels.
* **Heterogeneous Data Sources:**
  1. *Scheduling Software CSV:* Appointment bookings and client registrations.
  2. *Telephony Software CSV:* Inbound and outbound phone call logs.
  3. *Video Conference Software CSV:* Video coaching session records.
* **ETL & Reporting Architecture:**
  $$	ext{Raw Source CSVs} \longrightarrow 	ext{MySQL Database (Cleansing \& Massaging Script)} \longrightarrow 	ext{Google Drive / Sheets} \longrightarrow 	ext{Looker Studio Dashboard}$$
* **QA & Usability Findings:**
  * *Navigator / Staff Workload Tracking:* Measuring total hours and support volume per staff member.
  * *Geographic Concentration:* Call volumes peak in **California** (home region) and **Florida**.
  * *Filter UI/UX QA Defects:* Multi-select filters without a clear "Deselect All" button create friction for reviewers attempting to analyze a single navigator's workload.

---

## 🔮 4. QA Superpower and Student Q&A from Lecture

##### ❓ Question 1: "Why is it better to write a SQL Master Query or View in MySQL before connecting to Tableau/Power BI, rather than drag-and-dropping individual tables in the BI tool?"
* **Mentor Answer:** Drag-and-dropping multiple raw tables inside a BI tool causes the BI software to generate automated, often unoptimized SQL queries under the hood with redundant `JOIN` operations. A hand-crafted SQL Master Query or View guarantees optimal join paths, standardizes column names, applies consistent `CASE` transformations, and allows QA Engineers to test and validate the exact dataset query independently in MySQL Workbench.

##### ❓ Question 2: "How does a QA Engineer validate that numbers displayed on a BI dashboard (e.g., Looker Studio or Tableau) are 100% accurate?"
* **Mentor Answer:** QA performs **Data Reconciliation Testing**. For every widget on the dashboard (e.g., Total Incidents in 2024 = 15,420), QA writes a direct SQL query against the database using identical filter criteria (`SELECT COUNT(*) FROM sf_crime_data WHERE YEAR(incident_date) = 2024`). If the database result matches the visual widget exactly, the widget is validated. QA must also test filter parameters, date boundaries, and missing/NULL handling.

##### ❓ Question 3: "In the HKSpace non-profit pipeline case, what potential ETL bugs should QA test when merging 3 different data sources (Scheduling, Telephony, Video)?"
* **Mentor Answer:** Key ETL risk areas include:
  1. *ID Misalignment / Orphan Records:* Clients present in call logs but missing from the scheduling database.
  2. *Time Zone Discrepancies:* Telephony logs recorded in UTC while video session logs are in local PST.
  3. *Duplicate Records:* Re-importing overlapping CSV files during batch updates causing double-counting of calls or sessions.

##### ❓ Question 4: "Why do SF Crime statistics show major spikes at 12:00 PM and 12:00 AM, and how should SQL queries handle hour-of-day analytics?"
* **Mentor Answer:** Spikes at 12:00 PM and 12:00 AM are caused by default system timestamps (when exact incident time is unknown, systems often default to `00:00:00` or `12:00:00`) as well as actual midnight/noon activity shifts. In SQL, hour analytics utilize `HOUR(incident_time)` grouping (values `0` to `23`, where `0` represents midnight). QA must verify that `0` is properly recognized as midnight and not treated as a missing or NULL value.

---

## 💻 5. Practical Script: Live Queries & Hands-on Tasks

| № | DB / Object | SQL Query / Operation | Key Concept | 💡 QA Context & Verification Scenario |
| :-: | :--- | :--- | :--- | :--- |
| **1** | World DB | `SELECT city.name, countrylanguage.language FROM city JOIN countrylanguage ON city.countrycode = countrylanguage.countrycode WHERE city.name = 'Sydney';` | Multi-Table `JOIN` | Validating language breakdown for a specific city across joined tables. |
| **2** | World DB | `SELECT name, population, CASE WHEN population < 1000000 THEN 'Small (<1M)' ELSE 'Large (>=1M)' END AS pop_category FROM country;` | Conditional Logic (`CASE`) | Categorizing entities into reporting buckets for dashboard aggregation. |
| **3** | World DB | `SELECT CASE WHEN population < 1000000 THEN 'Small (<1M)' ELSE 'Large (>=1M)' END AS category, COUNT(*) AS country_count, FORMAT(AVG(lifeexpectancy), 2) AS avg_life_expectancy FROM country GROUP BY category;` | Aggregation & `FORMAT()` | Reconciling summary stats (<1M: 85 countries; >=1M: 154 countries) for BI scorecards. |
| **4** | Hotel DB | `SELECT g.first_name, g.last_name, s.service_name, st.transaction_date, st.amount FROM service_transactions st JOIN services s ON st.service_id = s.service_id JOIN reservations r ON st.reservation_id = r.reservation_id JOIN guests g ON r.guest_id = g.guest_id;` | 4-Table Relationship `JOIN` | Testing multi-table joins for customer billing and service report validation. |
| **5** | Reporting / ClassicModels | `CREATE VIEW v_master_reporting AS SELECT o.order_number, o.order_date, c.customer_name, p.product_name, p.product_line, od.quantity_ordered, od.price_each, (od.quantity_ordered * od.price_each) AS line_total FROM orders o JOIN customers c ON o.customer_number = c.customer_number JOIN orderdetails od ON o.order_number = od.order_number JOIN products p ON od.product_code = p.product_code;` | Master Query View (`CREATE VIEW`) | Constructing a flat denormalized reporting View for seamless BI tool integration. |
| **6** | SF Crime DB | `SELECT neighborhood, COUNT(*) AS incident_count FROM sf_crime_data WHERE YEAR(incident_date) = 2024 GROUP BY neighborhood ORDER BY incident_count DESC;` | District Aggregation DQL | Validating neighborhood bar chart metrics on BI dashboard against SQL truth. |
| **7** | SF Crime DB | `SELECT HOUR(incident_time) AS incident_hour, COUNT(*) AS total_incidents FROM sf_crime_data GROUP BY incident_hour ORDER BY incident_hour;` | Time Discretization (`HOUR()`) | Testing 24-hour crime distribution curve (hours 0 to 23). |

---

## ⚠️ 6. Common Student Errors & Live Troubleshooting

| Issue / Symptom | Root Cause | Mentor Instruction & Fix |
| :--- | :--- | :--- |
| **Error: Missing databases (`world`, `hotel_db`) in MySQL Workbench after computer restart or re-installation.** | Databases were not restored on the local MySQL instance on a new machine. | Instruct students to locate SQL source scripts (`world.sql`, `hotel_db.sql`) and run *File -> Open SQL Script -> Execute (Lightning bolt)* followed by *Refresh All*. |
| **Symptom: BI Dashboard totals do not match raw SQL `SUM()` or `COUNT()` results.** | The BI tool applied default duplicate handling, missing JOIN filters, or inconsistent date range parameters. | Verify the BI tool connection query. Ensure BI filters mirror the exact SQL `WHERE` clause conditions used in validation queries. |
| **Symptom: Hour 0 (midnight) is omitted or lumped into missing data in reporting charts.** | SQL query uses `1` to `24` indexing or misinterprets `00:00:00` timestamp strings. | Use `HOUR(time_column)` which correctly returns `0` for midnight through `23` for 11 PM. Ensure BI axis maps `0` to Midnight. |
| **Error: Dashboard multi-select filter makes it impossible to isolate a single entity without manually unchecking dozens of boxes.** | BI dashboard control element was configured as a multi-choice checkbox list without "Single Select" or "Search" controls. | Teach UX/QA filter testing: reconfigure dashboard filter settings to *Single Select Dropdown* or add a *Clear Selection* control. |

---

## ⏱️ 7. Step-by-Step Session Time Plan (50 Minutes)

##### 📌 Phase 1: Introduction & Course Map (5 min)
* Welcome students and situate Session 10 on the 10-lecture course map.
* Overview of core topics: SQL in BI & Reporting, Tableau / Power BI / Looker Studio, Master Queries, and Real-World Dashboards.

##### 📌 Phase 2: Homework #9 Review (10 min)
* Walk through `world` DB queries (Sydney languages, English `CASE` statement, population bracket aggregation <1M vs >=1M).
* Inspect `hotel_db` ER diagram (4-table service transaction join, housekeeping log).

##### 📌 Phase 3: Theoretical Deep-Dive (15 min)
* BI tool landscape (Tableau vs Power BI vs Looker Studio).
* Master Query vs Database View architecture for BI.
* Case Study 1: SF Crime Data (24-hour distribution, neighborhood rankings, theft metrics).
* Case Study 2: Non-Profit HKSpace ETL Pipeline (CSV -> MySQL -> Google Sheets -> Looker Studio).

##### 📌 Phase 4: Live Hands-on Practice in MySQL Workbench (15 min)
* Writing a flat `v_master_reporting` View in MySQL Workbench.
* Running validation SQL queries for SF Crime metrics and population brackets.
* Live demonstration of data reconciliation between SQL query output and dashboard visual widgets.

##### 📌 Phase 5: Wrap-up & Definition of Done (5 min)
* Answer student Q&A and verify Definition of Done criteria.
* Teaser for Lecture 11 (Final Session): Advanced Topics (Triggers, Stored Functions, Indexing Optimization) & Technical Interview Preparation.

---

## 🎯 8. Mentor Checklist & Definition of Done (DoD)
By the end of the review session, students must be able to:
* [-] Explain the role of SQL in BI reporting and data visualization architecture.
* [-] Compare major BI tools (Tableau, Power BI, Google Looker Studio) and their connection methods.
* [-] Construct a flat Master Query or Database View (`CREATE VIEW`) to supply clean denormalized data to reporting tools.
* [-] Execute SQL data reconciliation queries to validate visual dashboard widgets against underlying database truth.
* [-] Analyze multi-source ETL pipelines (Extract -> MySQL Transform -> Cloud Reporting) and identify potential QA risk areas.
* [-] Apply `CASE` statements, date/time functions (`HOUR()`, `FORMAT()`), and multi-table `JOIN`s to complex reporting requirements.
