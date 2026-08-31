# [SQL-A] 2. (08/26) Intro to SQL - Review Session Guide (QA Course)

This methodological guide is designed for mentors, reviewers, and seminar leaders. Its purpose is to help you conduct an interactive review session to reinforce the material from the introductory SQL lecture. The guide includes the lecture structure, a breakdown of key concepts, practical QA-related use cases, and a technical analysis of the common error that students encountered.

---

## 🧭 1. Perspective: The 10-Lesson Course Roadmap
Before diving into the technical details, show students the "big picture." This helps reduce anxiety about the volume of new information and shows the exact skills they will acquire by the end of the course.

*   **Lesson 1 (Current):** Introduction to Relational Databases (RDBMS), tables, rows, columns, Primary and Foreign Keys, and referential integrity [2, 3].
*   **Lesson 2:** Simple queries addressing a single table (Querying one table) [2].
*   **Lesson 3:** Components of SQL (detailed syntax breakdown of DDL, DML, and DQL) [2].
*   **Lesson 4:** Joining tables (Inner Join, Left Join, and Self Join). *Instructor's note: Right Joins and other exotic joins are rarely used in real-world practice; we focus strictly on what matters [13].*
*   **Lesson 5:** Grouping, unions, and subqueries (Group by, Union/Union all, Subqueries) [2, 12].
*   **Lesson 6:** Built-in functions (text, numeric, and date functions) [2, 14].
*   **Lesson 7:** Temporary tables and views (Views vs. Temp tables) [2, 15].
*   **Lesson 8:** Data Warehouse (DWH) concepts (Dimension vs. Fact tables) [2].
*   **Lesson 9:** SQL in reporting and analytics [2].
*   **Lesson 10:** Advanced topics (Trigger Functions) and SQL interview preparation [3].

---

## 🛒 2. RDBMS Concept: The Costco Receipt Analogy
The best way to solidify the concept of relational databases is to decompose a physical receipt from a store like Costco into separate, structured tables. Relational databases are designed to minimize data duplication and establish clear connections between entities [4, 5].

### Decomposing a Receipt into Tables:
1.  **Store Table:**
    *   *Receipt Source Data:* Address "El Camino" and Location #475 [4, 5].
    *   *Fields:* `location_id` (PK), `location_name`, `address`, `phone_number` [5].
2.  **Customer Table:**
    *   *Receipt Source Data:* Membership card number (Member ID: 11842...) [5].
    *   *Fields:* `customer_id` (PK), `first_name`, `last_name`, `membership_type`, `status` [5].
3.  **Product Table:**
    *   *Receipt Source Data:* Sweet Potato, Medalin cookies, Mushrooms [5].
    *   *Fields:* `product_id` (PK), `product_name`, `price` [5].
4.  **Transaction Table (Receipts):**
    *   *Receipt Source Data:* Date and time (September 25th), Transaction #475, Cashier ID (Omarzi #107) [5].
    *   *Fields:* `transaction_id` (PK), `transaction_time`, `location_id` (FK), `customer_id` (FK), `cashier_id` [5].

### 💡 QA Use Case: Why do testers need this?
*   **Integration Testing (Frontend ➔ Backend):** When a user clicks the "Pay" button on a website, the frontend sends an API request to the backend. The QA engineer's job is to verify that a record is successfully created in the database's `Transaction` table and that it correctly references the existing `customer_id` and the `product_id` of the purchased items [43]. If these relations are misaligned, a customer might receive someone else's order, and business analysts will see corrupted data [43].

---

## 🔑 3. Database Keys: Primary Key vs. Foreign Key
Connections in relational databases are built using keys. In MySQL Workbench, these keys have distinct graphical icons [27].

### Primary Key (PK)
*   **What it is:** A unique identifier for a row in a table (e.g., `customer_id` or `product_id`) [7, 8]. It must never repeat within the same table and cannot contain null values (`NOT NULL`) [7, 8].
*   **Workbench Representation:** A **yellow key / lamp icon** next to the column name [27]. It is conventionally positioned as the very first column in a table [8].

### Foreign Key (FK)
*   **What it is:** A connector column that references a Primary Key in another table [8]. For example, in the `Transaction` table, the `customer_id` column acts as a Foreign Key pointing to the Primary Key of the `Customer` table [8, 45].
*   **Workbench Representation:** A **red diamond icon** [27].
*   **Key Difference:** Unlike Primary Keys, Foreign Keys can repeat across different records (e.g., the same customer can make multiple purchases, meaning their `customer_id` will appear multiple times in the `Transaction` table) [8, 45].

### 💡 QA Use Case: Why do testers need this?
*   **Negative Testing of Referential Integrity:** QA engineers must verify how the system behaves when database constraints are challenged [45]. For instance, if you try to delete a customer's profile (`customer_id = 99`) via an API while that customer still has active order history (records in the `Transaction` table), the database should block the deletion due to a *Foreign Key Constraint Violation* [45]. The tester must verify that the server does not crash with a `500 Internal Server Error`, but instead gracefully returns a valid, descriptive error message to the client, preventing "orphaned" transaction records in the database [45].

---

## 📂 4. The Three Sub-languages of SQL: DDL, DQL, and DML
SQL (Structured Query Language) is a family of sub-languages, each designed to solve specific types of database tasks [10, 11].

