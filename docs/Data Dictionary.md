# Data Dictionary

## `gold.dim_customers`

**Type:** Dimension  
**Grain:** 1 row per **unique customer**  

### Business Purpose

`dim_customers` provides a single, conformed view of customers by combining CRM customer info with ERP customer attributes and location data.  
It is used to slice and segment metrics such as sales by customer demographics, geography, and lifecycle.

### Column Definitions

| Column Name      | Data Type   | Nullable | Description                                                                                   |
|------------------|------------|----------|-----------------------------------------------------------------------------------------------|
| `customer_key`   | INT        | NO       | Surrogate key generated via `ROW_NUMBER()`. Primary key of the dimension.                     |
| `customer_id`    | INT        | YES      | Natural customer ID from `silver.crm_cust_info.cst_id`.                                       |
| `customer_number`| NVARCHAR(50)| YES     | Business/customer number from `silver.crm_cust_info.cst_key`.                                 |
| `first_name`     | NVARCHAR(50)| YES     | Customer first name from `silver.crm_cust_info.cst_firstname`.                                |
| `last_name`      | NVARCHAR(50)| YES     | Customer last name from `silver.crm_cust_info.cst_lastname`.                                  |
| `country`        | NVARCHAR(50)| YES     | Country from ERP location table `silver.erp_loc_a101.cntry`.                                  |
| `marital_status` | NVARCHAR(50)| YES     | Marital status from `silver.crm_cust_info.cst_marital_status`.                                |
| `gender`         | NVARCHAR(50)| YES     | Gender. Uses CRM gender if not `'n/a'`, otherwise falls back to ERP gender `silver.erp_cust_az12.gen`. |
| `birthdate`      | DATE       | YES      | Date of birth from `silver.erp_cust_az12.bdate`.                                              |
| `create_date`    | DATE       | YES      | Customer creation date from `silver.crm_cust_info.cst_create_date`.                           |

### Upstream Sources

- `silver.crm_cust_info`
- `silver.erp_cust_az12`
- `silver.erp_loc_a101`

---

## `gold.dim_products`

**Type:** Dimension  
**Grain:** 1 row per **active product** (where `prd_end_dt IS NULL`)  

### Business Purpose

`dim_products` standardises product information and enriches it with product category and maintenance attributes from the ERP system.  
It is used to slice and analyze measures such as sales by product, category, subcategory, and lifecycle dates.

### Column Definitions

| Column Name    | Data Type    | Nullable | Description                                                                 |
|----------------|-------------|----------|-----------------------------------------------------------------------------|
| `product_key`  | INT         | NO       | Surrogate key generated via `ROW_NUMBER()`. Primary key of the dimension.   |
| `product_id`   | INT         | YES      | Natural product ID from `silver.crm_prd_info.prd_id`.                        |
| `product_number`| NVARCHAR(50)| YES     | Product key/number from `silver.crm_prd_info.prd_key`.                       |
| `product_name` | NVARCHAR(50)| YES      | Product name from `silver.crm_prd_info.prd_nm`.                              |
| `category_id`  | NVARCHAR(50)| YES      | Category ID from `silver.crm_prd_info.cat_id`.                               |
| `category`     | NVARCHAR(50)| YES      | Category description from `silver.erp_px_cat_g1v2.cat`.                      |
| `subcategory`  | NVARCHAR(50)| YES      | Subcategory from `silver.erp_px_cat_g1v2.subcat`.                            |
| `maintenance`  | NVARCHAR(50)| YES      | Maintenance flag/type from `silver.erp_px_cat_g1v2.maintenance`.             |
| `cost`         | INT         | YES      | Product cost from `silver.crm_prd_info.prd_cost`.                            |
| `product_line` | NVARCHAR(50)| YES      | Product line from `silver.crm_prd_info.prd_line`.                            |
| `start_date`   | DATE        | YES      | Product start date from `silver.crm_prd_info.prd_start_dt`.                  |
| `end_date`     | DATE        | YES      | Product end date from `silver.crm_prd_info.prd_end_dt`. Typically `NULL` for active products. |

### Upstream Sources

- `silver.crm_prd_info`
- `silver.erp_px_cat_g1v2`

### Filters / Business Rules

- Only includes products where `prd_end_dt IS NULL` (active products).

---

## `gold.fact_sales`

**Type:** Fact  
**Grain:** 1 row per **sales order line**  

### Business Purpose

`fact_sales` contains transactional sales data linked to product and customer dimensions.  
It is the primary fact table for reporting metrics such as sales amount, quantity, and price by customer, product, and time.

### Column Definitions

| Column Name    | Data Type    | Nullable | Description                                                                                              |
|----------------|-------------|----------|----------------------------------------------------------------------------------------------------------|
| `order_number` | NVARCHAR(50)| YES      | Order number from `silver.crm_sales_details.sls_ord_num`.                                                |
| `product_key`  | INT         | YES      | Surrogate product key from `gold.dim_products.product_key`. Foreign key to the product dimension.        |
| `customer_key` | INT         | YES      | Surrogate customer key from `gold.dim_customers.customer_key`. Foreign key to the customer dimension.    |
| `customer_id`  | INT         | YES      | Natural customer ID from `silver.crm_sales_details.sls_cust_id`.                                         |
| `order_date`   | DATE        | YES      | Order date from `silver.crm_sales_details.sls_order_dt`.                                                 |
| `shipping_date`| DATE        | YES      | Shipping date from `silver.crm_sales_details.sls_ship_dt`.                                               |
| `due_date`     | DATE        | YES      | Due date from `silver.crm_sales_details.sls_due_dt`.                                                     |
| `sales_amount` | INT         | YES      | Sales amount from `silver.crm_sales_details.sls_sales`.                                                  |
| `quantity`     | INT         | YES      | Quantity sold from `silver.crm_sales_details.sls_quantity`.                                              |
| `price`        | INT         | YES      | Unit price from `silver.crm_sales_details.sls_price`.                                                    |

### Upstream Sources

- `silver.crm_sales_details`
- `gold.dim_products`
- `gold.dim_customers`

### Join Relationships

- `fact_sales.product_key` → `dim_products.product_key`
- `fact_sales.customer_key` → `dim_customers.customer_key`
- Transactional link: `sls_prd_key` (silver) → `dim_products.product_number`
- Transactional link: `sls_cust_id` (silver) → `dim_customers.customer_id`

---

## Notes & Conventions

- **Surrogate keys** (`customer_key`, `product_key`) are generated using `ROW_NUMBER()` and should be used in star-schema joins from facts to dimensions.
- **Natural keys** (`customer_id`, `product_id`, `customer_number`, `product_number`) are retained for traceability and reconciliation to source systems.
- **Nullability** is inferred from current DDL; business rules may further constrain fields in ETL logic.
- All gold views depend on the **silver layer** being fully loaded and consistent before refresh.

