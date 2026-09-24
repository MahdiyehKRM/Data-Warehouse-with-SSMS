# Source Database Analysis — Northwind

**Project:** Northwind Data Warehouse  
**Layer:** Source (OLTP)  
**Owner:** Mahdieh Karimi  
**Role:** Data Warehouse Developer  
**Platform:** Microsoft SQL Server  
**Language:** T-SQL  
**Architecture:** Source → Stage → DDS (Star Schema) → Consume  
**Document Type:** Markdown (`.md`)  
**Last Updated:** 2026-09-24  
**Version:** 1.0.0  

---

## Table of Contents

1. Purpose
2. Overview
3. Source Database Structure
4. Main Business Entities
5. Detailed Table Analysis
6. Key Relationships
7. Entity Relationship Diagram (ERD)
8. Source Data Characteristics
9. Column-Level Analysis
10. Data Types and Constraints
11. Data Quality Observations
12. Tables Used vs. Out of Scope
13. Business Rules Derived from Source
14. Grain Analysis
15. Mapping to Data Warehouse
16. Source Database Diagram
17. Next Steps
18. Version History
19. Final Statement

---

## 1. Purpose

This document is the official Source Database Analysis for the Northwind Data Warehouse project.

The purpose of this analysis is to understand the structure, entities, relationships, and available data in the source system before designing the Data Warehouse.

This document serves as the foundation for:

- the Staging Layer design (`stage_dw.stage`)
- the DDS / Star Schema design (`dds.sale` + `dds.dbo`)
- the ETL pipeline and transformation logic
- the Data Catalog and Data Dictionary
- the Reporting Layer and KPI definitions

By documenting the source system in detail, we ensure traceability from source to target, consistency in naming and semantics, reliability in ETL design, and clarity for developers, analysts, and stakeholders.

---

## 2. Overview

The Northwind database is used as the source transactional (OLTP) database for this Business Intelligence project.

Northwind is a classic sample database representing a small trading company that:

- sells food products (beverages, condiments, dairy, seafood, etc.)
- manages customers, orders, and shipments
- employs sales representatives
- works with external suppliers

### Key Characteristics

| Characteristic | Value |
|---|---|
| Type | OLTP (Online Transaction Processing) |
| Platform | Microsoft SQL Server |
| Purpose | Sample database for learning and demos |
| Domain | Sales and Distribution |
| Schema | dbo (default) |
| Approximate Records | ~3,300 across all tables |
| Date Range | 1996 – 1998 (Gregorian) |

### Why Northwind?

Northwind is ideal for this project because it has a well-defined schema with clear relationships, contains realistic sales data (orders, products, customers), covers dimension and fact scenarios naturally, is widely used in BI and DW learning materials, and allows demonstration of advanced DW techniques.

---

## 3. Source Database Structure

The Northwind database contains several tables related to:

- Customers — customer master data
- Orders — order headers
- Order Details — order line items
- Products — product master data
- Categories — product categories
- Suppliers — supplier master data
- Employees — employee master data
- Shippers — shipping companies
- Territories — sales territories
- Region — sales regions
- Customer Demographics — customer demographic types

The database also contains several views and system-related objects. In this project, the main analysis focuses on the base tables that contain the operational data.

### 3.1 Table Categories

| Category | Tables |
|---|---|
| Master Data | Customers, Employees, Suppliers, Products, Categories, Shippers |
| Transactional Data | Orders, Order Details |
| Reference Data | Region, Territories, CustomerDemographics |
| Mapping Data | EmployeeTerritories, CustomerCustomerDemo |
| System Objects | Views, Stored Procedures, Functions |

### 3.2 Naming Convention

All source tables follow PascalCase naming:

| Naming Pattern | Example |
|---|---|
| Master Entities | Customers, Employees, Products |
| Transactional | Orders, Order Details (with space) |
| Mapping | EmployeeTerritories, CustomerCustomerDemo |
| Lookup | Region, Categories, Shippers |

Note: The table `Order Details` contains a space in its name — this requires bracket notation in T-SQL: `[Order Details]`.

