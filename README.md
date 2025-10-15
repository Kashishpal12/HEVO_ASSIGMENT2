HEVO_ASSIGMENT2: Messy E-Commerce Orders Transformation (Snowflake & Hevo Models)
This repository details the solution for the Hevo Assessment II, focused on transforming raw, messy e-commerce data into a clean, unified, and analytics-ready dataset using an ELT (Extract, Load, Transform) approach.

The solution leverages Hevo Data for ingestion and Snowflake as the powerful transformation layer, with all complex cleaning logic encapsulated in Hevo Models.

1. Phase I: Pipeline Setup and Data Ingestion
A. Source Setup (PostgreSQL)
The source data environment was prepared using a local PostgreSQL instance (reusing the setup from Assignment 1):

Tables Created: All four raw source tables (customers_raw, orders_raw, products_raw, country_dim) were defined.

Data Loaded: Sample data provided in the assessment was loaded using standard INSERT statements.

B. Hevo Pipeline Configuration
A Hevo pipeline was configured for robust and efficient data movement:

Source: PostgreSQL (connected via local networking).

Ingestion Mode: Logical Replication (CDC) was used to enable real-time change data capture.

Destination: Snowflake Partner Connect trial account.

Result: Hevo successfully performed the initial historical load, pushing all raw tables into Snowflake, ready for transformation.

2. Phase II: Data Transformation via Hevo Models
All data cleaning, standardization, and joining were executed as a sequence of SQL models within the Hevo environment, running directly against the data in Snowflake.

Model 1: CUSTOMERS_CLEANED (Tasks 5 & 6)
Purpose: Deduplicate records, standardize contact and location data, and handle nulls.

Transformation	Logic Implemented
Deduplication	Used ROW_NUMBER() partitioned by customer_id and ordered by updated_at DESC to keep only the most recent record.
Email & Phone	Emails standardized to lowercase (LOWER()). Phone numbers standardized to 10 digits or marked as 'Unknown' using REGEXP_REPLACE() and RLIKE.
Country Standard.	Used a LEFT JOIN on country_dim to map variations (e.g., 'usa', 'SINGAPORE') to standard ISO codes.
Null Handling	Replaced <null> created_at values with a default timestamp: 1900-01-01.

Export to Sheets
Model 2: PRODUCTS_CLEANED (Task 8)
Purpose: Standardize product names and categories, and flag discontinued items.

Transformation	Logic Implemented
Name & Category	Standardized both product_name and category to Title Case using INITCAP().
Inactive Products	Used a CASE statement to flag products where active_flag = 'N' as 'Discontinued Product'.

Export to Sheets
Model 3: ORDERS_CLEANED (Task 7)
Purpose: Remove duplicates, clean amounts, standardize currency, and derive USD equivalent.

Transformation	Logic Implemented
Deduplication	Used SELECT DISTINCT to remove exact duplicate rows.
Invalid Amounts	Negative amounts replaced with 0.00.
Null Amounts	Replaced <null> amounts with the calculated median amount per customer (MEDIAN() OVER (PARTITION BY customer_id)), falling back to 0.00 if the median was also null.
Currency & USD Conv.	Currency codes standardized to uppercase. amount_usd derived using predefined conversion rates (e.g., INR=0.012, SGD=0.74, EUR=1.07).

Export to Sheets
Model 4: FINAL_UNIFIED_DATASET (Tasks 9 & 10)
Purpose: Produce the final analytics-ready dataset by joining all three cleaned models and handling all edge cases.

This model used LEFT JOIN operations starting from ORDERS_CLEANED to ensure all orders were preserved, even if orphaned.

Edge Case Handled	Logic Implemented
Orphan Customer	
If an order's customer_id had no match in CUSTOMERS_CLEANED, the email was labeled 'Orphan Customer'.

Unknown Product	
If an order's product_id was missing or did not match, the product name was labeled 'Unknown/Discontinued Product'.

Invalid Customer	
Customers with completely null raw records (Customer ID 108) were specifically flagged with the email 'Invalid Customer'.
