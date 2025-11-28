
# SQL Server Data Warehouse Project

This project showcases the complete design and implementation of a full **SQL Server–based Data Warehouse** built entirely from raw text files originating from two separate operational systems: **ERP** and **CRM**. The goal of this project was not just to load and clean data, but to demonstrate how a disciplined and well‑structured warehouse architecture, supported by proper modelling, transformation logic, and integration workflows, can combine fragmented operational data into a unified analytical environment.

The warehouse follows a modern **medallion-style design** and includes detailed stages of ingestion, transformation, integration, and modelling. It also features a clean business-oriented star schema, making it suitable for analytics, reporting, and BI tools.

---
### Credits
This project is inspired by the work of **Baraa Salkni**, whose structure and approach helped shape the design and direction of this data warehouse project.
## Data Architecture  
<img width="3064" height="2360" alt="image" src="https://github.com/user-attachments/assets/27f7ec98-c3b9-436b-9349-bb60807c833d" />


At the highest level, the architecture defines how data moves from raw sources through structured layers. ERP and CRM files are ingested independently, processed through sequential transformation steps, and reshaped into a consolidated analytics-ready model. Each layer represents a higher level of structure and refinement, allowing the data to evolve gradually instead of being heavily transformed in a single step. This approach ensures transparency, troubleshooting ease, and future scalability.

---

## Data Integration Design  
<img width="5652" height="2004" alt="image" src="https://github.com/user-attachments/assets/393bdb74-8823-49ac-84c6-0921c15c2fec" />


The data integration strategy revolves around bringing together two systems that do not share a consistent structure. The ERP system maintains product, customer, and location information from an operational viewpoint. The CRM system captures customer attributes and sales details from the business-facing viewpoint. In this project, I designed a set of integration rules inside SQL Server to align identifiers, reconcile differences in naming and formatting, standardise product and customer references, and ensure that once loaded into Silver and Gold, records from both systems could be matched reliably.

This required:
- mapping ERP and CRM customer records,
- normalising product keys between systems,
- applying consistent date formats,
- recalculating inconsistent sales values,
- validating location and region fields,
- and ensuring referential integrity across sources.

---

## Data Flow  
<img width="2444" height="1484" alt="image" src="https://github.com/user-attachments/assets/94db5386-d426-440f-acfd-f1b36995cc46" />


The data flow demonstrates the exact journey of data from raw files to clean business-ready models. It begins with ingestion from two folders, followed by Bronze loading, then a series of Silver-layer transformations, and finally the construction of a Gold-layer star schema. At each stage, data quality increases due to incremental refinement—allowing issues to be addressed in manageable steps while maintaining full reproducibility.

---

## 🥉 Bronze Layer — Raw Ingestion

In this first layer, I imported all ERP and CRM text files exactly as they exist. The purpose of Bronze is to preserve the original structure, enabling traceability and auditability. Rather than fixing errors immediately, this layer creates a faithful mirror of the raw data.

In Bronze, I:
- loaded all text files into SQL Server using raw datatypes,
- ensured row counts matched the source files,
- preserved all messy fields without alteration,
- added fundamental structure so the files could be queried consistently.

This stage allowed downstream layers to build reliably on top of a stable foundation.

---

## 🥈 Silver Layer — Cleaning, Standardisation, Alignment

The Silver layer contains the most meaningful transformation logic. This is the stage where the ERP and CRM datasets begin to converge into a unified format. The goal is to clean inconsistencies, handle invalid values, normalise fields, integrate lookup data, and generate a refined dataset suitable for modelling.

In this layer, I:
- corrected invalid date strings and cast them into proper SQL date formats,
- standardised numeric fields, corrected decimal issues, and recalculated incorrect sales values,
- cleaned demographic and customer attributes (gender, country, birthdate),
- removed duplicate records and filtered invalid rows,
- reconciled mismatched product identifiers between ERP and CRM,
- merged CRM and ERP customer information into unified structures,
- aligned product category and product metadata fields,
- enforced referential integrity across the cleaned tables.

By the end of Silver, all major inconsistencies were resolved, and the dataset represented a cleaned, structured, and joined version of both systems.

---

## 🥇 Gold Layer — Business Data Model

<img width="5400" height="1988" alt="image" src="https://github.com/user-attachments/assets/4178ea51-8dad-477d-873f-0d71c9529bdd" />


The Gold layer contains the final business-ready analytical model. Here, I designed a star schema that centralises sales records and enriches them through descriptive dimensions. The model was constructed following Kimball dimensional modelling principles and is optimised for BI tools, dashboards, and ad‑hoc analysis.

In this stage, I:
- built a unified `fact_sales` table combining ERP and CRM fields,
- generated `dim_customers` with cleaned demographics and unified customer identifiers,
- built `dim_products` with merged ERP–CRM product references,
- incorporated business logic fields such as product line names, category groupings, and price validations,
- ensured the star schema supports fast aggregation and intuitive navigation for analysts.

The Gold model represents a single source of truth that integrates the entire ERP and CRM ecosystem into one coherent analytical layer.

---

## Final Summary

This project demonstrates a complete data engineering lifecycle implemented entirely in SQL Server. Starting from disconnected operational text files, I built a multi-layered warehouse that progressively enhances data quality, integrates multiple systems, and results in a polished dimensional model suitable for reporting. The approach emphasises clarity, strong modelling foundations, and realistic enterprise-grade design. It reflects experience not only in SQL coding, but also in architectural thinking, data quality stewardship, and analytical readiness.