---

## 4. Main Business Entities

The most important business entities identified in the source database are:

| Entity | Description | Business Role |
|---|---|---|
| Customers | Contains customer information | Who buys products |
| Orders | Contains order-level information | When and how orders are placed |
| Order Details | Contains individual product lines within each order | What is ordered |
| Products | Contains product information | What is sold |
| Categories | Contains product category information | Product classification |
| Suppliers | Contains supplier information | Who supplies products |
| Employees | Contains employee information | Who processes orders |
| Shippers | Contains shipping company information | How orders are shipped |
| Territories | Contains sales territory information | Where sales occur |
| Region | Contains regional information | Regional grouping |
| CustomerDemographics | Contains demographic descriptions | Customer segmentation |

### 4.1 Entity Classification

| Classification | Entities |
|---|---|
| Party (who) | Customers, Employees, Suppliers, Shippers |
| Product (what) | Products, Categories |
| Event (when) | Orders, Order Details |
| Location (where) | Region, Territories |
| Classification (how) | CustomerDemographics |

---

## 5. Detailed Table Analysis

### 5.1 Customers

Purpose: Stores customer master data.

Primary Key: `CustomerID` (NCHAR(5))

Business Key: `CustomerID`

Approximate Records: 91

Key Columns:

| Column | Type | Description |
|---|---|---|
| CustomerID | NCHAR(5) | Unique customer identifier |
| CompanyName | NVARCHAR(40) | Customer company name |
| ContactName | NVARCHAR(30) | Contact person |
| ContactTitle | NVARCHAR(30) | Contact title |
| Address | NVARCHAR(60) | Street address |
| City | NVARCHAR(15) | City |
| Region | NVARCHAR(15) | Region / state |
| PostalCode | NVARCHAR(10) | Postal code |
| Country | NVARCHAR(15) | Country |
| Phone | NVARCHAR(24) | Phone (PII) |
| Fax | NVARCHAR(24) | Fax (PII) |

Role in DW: Source for `dim_customer` and `dim_geography`.

---

### 5.2 Orders

Purpose: Stores order header information.

Primary Key: `OrderID` (INT)

Business Key: `OrderID`

Approximate Records: 830

Key Columns:

| Column | Type | Description |
|---|---|---|
| OrderID | INT | Unique order identifier |
| CustomerID | NCHAR(5) | FK to Customers |
| EmployeeID | INT | FK to Employees |
| OrderDate | DATETIME | Order date |
| RequiredDate | DATETIME | Required delivery date |
| ShippedDate | DATETIME | Actual ship date |
| ShipVia | INT | FK to Shippers |
| Freight | MONEY | Freight cost |
| ShipName | NVARCHAR(40) | Ship-to name |
| ShipAddress | NVARCHAR(60) | Ship-to address |
| ShipCity | NVARCHAR(15) | Ship-to city |
| ShipRegion | NVARCHAR(15) | Ship-to region |
| ShipPostalCode | NVARCHAR(10) | Ship-to postal code |
| ShipCountry | NVARCHAR(15) | Ship-to country |

Role in DW: Source for `fact_order` (order-level fields) and `dim_geography` (ship-to address).

---

### 5.3 Order Details

Purpose: Stores order line items (products within each order).

Composite Primary Key: `(OrderID, ProductID)`

Business Key: `(OrderID, ProductID)`

Approximate Records: 2,155

Key Columns:

| Column | Type | Description |
|---|---|---|
| OrderID | INT | FK to Orders |
| ProductID | INT | FK to Products |
| UnitPrice | MONEY | Price at order time |
| Quantity | SMALLINT | Quantity ordered |
| Discount | REAL | Discount rate (0–1) |

Role in DW: Source for `fact_order` (measures).

Note: This table defines the Fact Grain — one row per (OrderID, ProductID).

---

### 5.4 Products

Purpose: Stores product master data.

Primary Key: `ProductID` (INT)

Business Key: `ProductID`

Approximate Records: 77

Key Columns:

