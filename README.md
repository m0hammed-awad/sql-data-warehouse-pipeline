# SQL Data Warehouse Pipeline

> An end-to-end SQL Server data engineering project for building a modern data warehouse using ETL, data quality, layered architecture, and dimensional modeling.

## 📋 Project Requirements

### Business Requirements

The data warehouse should provide a centralized and reliable source of data for business analysis and reporting.

The system should:

- Consolidate data from CRM and ERP source systems.
- Provide a unified view of customers, products, and sales.
- Support sales performance analysis.
- Support customer and product analysis.
- Provide clean, consistent, and reliable data for reporting.
- Preserve historical business data for analytical purposes.
- Simplify querying and analysis across multiple source systems.

### Technical Requirements

The project should:

- Use Microsoft SQL Server as the data warehouse platform.
- Import data from CSV source files.
- Implement a Bronze, Silver, and Gold layered architecture.
- Load raw source data into the Bronze layer.
- Clean and transform data in the Silver layer.
- Integrate and organize business data in the Gold layer.
- Implement dimensional modeling.
- Create fact and dimension tables.
- Define appropriate primary and foreign keys.
- Handle NULL, duplicate, invalid, and inconsistent data.
- Use T-SQL for data transformation and processing.
- Create reusable SQL scripts and stored procedures where appropriate.
- Include data quality checks.
- Document the data architecture and data model.
- Use Git and GitHub for version control.

### Data Requirements

The warehouse should contain information related to:

- Customers
- Products
- Product categories
- Sales transactions
- Customer attributes
- Product attributes
- Sales dates
- Quantities
- Sales amounts

### Analytics Requirements

The final Gold layer should allow users to answer questions such as:

- How much was sold over a specific period?
- Which products generate the most sales?
- Which customers generate the most revenue?
- How are sales performing over time?
- Which product categories perform best?
- What are the purchasing patterns of customers?

---

## 🏗️ Data Architecture

The project follows a layered Medallion Architecture:

```text
                    Source Systems
                   /              \
                 CRM              ERP
                  \                /
                   \              /
                    ▼            ▼
                  ┌───────────────┐
                  │ Bronze Layer  │
                  │   Raw Data    │
                  └───────┬───────┘
                          │
                          ▼
                  ┌───────────────┐
                  │ Silver Layer  │
                  │ Cleaned Data  │
                  └───────┬───────┘
                          │
                          ▼
                  ┌───────────────┐
                  │  Gold Layer   │
                  │ Analytics Data│
                  └───────┬───────┘
                          │
                          ▼
                  Reporting & Analysis
