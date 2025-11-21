# SQL Data Warehouse Project  
## End-to-End Data Warehouse Implementation (Bronze → Silver → Gold)

This project demonstrates the complete development of a SQL-based data warehouse using Microsoft SQL Server.  
It covers raw data ingestion, data cleansing and standardisation, and the creation of analytics-ready dimensional models.

The solution follows a **medallion architecture**, providing clarity, maintainability, and scalability across all warehouse layers.

---

## 1. Project Overview  

The purpose of this project is to design and build a structured data warehouse using CRM and ERP datasets.  
The warehouse is divided into:

- **Bronze Layer**: Raw, unmodified ingested data.  
- **Silver Layer**: Cleaned, standardised, and integrated tables.  
- **Gold Layer**: Final star-schema views for analytics.

This structure ensures data quality, clear traceability, and straightforward reporting.

---

## 2. Architecture Diagram (ASCII)

```
            +------------------+
            |     RAW DATA     |
            |   (CSV: CRM/ERP) |
            +---------+--------+
                      |
                      v
         +---------------------------+
         |        BRONZE LAYER       |
         | Raw ingestion of source   |
         | files (no transformations)|
         +-------------+-------------+
                       |
                       v
         +---------------------------+
         |        SILVER LAYER       |
         | Data cleansing, typing,   |
         | natural key alignment,    |
         | merging CRM + ERP         |
         +-------------+-------------+
                       |
                       v
         +---------------------------+
         |         GOLD LAYER        |
         | Star schema:              |
         |   - dim_customers         |
         |   - dim_products          |
         |   - fact_sales            |
         | Analytics-ready outputs   |
         +---------------------------+
```

---

## 3. Repository Structure  

```
/datasets               → Raw CSV files
/scripts
    /bronze            → Ingestion scripts
    /silver            → Cleansing + standardisation scripts
    /gold              → Star-schema dimensional views
/docs                  → Diagrams, notes, data dictionary
/tests                 → SQL quality checks
README.md              → Project documentation
```

---

## 4. Bronze Layer  
### Purpose  
The Bronze layer stores **raw operational data exactly as received**.  
No transformations, no logic, no changes.

### Actions Performed  
- Imported CRM and ERP CSV files into SQL Server.  
- Preserved original column names and formats.  
- Maintained raw structures for auditability and traceability.

### Raw Tables  
Examples:  
- `bronze.crm_cust_info_raw`  
- `bronze.crm_sales_details_raw`  
- `bronze.crm_prd_info_raw`  
- `bronze.erp_loc_a101_raw`  
- `bronze.erp_px_cat_g1v2_raw`  

---

## 5. Silver Layer  
### Purpose  
The Silver layer standardises the raw data, fixes inconsistencies, ensures consistent key structures, and prepares tables for analytical modeling.

### Key Transformations  

---

### 5.1 Customer Cleansing (`silver.crm_cust_info`)  
- Unified column names  
- Converted creation timestamps  
- Cleaned up gender values  
- Prepared CRM natural keys  
- Validated first/last names  
- Ensured consistent date formatting  

---

### 5.2 Product Cleansing (`silver.crm_prd_info`)  
- Converted numeric product cost  
- Standardised product line and category identifiers  
- Prepared start/end dates  
- Ensured valid keys for linking with ERP categories  

---

### 5.3 Sales Details (`silver.crm_sales_details`)  
- Casted quantity, price, and sales amounts as numeric  
- Normalised key values (`sls_prd_key`, `sls_cust_id`)  
- Converted all date fields (order, ship, due)  
- Removed inconsistent values  

---

### 5.4 ERP Customer Data (`silver.erp_cust_az12`)  
- Converted birthdates from string → DATE  
- Cleaned gender entries  
- Standardised customer identifiers  
- Prepared attributes for dimensional enrichment  

---

### 5.5 ERP Category Mapping (`silver.erp_px_cat_g1v2`)  
- Standardised category and subcategory labels  
- Cleaned maintenance attributes  
- Normalised product/category IDs  

---

### Result  
The Silver layer produces a set of clean, consistent, relational tables suitable for joining, surrogate key creation, and analytics modeling.

---

## 6. Gold Layer  
### Purpose  
The Gold layer implements the **dimensional model** (star schema) used for reporting, analytics, and BI tools.

Gold tables are created as **SQL views**, ensuring:  
- Reusability  
- Real-time updates when Silver data updates  
- Clean separation from transformation logic  

---

## 6.1 Dimension: Customers (`gold.dim_customers`)  

### Transformations  
- Joined CRM customer data with ERP customer-birthdate data.  
- Added country information via ERP location table.  
- Implemented a gender fallback rule:
  ```
  IF CRM gender = 'n/a' → use ERP gender
  ```
- Generated surrogate primary key using:
  ```
  ROW_NUMBER() OVER (ORDER BY cst_id)
  ```
- Selected final output columns required for analytics.

### Final Columns  
- customer_key  
- customer_id  
- customer_number  
- first_name  
- last_name  
- marital_status  
- gender  
- birthdate  
- country  
- create_date  

---

## 6.2 Dimension: Products (`gold.dim_products`)  

### Transformations  
- Joined CRM products with ERP category definitions.  
- Filtered to include **active products only** (`prd_end_dt IS NULL`).  
- Generated surrogate product key:
  ```
  ROW_NUMBER() OVER (ORDER BY prd_start_dt, prd_key)
  ```
- Mapped category, subcategory, cost, line, and maintenance attributes.

### Final Columns  
- product_key  
- product_id  
- product_number  
- product_name  
- category_id  
- category  
- subcategory  
- maintenance  
- cost  
- product_line  
- start_date  

---

## 6.3 Fact Table: Sales (`gold.fact_sales`)  

### Transformations  
- Joined sales details with both dimensions:
  - `gold.dim_products` using product number  
  - `gold.dim_customers` using customer ID  
- Provided both surrogate and natural keys  
- Cleaned metrics (sales, quantity, price)  
- Preserved all date attributes  

### Final Columns  
- order_number  
- product_key  
- customer_key  
- customer_id  
- order_date  
- shipping_date  
- due_date  
- sales_amount  
- quantity  
- price  

---

## 7. Analytical Use Cases  
The final star schema supports questions such as:

- Total sales by customer, country, or demographic  
- Product performance by category or subcategory  
- Sales quantity vs. revenue trends  
- Customer-product segmentation  
- Temporal analysis (order vs shipping vs due dates)

These use cases are unlocked through the unified and cleansed data model in the Gold layer.

---

## 8. How to Run the Project  

1. Install SQL Server.  
2. Create a new database.  
3. Run scripts in the following order:
   - `/scripts/bronze`
   - `/scripts/silver`
   - `/scripts/gold`
4. Load raw CSVs into Bronze tables.  
5. Refresh Silver and Gold views.

---

## 9. Conclusion  

This project implements a complete medallion-based data warehouse, transforming operational CRM and ERP data from raw ingestion to analytics-ready dimensional models.  
The structured Bronze → Silver → Gold approach ensures data reliability, consistency, and usability for analysis.




## 🛡️ License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and share this project with proper attributi