| Column | Type | Description |
|---|---|---|
| ProductID | INT | Unique product identifier |
| ProductName | NVARCHAR(40) | Product name |
| SupplierID | INT | FK to Suppliers |
| CategoryID | INT | FK to Categories |
| QuantityPerUnit | NVARCHAR(20) | Packaging description |
| UnitPrice | MONEY | Current unit price |
| UnitsInStock | SMALLINT | Units in stock |
| UnitsOnOrder | SMALLINT | Units on order |
| ReorderLevel | SMALLINT | Reorder threshold |
| Discontinued | BIT | Discontinued flag |

Role in DW: Source for `dim_product`.

---

### 5.5 Categories

Purpose: Stores product category information.

Primary Key: `CategoryID` (INT)

Business Key: `CategoryID`

Approximate Records: 8

Key Columns:

| Column | Type | Description |
|---|---|---|
| CategoryID | INT | Unique category identifier |
| CategoryName | NVARCHAR(15) | Category name |
| Description | NVARCHAR(MAX) | Category description |
| Picture | IMAGE | Category image |

Role in DW: Denormalized into `dim_product` (no separate `dim_category`).

---

### 5.6 Suppliers

Purpose: Stores supplier master data.

Primary Key: `SupplierID` (INT)

Business Key: `SupplierID`

Approximate Records: 29

Key Columns:

| Column | Type | Description |
|---|---|---|
| SupplierID | INT | Unique supplier identifier |
| CompanyName | NVARCHAR(40) | Supplier company name |
| ContactName | NVARCHAR(30) | Contact person |
| ContactTitle | NVARCHAR(30) | Contact title |
| Address | NVARCHAR(60) | Street address |
| City | NVARCHAR(15) | City |
| Region | NVARCHAR(15) | Region |
| PostalCode | NVARCHAR(10) | Postal code |
| Country | NVARCHAR(15) | Country |
| Phone | NVARCHAR(24) | Phone (PII) |
| Fax | NVARCHAR(24) | Fax (PII) |
| HomePage | NVARCHAR(MAX) | Website |

Role in DW: Source for `dim_supplier` and `dim_geography`.

---

### 5.7 Employees

Purpose: Stores employee master data.

Primary Key: `EmployeeID` (INT)

Business Key: `EmployeeID`

Approximate Records: 9

Key Columns:

| Column | Type | Description |
|---|---|---|
| EmployeeID | INT | Unique employee identifier |
| LastName | NVARCHAR(20) | Last name |
| FirstName | NVARCHAR(10) | First name |
| Title | NVARCHAR(30) | Job title |
| TitleOfCourtesy | NVARCHAR(25) | Courtesy title |
| BirthDate | DATETIME | Birth date |
| HireDate | DATETIME | Hire date |
| Address | NVARCHAR(60) | Street address |
| City | NVARCHAR(15) | City |
| Region | NVARCHAR(15) | Region |
| PostalCode | NVARCHAR(10) | Postal code |
| Country | NVARCHAR(15) | Country |
| HomePhone | NVARCHAR(24) | Home phone (PII) |
| Extension | NVARCHAR(4) | Extension |
| Photo | IMAGE | Employee photo |
| Notes | NVARCHAR(MAX) | Notes |
| ReportsTo | INT | Manager reference |
| PhotoPath | NVARCHAR(255) | Photo file path |

Role in DW: Source for `dim_employee` and `dim_geography`.

---

### 5.8 Shippers

Purpose: Stores shipping company information.

Primary Key: `ShipperID` (INT)

Business Key: `ShipperID`

Approximate Records: 3

Key Columns:

| Column | Type | Description |
|---|---|---|
| ShipperID | INT | Unique shipper identifier |
| CompanyName | NVARCHAR(40) | Shipper company name |
| Phone | NVARCHAR(24) | Phone |

Role in DW: Source for `dim_shipper`.

---

### 5.9 Territories (Out of Scope)

Purpose: Stores sales territory information.

Primary Key: `TerritoryID` (NVARCHAR(20))

