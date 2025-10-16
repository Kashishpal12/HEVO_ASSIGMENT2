

# HEVO\_ASSIGMENT2: Messy E-Commerce Orders Transformation (Snowflake & Hevo Models)

**Author:** Kashish Pal

This repository documents the solution for the Hevo Assessment II, focused on transforming raw, messy e-commerce data into a clean, unified, and analytics-ready dataset. The approach leverages an **ELT (Extract, Load, Transform)** methodology, utilizing **Hevo Data** for ingestion and **Snowflake** as the powerful transformation layer, with all cleaning logic encapsulated in **Hevo Models**.

***NOTE - Please refer to the word document attached which have step by step detail process and the relevant screenshots*****

---

## 1. Phase I: Pipeline Setup and Data Ingestion

The initial phase involved preparing the source data (reusing the setup from Assignment 1) and configuring the Hevo pipeline.

### Source Data Preparation (PostgreSQL)

* **Tables Created:** All four raw source tables (`customers_raw`, `orders_raw`, `products_raw`, `country_dim`) were defined.
* **Data Loaded:** Sample data was inserted into these tables using standard `INSERT` statements.

### Hevo Pipeline Configuration: PostgreSQL $\rightarrow$ Snowflake

| Setting | Value |
| :--- | :--- |
| **Source** | PostgreSQL (Neon) |
| **Ingestion Mode** | Logical Replication (CDC) |
| **Destination** | Snowflake Partner Connect Trial Account |

#### 💡 Initial Issue & Fix: Logical Replication

| Issue Encountered | Fix Applied |
| :--- | :--- |
| **Hevo Error:** "Unable to use logical replication. `wal_level` must be set to 'logical'" | Logged in to **Neon $\rightarrow$ Settings** and explicitly enabled Logical Replication. Verified with `SHOW wal_level;`. |
| **Result:** | ✅ Connection established successfully. |

---

## 2. Phase II: Data Transformation via Hevo Models

All cleaning, standardization, and joining were executed as a sequence of SQL Models within the Hevo environment, running directly against the data in Snowflake.

### 📜 Model 1: `CUSTOMERS_CLEANED` (Tasks 5 & 6)
**Purpose:** Deduplicate records, standardize contact/location data, and handle nulls.

| Transformation | Logic Implemented |
| :--- | :--- |
| **Deduplication** | Used `ROW_NUMBER()` partitioned by `customer_id` and ordered by `updated_at DESC` to keep only the **most recent record**. |
| **Email & Phone** | Emails standardized to **lowercase** (`LOWER()`). Phone numbers standardized to **10 digits** or marked as **'Unknown'** using `REGEXP_REPLACE()` and `RLIKE`. |
| **Country Standard.** | Used a `LEFT JOIN` on `country_dim` to map variations (e.g., 'usa', 'SINGAPORE') to standard **ISO codes**. |
| **Null Handling** | Replaced `<null>` `created_at` values with a default timestamp: **`1900-01-01`**. |

### 📜 Model 2: `PRODUCTS_CLEANED` (Task 8)
**Purpose:** Standardize product names and categories, and flag discontinued items.

| Transformation | Logic Implemented |
| :--- | :--- |
| **Name & Category** | Standardized both `product_name` and `category` to **Title Case** using `INITCAP()`. |
| **Inactive Products** | Used a `CASE` statement to flag products where `active_flag = 'N'` as **'Discontinued Product'**. |

### 📜 Model 3: `ORDERS_CLEANED` (Task 7)
**Purpose:** Remove duplicates, clean negative/null amounts, standardize currency, and derive USD equivalent.

| Transformation | Logic Implemented |
| :--- | :--- |
| **Deduplication** | Used `SELECT DISTINCT` to remove exact duplicate rows. |
| **Invalid Amounts** | Negative amounts replaced with **`0.00`**. |
| **Null Amounts** | Replaced `<null>` amounts with the calculated **median amount per customer** (`MEDIAN() OVER (PARTITION BY customer_id)`), falling back to `0.00` if the median was not available. |
| **Currency & USD Conv.** | Currency codes standardized to **uppercase**. `amount_usd` derived using predefined conversion rates (e.g., INR=0.012, SGD=0.74, EUR=1.07). |

### 📜 Model 4: `FINAL_UNIFIED_DATASET` (Tasks 9 & 10)
**Purpose:** Produce the final analytics-ready dataset by joining all three cleaned models and handling all specified edge cases. This model used **`LEFT JOIN`** to ensure all orders, including orphaned ones, were preserved.

| Edge Case Handled | Logic Implemented |
| :--- | :--- |
| **Orphan Customer** | If an order's `customer_id` had no match in `CUSTOMERS_CLEANED`, the email was labeled **'Orphan Customer'**. |
| **Unknown Product** | If an order's `product_id` was missing or did not match, the product name was labeled **'Unknown/Discontinued Product'**. |
| **Invalid Customer** | Customers with completely null raw records (e.g., Customer ID 108) were specifically flagged with the email **'Invalid Customer'**. |

---

## 3. Conclusion and Deliverables

✅ **Summary:** Successfully implemented an ELT pipeline, handled logical replication setup, performed complex data quality checks (deduplication, standardization, null handling), and integrated data into a unified Snowflake table via Hevo Models.


