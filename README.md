
```markdown
# 📊 Enterprise Sales Data Warehouse (Medallion Architecture)

## 📌 Executive Summary
This project demonstrates the end-to-end design and implementation of an Enterprise Data Warehouse (EDW) built on **Microsoft SQL Server (T-SQL)**.

The system integrates raw transactional CRM and ERP datasets into a unified analytical environment. By leveraging the **Medallion Architecture** pattern (Bronze ➔ Silver ➔ Gold), raw data is cleansed, standardized, deduplicated, and transformed into a high-performance **Star Schema** optimized for Business Intelligence (BI) and reporting.

---

## 🏗️ Data Architecture & Pipeline

The warehouse architecture is divided into three logical layers:


```

+------------------+      +--------------------+      +------------------+
|   Bronze Layer   | ---> |    Silver Layer    | ---> |    Gold Layer    |
|   (Raw Ingest)   |      | (Clean & Enriched) |      |   (Star Schema)  |
+------------------+      +--------------------+      +------------------+
| - CRM Tables     |      | - Data Cleansing   |      | - dim_customers  |
| - ERP Tables     |      | - Deduplication    |      | - dim_products   |
| (Full Lineage)   |      | - Automated ETL    |      | - fact_sales     |
+------------------+      +--------------------+      +------------------+

```

### 1. 🥉 Bronze Layer (Raw Ingestion)
- **Purpose**: Stores raw ingested data as-is from CRM and ERP source systems without modifications to preserve auditability and data lineage.
- **Tables Included**:
  - CRM: `crm_cust_info`, `crm_prd_info`, `crm_sales_details`
  - ERP: `erp_cust_az12`, `erp_px_cat_g1v2`, `erp_loc_a101`

---

### 2. 🥈 Silver Layer (Data Cleansing & Transformation)
- **Purpose**: Cleanses, standardizes, enriches, and deduplicates raw data from the Bronze layer.
- **Key Transformations**:
  - **Deduplication**: Eliminates duplicate customer records using window functions.
  - **Data Normalization**: Standardizes gender codes ('M' ➔ 'Male', 'F' ➔ 'Female') and marital status ('S' ➔ 'Single', 'M' ➔ 'Married').
  - **Trimming & NULL Handling**: Removes trailing whitespace via `TRIM()` and replaces missing numerical values with fallbacks.
  - **Automated Pipeline**: Encapsulated within a T-SQL Stored Procedure (`silver.load_silver`) featuring automated execution timing and `TRY...CATCH` error handling.

---

### 3. 🥇 Gold Layer (Analytical Dimensional Modeling)
- **Purpose**: Exposes business-ready analytical views formatted in a **Star Schema** for direct querying or consumption by Power BI / Tableau.
- **Entities**:
  - `gold.dim_customers`: Combines CRM and ERP customer profiles, resolves country/gender mismatches, and creates a surrogate key.
  - `gold.dim_products`: Merges product details with category mappings, filters out inactive/historical records (`WHERE prd_end_dt IS NULL`), and assigns a surrogate key.
  - `gold.fact_sales`: Fact table linking sales transactions to customer and product dimension keys alongside sales measures (sales amount, quantity, price).

---

## 🛠 Technical Highlights & Window Functions

### Advanced SQL & Window Functions
To maintain high data quality and schema integrity across layers, advanced **Window Functions** were utilized:

1. **Deduplication in Silver Layer**:
   ```sql
   ROW_NUMBER() OVER (PARTITION BY cst_id ORDER BY cst_create_date DESC) AS flag

```

Filters out historical/duplicate customer entries to retain only the most recent profile (`WHERE flag = 1`).

2. **Surrogate Key Generation in Gold Layer**:
```sql
ROW_NUMBER() OVER (ORDER BY cst_id) AS customer_key
ROW_NUMBER() OVER (ORDER BY prd_start_dt, prd_key) AS product_key

```


Generates unique, integer surrogate keys for dimension tables to decouple the analytical model from natural source keys.

---

## 📸 Architecture Diagram & Data Model

---

## 📂 Project Directory Structure

```text
sql-data-warehouse-project/
│
├── scripts/
│   ├── 01_bronze_layer.sql    -- DDL scripts for Bronze tables
│   ├── 02_silver_layer.sql    -- ETL Stored Procedures & data cleansing logic
│   └── 03_gold_layer.sql      -- Dimension and Fact Views (Star Schema)
│
├── docs/
│   ├── architecture_diagram.png -- Visual diagram of Data Pipeline & Model
│   └── data_dictionary.md       -- Detailed field mappings & metadata
│
├── power_bi/                  -- (Optional) Power BI Report (.pbix)
│   └── sales_analytics.pbix
│
├── .gitignore
└── README.md                  -- Main project documentation

```

---

## 🚀 How to Run the Project

1. **Setup Database**:
Open SQL Server Management Studio (SSMS) and create a new database.
2. **Execute Scripts in Sequence**:
* Run `scripts/01_bronze_layer.sql` to instantiate Bronze tables.
* Run `scripts/02_silver_layer.sql` to build and execute the ETL pipeline procedure:
```sql
EXEC silver.load_silver;

```


* Run `scripts/03_gold_layer.sql` to construct the Gold analytical views.


3. **Connect BI Tools**:
Connect Power BI or Tableau to the `gold` views (`gold.dim_customers`, `gold.dim_products`, `gold.fact_sales`).

---

## 🧰 Tech Stack

* **Database Engine**: Microsoft SQL Server (T-SQL)
* **Modeling**: Star Schema (Fact & Dimension Tables)
* **ETL Processes**: Stored Procedures, Window Functions, DDL/DML, Error Handling (`TRY...CATCH`)
* **Version Control**: Git & GitHub

```

```