Approximate Records: 53

Key Columns:

| Column | Type | Description |
|---|---|---|
| TerritoryID | NVARCHAR(20) | Territory identifier |
| TerritoryDescription | NCHAR(50) | Description |
| RegionID | INT | FK to Region |

Status: Not used in Sales Data Mart.

---

### 5.10 Region (Out of Scope)

Purpose: Stores regional information.

Primary Key: `RegionID` (INT)

Approximate Records: 4

Key Columns:

| Column | Type | Description |
|---|---|---|
| RegionID | INT | Region identifier |
| RegionDescription | NCHAR(50) | Description |

Status: Not used in Sales Data Mart.

---

### 5.11 EmployeeTerritories (Out of Scope)

Purpose: Maps employees to territories.

Composite Primary Key: `(EmployeeID, TerritoryID)`

Approximate Records: 49

Status: Not used in Sales Data Mart.

---

### 5.12 CustomerDemographics (Out of Scope)

Purpose: Stores customer demographic types.

Primary Key: `CustomerTypeID` (NCHAR(10))

Approximate Records: 0 (empty)

Status: Not used in Sales Data Mart.

---

### 5.13 CustomerCustomerDemo (Out of Scope)

Purpose: Maps customers to demographic types.

Composite Primary Key: `(CustomerID, CustomerTypeID)`

Approximate Records: 0 (empty)

Status: Not used in Sales Data Mart.

---

## 6. Key Relationships

### 6.1 Core Sales Flow

Customers → Orders → Order Details → Products

Orders also connects to Employees and Shippers.

Products connects to Categories and Suppliers.

### 6.2 Relationship Details

| Parent | Child | Type | FK Column |
|---|---|---|---|
| Customers | Orders | 1:N | CustomerID |
| Employees | Orders | 1:N | EmployeeID |
| Shippers | Orders | 1:N | ShipVia |
| Orders | Order Details | 1:N | OrderID |
| Products | Order Details | 1:N | ProductID |
| Categories | Products | 1:N | CategoryID |
| Suppliers | Products | 1:N | SupplierID |
| Employees | EmployeeTerritories | 1:N | EmployeeID |
| Territories | EmployeeTerritories | 1:N | TerritoryID |
| Region | Territories | 1:N | RegionID |
| Customers | CustomerCustomerDemo | 1:N | CustomerID |
| CustomerDemographics | CustomerCustomerDemo | 1:N | CustomerTypeID |

### 6.3 Cardinality Summary

| Relationship | Cardinality |
|---|---|
| One Customer → Many Orders | 1:N |
| One Order → Many Order Details | 1:N |
| One Product → Many Order Details | 1:N |
| One Category → Many Products | 1:N |
| One Supplier → Many Products | 1:N |
| One Employee → Many Orders | 1:N |
| One Shipper → Many Orders | 1:N |

---

## 7. Entity Relationship Diagram (ERD)

Region (1) ──── (N) Territories
Territories (1) ──── (N) EmployeeTerritories
Employees (1) ──── (N) EmployeeTerritories
Employees (1) ──── (N) Orders
Customers (1) ──── (N) Orders
Shippers (1) ──── (N) Orders
Orders (1) ──── (N) Order Details
Products (1) ──── (N) Order Details
Categories (1) ──── (N) Products
Suppliers (1) ──── (N) Products

---

## 8. Source Data Characteristics

### 8.1 Row Counts

| Table | Row Count | Category |
|---|---:|---|
| Categories | 8 | Master |
| Customers | 91 | Master |
| Employees | 9 | Master |
| EmployeeTerritories | 49 | Mapping |
| Order Details | 2,155 | Transactional |
| Orders | 830 | Transactional |
| Products | 77 | Master |
| Region | 4 | Reference |
| Shippers | 3 | Master |
| Suppliers | 29 | Master |
| Territories | 53 | Reference |
| CustomerDemographics | 0 | Reference |
| CustomerCustomerDemo | 0 | Mapping |

### 8.2 Data Volume Summary

