# 📚 Data Catalog — Gold Layer

## 📌 Overview

The **Gold Layer** represents the business-level data model of the Data Warehouse. It is designed to support **analytics, reporting, and business intelligence (BI)** use cases.

The Gold Layer follows a **Star Schema** design and consists of:

* **Dimension tables** — descriptive information about business entities.
* **Fact tables** — measurable business events and transactions.

---

## 🏗️ Gold Layer Architecture

```text
                    ┌─────────────────────┐
                    │  gold.dim_customers │
                    │─────────────────────│
                    │ customer_key (PK)   │
                    │ customer_id         │
                    │ customer_number     │
                    │ first_name          │
                    │ last_name            │
                    │ country             │
                    │ marital_status      │
                    │ gender              │
                    │ birthdate            │
                    └──────────┬──────────┘
                               │
                               │ customer_key
                               ▼
                    ┌─────────────────────┐
                    │   gold.fact_sales   │
                    │─────────────────────│
                    │ order_number        │
                    │ product_key (FK)    │
                    │ customer_key (FK)   │
                    │ order_date          │
                    │ shipping_date       │
                    │ due_date            │
                    │ sales_amount        │
                    │ quantity             │
                    │ price                │
                    └──────────┬──────────┘
                               │
                               │ product_key
                               ▼
                    ┌─────────────────────┐
                    │  gold.dim_products  │
                    │─────────────────────│
                    │ product_key (PK)    │
                    │ product_id          │
                    │ product_number      │
                    │ product_name        │
                    │ category_id         │
                    │ category             │
                    │ subcategory          │
                    │ maintenance_required│
                    │ cost                 │
                    │ product_line         │
                    │ start_date           │
                    └─────────────────────┘
```

---

# 📊 Data Catalog

## 1. `gold.dim_customers`

### Purpose

Stores customer information enriched with demographic and geographic attributes.

### Columns

| Column            | Data Type    | Description                                                      |
| ----------------- | ------------ | ---------------------------------------------------------------- |
| `customer_key`    | INT          | Surrogate key uniquely identifying each customer record.         |
| `customer_id`     | INT          | Unique numerical identifier assigned to the customer.            |
| `customer_number` | NVARCHAR(50) | Alphanumeric identifier representing the customer.               |
| `first_name`      | NVARCHAR(50) | Customer's first name.                                           |
| `last_name`       | NVARCHAR(50) | Customer's last name.                                            |
| `country`         | NVARCHAR(50) | Country where the customer resides.                              |
| `marital_status`  | NVARCHAR(50) | Customer's marital status, such as `Married` or `Single`.        |
| `gender`          | NVARCHAR(50) | Customer's gender, such as `Male`, `Female`, or `n/a`.           |
| `birthdate`       | DATE         | Customer's date of birth.                                        |
| `create_date`     | DATE         | Date when the customer record was created in the data warehouse. |

---

## 2. `gold.dim_products`

### Purpose

Provides descriptive information about products and their business attributes.

### Columns

| Column                 | Data Type    | Description                                             |
| ---------------------- | ------------ | ------------------------------------------------------- |
| `product_key`          | INT          | Surrogate key uniquely identifying each product record. |
| `product_id`           | INT          | Unique identifier assigned to the product.              |
| `product_number`       | NVARCHAR(50) | Structured alphanumeric code representing the product.  |
| `product_name`         | NVARCHAR(50) | Descriptive name of the product.                        |
| `category_id`          | NVARCHAR(50) | Unique identifier of the product category.              |
| `category`             | NVARCHAR(50) | High-level product classification.                      |
| `subcategory`          | NVARCHAR(50) | Detailed classification within the product category.    |
| `maintenance_required` | NVARCHAR(50) | Indicates whether the product requires maintenance.     |
| `cost`                 | INT          | Base cost of the product.                               |
| `product_line`         | NVARCHAR(50) | Product line or series, such as `Road` or `Mountain`.   |
| `start_date`           | DATE         | Date when the product became available.                 |

---

## 3. `gold.fact_sales`

### Purpose

Stores transactional sales data for analytical and reporting purposes.

### Columns

| Column          | Data Type    | Description                                      |
| --------------- | ------------ | ------------------------------------------------ |
| `order_number`  | NVARCHAR(50) | Unique identifier of the sales order.            |
| `product_key`   | INT          | Foreign key referencing `gold.dim_products`.     |
| `customer_key`  | INT          | Foreign key referencing `gold.dim_customers`.    |
| `order_date`    | DATE         | Date when the customer placed the order.         |
| `shipping_date` | DATE         | Date when the order was shipped.                 |
| `due_date`      | DATE         | Expected delivery date of the order.             |
| `sales_amount`  | INT          | Total sales amount generated by the transaction. |
| `quantity`      | INT          | Number of units sold.                            |
| `price`         | INT          | Selling price per unit.                          |

---

# 🔗 Relationships

The Gold Layer uses a **Star Schema** where `fact_sales` acts as the central fact table.

### Relationships

```text
fact_sales.customer_key
        │
        ▼
dim_customers.customer_key
```

```text
fact_sales.product_key
        │
        ▼
dim_products.product_key
```

### Primary Keys

| Table                | Primary Key    |
| -------------------- | -------------- |
| `gold.dim_customers` | `customer_key` |
| `gold.dim_products`  | `product_key`  |

### Foreign Keys

| Fact Table        | Foreign Key    | References                        |
| ----------------- | -------------- | --------------------------------- |
| `gold.fact_sales` | `customer_key` | `gold.dim_customers.customer_key` |
| `gold.fact_sales` | `product_key`  | `gold.dim_products.product_key`   |

---

# 🎯 Business Use Cases

The Gold Layer enables analytical questions such as:

* What are the total sales?
* Which products generate the highest revenue?
* Which customers generate the most revenue?
* Which product categories perform best?
* What are the sales trends over time?
* What is the average selling price?
* How many products are sold per category?
* Which countries generate the highest sales?

---

# 🏛️ Data Warehouse Layers

The complete data flow follows a **Medallion Architecture**:

```text
┌──────────────────┐
│  BRONZE LAYER    │
│                  │
│    Raw Data      │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  SILVER LAYER    │
│                  │
│ Cleaned &        │
│ Transformed Data │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   GOLD LAYER     │
│                  │
│ Business-Level   │
│ Data Model       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Analytics & BI   │
│                  │
│ Reports /        │
│ Dashboards       │
└──────────────────┘
```

---

# 📁 Gold Layer Tables

| Table                | Type      | Purpose             |
| -------------------- | --------- | ------------------- |
| `gold.dim_customers` | Dimension | Customer attributes |
| `gold.dim_products`  | Dimension | Product attributes  |
| `gold.fact_sales`    | Fact      | Sales transactions  |

---

## 📝 Notes

* Surrogate keys are used in the dimension tables to support the dimensional model.
* The fact table stores measurable business events.
* Dimension tables provide descriptive context for analyzing sales.
* The Gold Layer is designed for **business consumption**, reporting, and analytical workloads.
* The model follows a **Star Schema** to simplify analytical queries and BI reporting.