| SQL Sub-language | Definition | Target Scope | Key Commands | QA Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **DDL** [10] | **Data Definition Language** | Table and database **structures** (schemas) [10] | `CREATE`, `ALTER`, `DROP` [10] | Used in automation testing during database initialization (e.g., creating temporary schemas or tables for test runs). |
| **DQL** [10, 11] | **Data Query Language** | Column-level **queries** and data retrieval [10, 11] | `SELECT` [10, 11] | The primary daily tool for manual and automated QA engineers to verify backend state after performing UI or API actions. |
| **DML** [11] | **Data Manipulation Language** | Actual data **records** (rows) inside tables [11] | `INSERT`, `UPDATE`, `DELETE` [11] | **Data Seeding & Test Data Preparation.** Rapidly setting up a specific user state directly in the database without going through UI forms. |

### 💡 DML QA Case Study (Test Data Preparation)
Instead of spending minutes manually going through a long UI registration wizard, clicking confirmation emails, manually accumulating bonus points, and waiting for a VIP status upgrade, a QA engineer can execute a single, rapid DML statement before running their test [47]:

```sql
UPDATE users 
SET status = 'VIP', email_verified = 1, loyalty_points = 5000 
WHERE user_id = 1042;
```
This saves hours of testing time when verifying complex features.

---

## 🛠️ 5. Practical MySQL Workbench Configurations
To help students successfully complete their homework assignments and write queries, they need to apply a few baseline configurations in MySQL Workbench [27].

### 1. Disabling Safe Updates (Safe Mode)
By default, MySQL Workbench blocks any data modification (`UPDATE`) or deletion (`DELETE`) queries that do not explicitly target a primary key column in the `WHERE` clause [18, 48]. While this is a safety feature for production environments, it hinders test data preparation [18, 48].

*   **Step-by-Step Guide for Students:**
    1. Navigate to the top menu: **Edit ➔ Preferences** [18, 48].
    2. Select the **SQL Editor** tab from the left sidebar [18, 48].
    3. Scroll down to the bottom of the page and **uncheck** the checkbox: **"Safe Updates (rejects UPDATEs and DELETEs with no restrictions)"** [18, 48].
    4. **Crucial Step:** Disconnect and **reconnect** to your database instance (or restart MySQL Workbench) to apply the changes [18, 48].

### 2. Code Formatting & Comments in SQL
Encouraging students to write clean, readable SQL queries from day one is an essential habit. Comments are also invaluable for documenting individual test steps inside SQL scripts [25, 49].
*   **Single-line comments:** Start with two hyphens followed by a space: `-- comment text` [25, 49].
*   **Multi-line comments:** Enclosed within slash-star boundaries: `/* comment text */` [25, 49].

### 3. Generating EER Diagrams (Enhanced Entity Relationship)
An EER diagram acts as a visual roadmap of a database, showing exactly how tables are interconnected via Primary and Foreign Keys [25, 49].

*   **How to generate an EER Diagram in Workbench:**
    1. In the top menu, select **Database ➔ Reverse Engineer...** [26, 49]
    2. Click **Next**, then enter your local database password [26, 49].
    3. Select the **classicmodels** schema, click **Next**, and click **Execute** [26, 49].
*   **Value for QA:** It allows the QA engineer to quickly grasp the architecture of the entire system before creating integration test plans.

---

## 🔍 6. Troubleshooting Common Student Mistakes (Review Case Study!)
During the database setup phase, many students encounter SQL Error Code 1217 when running the initialization script:  
`Error Code: 1217. Cannot drop table productlines referenced by foreign key` [31, 34, 50].

### Why does this error occur?
At the very beginning of the setup script, there are commands to clean up existing tables so they can be fresh-started (e.g., `DROP TABLE IF EXISTS productlines;`) [50]. However, `productlines` is a parent table [50]. Columns in other tables (such as `products`) contain Foreign Keys pointing directly to the Primary Key of `productlines` [31, 50]. 

MySQL blocks the deletion (`DROP`) of the parent table to protect referential integrity [34, 50]. If it allowed the deletion, the child tables would end up with "orphaned" keys referencing a non-existent table [34, 50].

### How to explain this to students during the review:
1.  **The data is already safe:** Explain that this error only occurs on **subsequent runs** of the script after the database has already been successfully created once [37, 51]. Because all tables have already been created and populated with data on the first run, there is absolutely no need to re-run this entire setup script [37, 51]. The database is fully operational and ready for querying [37, 51].
2.  **How developers and automated pipelines solve this:** When developers or automated test suites need to completely tear down and rebuild a database schema, they temporarily turn off Foreign Key constraint checks before dropping tables, and then re-enable them afterwards:

```sql
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE IF EXISTS productlines;
DROP TABLE IF EXISTS products;
-- [Rest of the DROP and CREATE statements]
SET FOREIGN_KEY_CHECKS = 1;
```

---

## 🎓 Reviewer Checklist: Session Activities
Use this timeline to run an engaging 50-minute review session:

*   [ ] **Icebreaker (5 mins):** Ask students who successfully installed MySQL Workbench on their first try and who had to troubleshoot.
*   [ ] **Costco Receipt Interactive (10 mins):** Project a physical receipt on-screen. Ask students to name the database tables they would design to store this data without duplication [42].
*   [ ] **Key Hunt (10 mins):** Have students open their generated EER Diagram of `classicmodels` [27, 49]. Challenge them to find a table containing both a Primary Key (yellow key/lamp icon) and a Foreign Key (red diamond icon) [27, 49]. Ask them to explain the relationship in their own words.
*   [ ] **Error Deconstruction (10 mins):** Walk through the `Cannot drop table...` error on screen [31]. Explain why the database engine prevents this drop to safeguard data integrity [34, 50].
*   [ ] **Q&A & Homework Prep (15 mins):** Answer technical questions and help anyone who is still struggling to query their local database.