| Category | Total Records |
|---|---:|
| Master Data | 217 |
| Transactional Data | 2,985 |
| Reference / Mapping | 106 |
| Empty Tables | 2 |
| Grand Total | ~3,308 |

### 8.3 Date Range

| Entity | Min Date | Max Date |
|---|---|---|
| Orders.OrderDate | 1996-07-04 | 1998-05-06 |
| Orders.RequiredDate | 1996-07-11 | 1998-06-03 |
| Orders.ShippedDate | 1996-07-08 | 1998-05-06 |

Note: All dates are Gregorian (Miladi) — critical for Date Dimension mapping.

---

## 9. Column-Level Analysis

### 9.1 Data Type Distribution

| Data Type | Usage | Example Columns |
|---|---|---|
| INT | Surrogate and FK keys | OrderID, ProductID |
| NCHAR(5) | Customer ID | CustomerID |
| NVARCHAR(n) | Names and descriptions | CompanyName, City |
| NVARCHAR(MAX) | Long text | Notes, Description |
| MONEY | Monetary values | UnitPrice, Freight |
| SMALLINT | Quantities | Quantity, UnitsInStock |
| REAL | Discount rate | Discount |
| BIT | Flags | Discontinued |
| DATETIME | Dates | OrderDate, HireDate |
| IMAGE | Binary images | Photo, Picture |

### 9.2 Nullable Columns

Most descriptive columns are nullable in the source:

- ContactName, ContactTitle (Customers, Suppliers)
- Region, PostalCode (addresses)
- Fax, HomePage
- ShippedDate (may be NULL for unshipped orders)

### 9.3 PII Columns

| Table | Column | Classification |
|---|---|---|
| Customers | ContactName | PII |
| Customers | Phone | PII |
| Customers | Fax | PII |
| Employees | FirstName | PII |
| Employees | LastName | PII |
| Employees | HomePhone | PII |
| Employees | BirthDate | PII |
| Suppliers | ContactName | PII |
| Suppliers | Phone | PII |
| Suppliers | Fax | PII |

---

## 10. Data Types and Constraints

### 10.1 Primary Keys

| Table | Primary Key |
|---|---|
| Customers | CustomerID |
| Orders | OrderID |
| Order Details | (OrderID, ProductID) |
| Products | ProductID |
| Categories | CategoryID |
| Suppliers | SupplierID |
| Employees | EmployeeID |
| Shippers | ShipperID |
| Territories | TerritoryID |
| Region | RegionID |
| EmployeeTerritories | (EmployeeID, TerritoryID) |
| CustomerDemographics | CustomerTypeID |
| CustomerCustomerDemo | (CustomerID, CustomerTypeID) |

### 10.2 Foreign Keys

| Child Table | FK Column | Parent Table |
|---|---|---|
| Orders | CustomerID | Customers |
| Orders | EmployeeID | Employees |
| Orders | ShipVia | Shippers |
| Order Details | OrderID | Orders |
| Order Details | ProductID | Products |
| Products | CategoryID | Categories |
| Products | SupplierID | Suppliers |
| EmployeeTerritories | EmployeeID | Employees |
| EmployeeTerritories | TerritoryID | Territories |
| Territories | RegionID | Region |
| CustomerCustomerDemo | CustomerID | Customers |
| CustomerCustomerDemo | CustomerTypeID | CustomerDemographics |
| Employees | ReportsTo | Employees (self) |

### 10.3 Unique Constraints

| Table | Unique Column |
|---|---|
| Customers | CustomerID |
| Products | ProductID |
| Categories | CategoryID |
| Suppliers | SupplierID |
| Employees | EmployeeID |
| Shippers | ShipperID |

---

## 11. Data Quality Observations

### 11.1 Known Issues in Source

