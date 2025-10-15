HEVO_ASSIGMENT2: Messy E-Commerce Orders Transformation (Snowflake & Hevo Models)
Author: Kashish Pal

This repository details the solution for the Hevo Assessment II, focused on transforming raw, messy e-commerce data into a clean, unified, and analytics-ready dataset. The project implements a robust ELT (Extract, Load, Transform) architecture, leveraging Hevo Data for efficient ingestion and Snowflake as the powerful transformation engine, with all complex cleaning logic executed via Hevo Models.

1. Phase I: Pipeline Setup and Data Ingestion
This phase focused on establishing the source data environment and setting up the Hevo pipeline to load data into the data warehouse.

A. Source Setup (PostgreSQL)
The source environment was prepared using a local PostgreSQL instance:

Tables Created: All four required raw tables (customers_raw, orders_raw, products_raw, country_dim) were defined.

Data Loaded: Sample data was inserted into these tables using standard SQL INSERT statements as specified in the assignment.

B. Hevo Pipeline Configuration
A Hevo Data pipeline was configured for robust and efficient data movement:

Source: Local PostgreSQL database.

Ingestion Mode: Logical Replication (CDC) was used, enabling Change Data Capture for potential real-time synchronization.

Destination: Snowflake Partner Connect trial account.

Result: Hevo successfully performed the initial historical load, pushing all raw tables into the Snowflake destination, where they were ready for post-load transformation.

2. Phase II: Data Transformation via Hevo Models
All data cleaning, standardization, and integration logic were executed as a sequence of SQL models directly within the Hevo environment against the Snowflake data.

📜 Model 1: CUSTOMERS_CLEANED (Tasks 5 & 6)
Purpose: Deduplicate customers, standardize contact/location data, and handle nulls.

Transformation	Logic Implemented
Deduplication	Used ROW_NUMBER() partitioned by customer_id and ordered by updated_at DESC to keep only the most recent record.
Email & Phone	Emails standardized to lowercase (LOWER()). Phone numbers standardized to 10 digits or marked as 'Unknown' using REGEXP_REPLACE() and RLIKE.
Country Standard.	Used a LEFT JOIN on country_dim to map variations (e.g., 'usa', 'SINGAPORE') to standard ISO codes.
Null Handling	Replaced <null> created_at values with a default timestamp: 1900-01-01.

Export to Sheets
📜 Model 2: PRODUCTS_CLEANED (Task 8)
Purpose: Standardize product names and categories, and flag discontinued items.

Transformation	Logic Implemented
Name & Category	Standardized both product_name and category to Title Case using INITCAP().
Inactive Products	Used a CASE statement to flag products where active_flag = 'N' as 'Discontinued Product'.

Export to Sheets
📜 Model 3: ORDERS_CLEANED (Task 7)
Purpose: Remove duplicates, clean negative/null amounts, standardize currency, and derive USD equivalent.

Transformation	Logic Implemented
Deduplication	Used SELECT DISTINCT to remove exact duplicate rows.
Invalid Amounts	Negative amounts replaced with 0.00.
Null Amounts	Replaced <null> amounts with the calculated median amount per customer (MEDIAN() OVER (PARTITION BY customer_id)), falling back to 0.00 if the median was not available.
Currency & USD Conv.	Currency codes standardized to uppercase. amount_usd derived using predefined conversion rates (e.g., INR=0.012, SGD=0.74, EUR=1.07).

Export to Sheets
📜 Model 4: FINAL_UNIFIED_DATASET (Tasks 9 & 10)
Purpose: Produce the final analytics-ready dataset by joining all three cleaned models and handling all specified edge cases.

This final model used LEFT JOIN operations starting from ORDERS_CLEANED to guarantee that all orders were preserved, satisfying the requirement for orphaned records.

Edge Case Handled	Logic Implemented
Orphan Customer	If an order's customer_id had no match in CUSTOMERS_CLEANED, the email was labeled 'Orphan Customer'.
Unknown Product	If an order's product_id was missing or did not match, the product name was labeled 'Unknown/Discontinued Product'.
Invalid Customer	Customers with completely null raw records (e.g., Customer ID 108) were specifically flagged with the email 'Invalid Customer'.
