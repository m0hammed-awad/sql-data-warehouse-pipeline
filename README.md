<div align="center">

# 🏢 SQL Server Data Warehouse & Analytics Project

### A modern data warehouse built with SQL Server, following the Medallion Architecture

[![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=flat&logo=microsoft-sql-server&logoColor=white)](https://www.microsoft.com/sql-server)
[![T-SQL](https://img.shields.io/badge/T--SQL-4479A1?style=flat&logo=databricks&logoColor=white)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-active-success.svg)](#)

</div>

---

## 📌 Overview

This project implements a **modern SQL Server data warehouse** that integrates customer, product, and sales data from two source systems — **CRM** and **ERP** — into a single, analytics-ready model.

The warehouse follows a **Medallion Architecture** with three progressive layers:

| Layer | Purpose |
|-------|---------|
| 🥉 **Bronze** | Raw, as-is ingestion from source CSV files |
| 🥈 **Silver** | Cleaned, standardized, validated, and integrated data |
| 🥇 **Gold** | Business-ready **Star Schema** for analytics & reporting |

The final Gold layer is consumable directly by BI tools, ad-hoc SQL analysis, and downstream ML workloads.

---

## 📑 Table of Contents

- [Objectives](#-objectives)
- [Architecture](#️-architecture)
- [Data Flow](#-data-flow)
- [Data Integration](#-data-integration)
- [Bronze Layer](#-bronze-layer)
- [Silver Layer](#-silver-layer)
- [Gold Layer](#-gold-layer)
- [Sales Data Mart](#-sales-data-mart)
- [Data Quality](#-data-quality)
- [Project Structure](#-project-structure)
- [How to Run](#-how-to-run-the-project)
- [End-to-End Workflow](#-end-to-end-workflow)
- [Technology Stack](#️-technology-stack)
- [Key Concepts Demonstrated](#-key-data-engineering-concepts-demonstrated)
- [Future Improvements](#-future-improvements)
- [Documentation](#-documentation)
- [Author](#-author)
- [License](#-license)

---

## 🎯 Objectives

- Build a complete SQL Server data warehouse from raw CSV sources.
- Integrate data from **CRM** and **ERP** systems.
- Separate raw ingestion from transformation and business logic.
- Clean and standardize inconsistent source data.
- Handle invalid dates, missing values, duplicate customer records, and inconsistent categorical values.
- Enrich CRM data with ERP customer, location, and product-category information.
- Build a business-ready **Sales Data Mart**.
- Implement reusable stored procedures for Bronze and Silver loading.
- Create data-quality checks to validate the warehouse.
- Document the architecture, data flow, and data model.

---

## 🏗️ Architecture

The project follows a layered data warehouse architecture, moving data from raw CSV files through progressively refined layers until it reaches a consumable, business-ready state.

<p align="center">
  <img src="docs/high level architecture.png" alt="High-Level Architecture" width="850">
</p>

```text
CRM / ERP CSV Sources
        │
        ▼
┌───────────────────────────┐
│       BRONZE LAYER        │   Raw tables · Truncate & Insert · No transformations
└─────────────┬─────────────┘
              ▼
┌───────────────────────────┐
│       SILVER LAYER        │   Cleaning · Standardization · Deduplication · Integration prep
└─────────────┬─────────────┘
              ▼
┌───────────────────────────┐
│        GOLD LAYER         │   Star Schema · dim_customers · dim_products · fact_sales
└─────────────┬─────────────┘
              ▼
     Analytics / Reporting / Ad-Hoc SQL
```

> 📎 Diagram source: `docs/high level architecture.png`

---

## 🔄 Data Flow

Data flows from two source systems, through Bronze and Silver, into the final Gold-layer objects.

<p align="center">
  <img src="docs/Data flow.png" alt="Data Flow Diagram" width="850">
</p>

**CRM** provides:
- Customer information
- Product information
- Sales transaction details

**ERP** provides:
- Additional customer information
- Customer country / location information
- Product category information

Data is loaded into **Bronze**, transformed in **Silver**, and integrated into **Gold**.

> 📎 Diagram source: `docs/Data flow.png`

---

## 🔗 Data Integration

CRM and ERP data are joined in the Silver/Gold layers via shared business keys.

<p align="center">
  <img src="docs/Data integration.png" alt="Data Integration Diagram" width="850">
</p>

| CRM Data | ERP Data | Purpose |
|---|---|---|
| `crm_cust_info.cst_key` | `erp_cust_az12.cid` | Customer enrichment |
| `crm_cust_info.cst_key` | `erp_loc_a101.cid` | Customer country |
| `crm_prd_info.cat_id` | `erp_px_cat_g1v2.id` | Product category enrichment |
| `crm_sales_details.sls_prd_key` | `dim_products.product_number` | Sales-to-product relationship |
| `crm_sales_details.sls_cust_id` | `dim_customers.customer_id` | Sales-to-customer relationship |

> 📎 Diagram source: `docs/Data integration.png`

---

## 🥉 Bronze Layer

The raw ingestion layer — data lands exactly as it appears in the source.

**Characteristics**
- Source data loaded as-is, with no major transformations.
- Loaded via batch processing.
- Tables refreshed using `TRUNCATE` followed by `INSERT`.
- Loading automated through stored procedures.

**CRM Bronze Tables**
- `bronze.crm_cust_info`
- `bronze.crm_prd_info`
- `bronze.crm_sales_details`

**ERP Bronze Tables**
- `bronze.erp_cust_az12`
- `bronze.erp_loc_a101`
- `bronze.erp_px_cat_g1v2`

**Loading procedure:** [`scripts/bronze/proc_load_bronze.sql`](scripts/bronze/proc_load_bronze.sql)

---

## 🥈 Silver Layer

Cleaned, standardized data that makes the source data reliable and consistent before it reaches the business-facing Gold layer.

<details>
<summary><b>Customer Data</b></summary>

- Trimmed first and last names.
- Standardized marital status and gender values.
- Removed duplicate customer records using `ROW_NUMBER()`.
- Kept the latest record based on creation date:

```sql
ROW_NUMBER() OVER (
    PARTITION BY cst_id
    ORDER BY cst_create_date DESC
)
```
</details>

<details>
<summary><b>Product Data</b></summary>

- Standardized category IDs.
- Extracted product numbers from product keys.
- Replaced missing product costs with `0`.
- Standardized product-line values.
- Generated product validity periods using `LEAD()`.
</details>

<details>
<summary><b>Sales Data</b></summary>

- Validated order, shipping, and due dates.
- Converted source date values into SQL `DATE`.
- Recalculated invalid sales amounts.
- Corrected invalid or missing prices.
- Prevented division-by-zero using `NULLIF()`.
</details>

<details>
<summary><b>ERP Customer Data</b></summary>

- Removed the `NAS` prefix from customer identifiers where applicable.
- Converted future birthdates to `NULL`.
- Standardized gender values.
</details>

<details>
<summary><b>ERP Location Data</b></summary>

- Removed formatting characters from customer IDs.
- Standardized country names and country codes.
- Handled missing country values.
</details>

<details>
<summary><b>ERP Product Categories</b></summary>

- Loaded and standardized product category information.
</details>

**Silver Tables**

| CRM | ERP |
|---|---|
| `silver.crm_cust_info` | `silver.erp_cust_az12` |
| `silver.crm_prd_info` | `silver.erp_loc_a101` |
| `silver.crm_sales_details` | `silver.erp_px_cat_g1v2` |

**Loading procedure:** [`scripts/silver/proc_load_silver.sql`](scripts/silver/proc_load_silver.sql)

---

## 🥇 Gold Layer

The business-ready analytical model, exposed through SQL **views** that build the final dimensional model from Silver data, following a **Star Schema**.

### `gold.dim_customers`
Enriched customer information from CRM and ERP sources.

`customer_key` · `customer_id` · `customer_number` · `first_name` · `last_name` · `country` · `marital_status` · `gender` · `birthdate` · `create_date`

> CRM is treated as the primary source for gender, with ERP used as a fallback when CRM gender is unavailable.

### `gold.dim_products`
Current version of product information, enriched with ERP category data.

`product_key` · `product_id` · `product_number` · `product_name` · `category_id` · `category` · `subcategory` · `maintenance` · `cost` · `product_line` · `start_date`

> Historical product records are filtered out so the dimension reflects only the current product state.

### `gold.fact_sales`
Sales transactions connected to the customer and product dimensions via surrogate keys.

`order_number` · `product_key` · `customer_key` · `order_date` · `shipping_date` · `due_date` · `sales_amount` · `quantity` · `price`

**DDL:** [`scripts/gold/ddl_gold.sql`](scripts/gold/ddl_gold.sql)

---

## ⭐ Sales Data Mart

<p align="center">
  <img src="docs/Sales Data Mart.png" alt="Sales Data Mart Star Schema" width="850">
</p>

```text
        ┌────────────────────────┐
        │   gold.dim_customers   │
        │------------------------│
        │ PK customer_key        │
        │    customer_id         │
        │    customer_number     │
        │    first_name          │
        │    last_name           │
        │    country             │
        │    gender              │
        │    birthdate           │
        └───────────┬────────────┘
                     │ FK
                     ▼
        ┌────────────────────────┐
        │     gold.fact_sales    │
        │------------------------│
        │    order_number        │
        │ FK product_key         │
        │ FK customer_key        │
        │    order_date          │
        │    shipping_date       │
        │    due_date            │
        │    sales_amount        │
        │    quantity            │
        │    price               │
        └───────────┬────────────┘
                     │ FK
                     ▼
        ┌────────────────────────┐
        │    gold.dim_products   │
        │------------------------│
        │ PK product_key         │
        │    product_id          │
        │    product_number      │
        │    product_name        │
        │    category_id         │
        │    category            │
        │    subcategory         │
        │    maintenance         │
        │    cost                │
        │    product_line        │
        └────────────────────────┘
```

**Sales Calculation:** `sales_amount = quantity × price`

This model enables analysts to answer questions such as:

- What are total sales by product?
- Which products generate the most revenue?
- Which customers generate the highest sales?
- What are sales trends over time?
- How does sales performance vary by country?
- What product categories contribute most to revenue?
- What is the relationship between quantity, price, and sales?

> 📎 Diagram source: `docs/Sales Data Mart.png`

---

## 🧪 Data Quality

Data quality checks validate the warehouse before the Gold layer is consumed.

| Script | Purpose |
|---|---|
| [`tests/quality_checks.sql`](tests/quality_checks.sql) | Validates Bronze → Silver transformations |
| [`tests/quality_checks_gold.sql`](tests/quality_checks_gold.sql) | Validates the final Gold-layer relationships |

**Checks include:**
- Duplicate customer detection
- Null-value checks
- Invalid date checks
- Invalid product keys
- Invalid sales calculations
- Referential integrity between facts and dimensions
- Validation of Gold-layer relationships

---

## 📂 Project Structure

```text
.
├── datasets/
│   └── placeholder
│
├── docs/
│   ├── data_catalog.md
│   ├── Data flow.png
│   ├── Data integration.png
│   ├── high level architecture.png
│   └── Sales Data Mart.png
│
├── scripts/
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   │
│   ├── gold/
│   │   └── ddl_gold.sql
│   │
│   └── init_database.sql
│
├── tests/
│   ├── quality_checks.sql
│   └── quality_checks_gold.sql
│
├── LICENSE
└── README.md
```

---

## 🚀 How to Run the Project

### 1. Clone the Repository
```bash
git clone <YOUR_REPOSITORY_URL>
cd <YOUR_REPOSITORY_FOLDER>
```

### 2. Initialize the Database
```sql
-- Run:
scripts/init_database.sql
```
Creates and prepares the database environment.

### 3. Create the Bronze Layer
```sql
-- Run:
scripts/bronze/ddl_bronze.sql
```
Creates the Bronze tables.

### 4. Load the Bronze Layer
```sql
-- Run:
scripts/bronze/proc_load_bronze.sql
```
Executes the loading procedure to ingest CRM and ERP source data.

### 5. Create the Silver Layer
```sql
-- Run:
scripts/silver/ddl_silver.sql
```
Creates the Silver tables.

### 6. Load the Silver Layer
```sql
-- Run:
scripts/silver/proc_load_silver.sql
```
Cleans, standardizes, and transforms the Bronze data into the Silver layer.

### 7. Validate Data Quality
```sql
-- Run:
tests/quality_checks.sql

-- Then, after creating the Gold layer, validate with:
tests/quality_checks_gold.sql
```

### 8. Create the Gold Layer
```sql
-- Run:
scripts/gold/ddl_gold.sql
```
Creates the final Gold-layer views: `gold.dim_customers`, `gold.dim_products`, `gold.fact_sales`.

---

## 🔁 End-to-End Workflow

```text
CSV Files
   │
   ├── CRM
   │
   └── ERP
        │
        ▼
   Bronze Layer
   Raw Tables
        │
        │ Batch Load
        ▼
   Silver Layer
   Clean + Standardized Tables
        │
        │ Integration + Business Logic
        ▼
   Gold Layer
   Star Schema
        │
        ├── dim_customers
        ├── dim_products
        └── fact_sales
        │
        ▼
   Analytics / Reporting / Ad-Hoc SQL
```

---

## ⚙️ Technology Stack

| Technology | Purpose |
|---|---|
| Microsoft SQL Server | Data warehouse database |
| T-SQL | Data transformation and querying |
| SQL Server Management Studio / VS Code | Development and database management |
| CSV | Source data format |
| Git & GitHub | Version control and project management |
| Medallion Architecture | Data warehouse layering |
| Star Schema | Gold-layer dimensional modeling |

---

## 🧠 Key Data Engineering Concepts Demonstrated

- Data warehouse architecture & Medallion architecture
- ETL/ELT concepts and batch data ingestion
- SQL Server & T-SQL, stored procedures
- Data cleansing, standardization, validation, and deduplication
- Data integration via business keys
- Surrogate keys, fact and dimension modeling
- Star schema design
- Data quality testing
- SQL views
- Git/GitHub project organization
- Technical documentation

---

## 📈 Future Improvements

- [ ] Add an orchestration tool such as Apache Airflow
- [ ] Add incremental loading instead of full refreshes
- [ ] Add automated data-quality execution
- [ ] Add logging and audit tables for pipeline monitoring
- [ ] Add load duration and row-count metrics
- [ ] Connect the Gold layer to Power BI
- [ ] Add automated CI/CD validation for SQL scripts
- [ ] Add additional analytical marts and business metrics

---

## 📚 Documentation

Additional documentation is available in the [`docs/`](docs/) directory:

| File | Description |
|---|---|
| [`docs/data_catalog.md`](docs/data_catalog.md) | Warehouse tables, columns, business meaning, and data-layer information |
| `docs/high level architecture.png` | Layered warehouse architecture diagram |
| `docs/Data flow.png` | End-to-end data movement diagram |
| `docs/Data integration.png` | CRM/ERP source integration diagram |
| `docs/Sales Data Mart.png` | Final Gold-layer star schema |

---

## 👤 Author

**Mohamed Awad**
Computer Science & Engineering
Interested in Data Engineering, Data Analytics, SQL, and Data Warehousing.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

<div align="center">

⭐ If you found this project useful, consider giving it a star!

</div>