| Issue | Table | Description |
|---|---|---|
| Empty strings | Customers, Suppliers | Some fields may have empty strings instead of NULL |
| Leading/trailing spaces | Text fields | Names may have extra spaces |
| NULL regions | Customers, Orders | Region may be NULL |
| NULL ship dates | Orders | ShippedDate NULL for unshipped orders |
| NULL discounts | Order Details | Discount may be 0 |
| Missing FKs | Orders | EmployeeID can be NULL |
| Duplicate prevention | Order Details | Composite PK prevents duplicates |
| Legacy data | Employees | Photo, Notes are legacy |

### 11.2 Data Quality Rules Derived

Based on observations, the following rules must be applied during ETL:

1. Trim all text fields (LTRIM, RTRIM)
2. Convert empty strings to NULL (NULLIF)
3. Replace NULLs with standard defaults (COALESCE)
4. Validate quantities (Quantity > 0)
5. Validate prices (UnitPrice >= 0)
6. Normalize discounts (0 <= Discount <= 1)
7. Standardize geography (UPPER)
8. Handle NULL FKs with Unknown Members (Key = 0)

---

## 12. Tables Used vs. Out of Scope

### 12.1 Tables Used in the Data Warehouse

| Source Table | Target Layer | Target Object |
|---|---|---|
| Customers | DDS | dim_customer + dim_geography |
| Employees | DDS | dim_employee + dim_geography |
| Suppliers | DDS | dim_supplier + dim_geography |
| Products | DDS | dim_product |
| Categories | DDS | dim_product (Denormalized) |
| Shippers | DDS | dim_shipper |
| Orders | DDS | fact_order + dim_geography |
| Order Details | DDS | fact_order |

### 12.2 Tables Out of Scope

| Source Table | Reason for Exclusion |
|---|---|
| Region | Territory data, not relevant to Sales |
| Territories | Territory assignments, not needed for Sales |
| EmployeeTerritories | Employee-territory mapping, not needed |
| CustomerCustomerDemo | Customer demographics mapping (empty, not Sales-related) |
| CustomerDemographics | Demographic descriptions (empty, not needed) |

### 12.3 Design Principle

Kimball Dimensional Modeling: Include only tables that serve the business questions of the Sales Data Mart.

---

## 13. Business Rules Derived from Source

| Rule | Description |
|---|---|
| Grain stability | Order Details defines the natural grain: (OrderID, ProductID) |
| One order → many lines | An order can contain multiple products |
| One product → many lines | A product can appear in many orders |
| Customer → many orders | A customer can place multiple orders |
| Employee → many orders | An employee can process multiple orders |
| Shipper → many orders | A shipper can handle multiple orders |
| Product → one category | Each product belongs to exactly one category |
| Product → one supplier | Each product comes from exactly one supplier |
| Discount range | Discount must be between 0 and 1 |
| Quantity positivity | Quantity must be > 0 |
| Price non-negativity | UnitPrice must be >= 0 |
| Date consistency | OrderDate <= RequiredDate |
| Ship date consistency | ShippedDate >= OrderDate (when not NULL) |

---

## 14. Grain Analysis

### 14.1 Grain of Source Tables

| Table | Grain |
|---|---|
| Customers | One row per customer |
| Orders | One row per order |
| Order Details | One row per order + product |
| Products | One row per product |
| Categories | One row per category |
| Suppliers | One row per supplier |
| Employees | One row per employee |
| Shippers | One row per shipper |

### 14.2 Grain of Target Fact

| Target | Grain |
|---|---|
| sale.fact_order | One row per Order ID + Product ID |

Source for Grain: Order Details (natural grain).

### 14.3 Why This Grain?

The Order Details table naturally defines the lowest level of detail. Each row represents one product in one order. This grain supports sales analysis by product, by order, discount analysis, quantity analysis, and freight allocation.

---

## 15. Mapping to Data Warehouse

### 15.1 Source → Stage Mapping

| Source Table | Stage Table |
|---|---|
| Categories | stage.northwind_categories |
| Customers | stage.northwind_customers |
| Employees | stage.northwind_employees |
| Suppliers | stage.northwind_suppliers |
| Products | stage.northwind_products |
| Shippers | stage.northwind_shippers |
| Orders | stage.northwind_orders |
| Order Details | stage.northwind_order_details |

### 15.2 Stage → DDS Mapping

| Stage Table | DDS Object |
|---|---|
| northwind_customers | sale.dim_customer + sale.dim_geography |
| northwind_employees | sale.dim_employee + sale.dim_geography |
| northwind_suppliers | sale.dim_supplier + sale.dim_geography |
| northwind_products | sale.dim_product |
| northwind_categories | sale.dim_product (Denormalized) |
| northwind_shippers | sale.dim_shipper |
| northwind_orders | sale.fact_order + sale.dim_geography |
| northwind_order_details | sale.fact_order |

### 15.3 Transformation Summary

| Transformation | Source | Target |
|---|---|---|
| Deduplication | All tables | All dims |
| Text cleaning | Text columns | All text columns |
| Denormalization | Products + Categories | dim_product |
| Integration | 4 sources | dim_geography |
| Derivation | QuantityPerUnit | package_quantity |
| Standardization | Discontinued | discontinued_status |
| Discount validation | Discount | discount_rate |
| Date mapping | OrderDate etc. | *_date_key |
| Measure derivation | UnitPrice, Quantity | gross_amount, etc. |
| Freight allocation | Freight | freight_amount |

---

## 16. Source Database Diagram

The source database structure is also documented using a Database Diagram created in SQL Server Management Studio (SSMS).

### 16.1 Diagram Location

screenshots/source_data_diagram.png

### 16.2 Diagram Contents

The diagram shows 13 tables in the Northwind database, primary keys for each table, foreign key relationships (1:N), column names and data types, and table groupings (Master, Transactional, Reference).

### 16.3 Related Diagrams

| Diagram | File | Purpose |
|---|---|---|
| Data Warehouse Architecture | data_warehouse_architecture.png | 4-layer architecture |
| Data Flow | data_flow.png | Source → Stage → DDS lineage |
| Star Schema | core_dw.png | Fact + Dimensions |
| Staging Load Execution | staging_load_execution.png | SP output |
| Staging Load Log | staging_load_log.png | ETL log |

---

## 17. Next Steps

After completing the source database analysis, the next stages are:

### 17.1 Immediate Next Steps

1. Define Data Catalog for relevant source tables
2. Design Staging Layer tables (stage_dw.stage)
3. Implement Stage ETL (stage.usp_load_stg_northwind_full)
4. Design DDS Layer (Star Schema)
5. Implement Dimension ETL (sale.usp_load_dim_northwind)
6. Implement Fact ETL (sale.usp_load_fact_order_full)
7. Build Reporting Views (rpt.vw_*)

### 17.2 Documentation Deliverables

| # | Document | Status |
|---|---|---|
| 1 | docs/SOURCE_ANALYSIS.md | This document |
| 2 | docs/DATA_CATALOG.md | Complete |
| 3 | docs/ARCHITECTURE.md | Pending |
| 4 | docs/DATA_DICTIONARY.md | Pending |
| 5 | README.md | Pending |

### 17.3 Related Documents

- Data Catalog: ./DATA_CATALOG.md
- Architecture: ./ARCHITECTURE.md (optional)
- Data Dictionary: ./DATA_DICTIONARY.md (optional)
- README: ../README.md

---

## 18. Version History

| Version | Date | Author | Notes |
|---|---|---|---|
| 1.0.0 | 2026-09-24 | Mahdieh Karimi | Initial Source Database Analysis for Northwind Data Warehouse |

---

## 19. Final Statement

This Source Database Analysis defines the structural and business understanding of the Northwind OLTP database as the source system for the Northwind Data Warehouse project.

It serves as the foundation for the Staging Layer design, the DDS / Star Schema design, the ETL pipeline, the Data Catalog and Data Dictionary, and the Reporting Layer.

Any modification to the source system, ETL pipeline, or warehouse schema must be reflected in this document to preserve consistency, reliability, and traceability.

---

**Maintained by:** Mahdieh Karimi  
**Format:** Markdown (.md)  
**Repository Use:** GitHub Documentation