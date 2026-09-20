# Source Database Analysis

## 1. Overview

The Northwind database is used as the source transactional database for this Business Intelligence project.

The purpose of this analysis is to understand the structure, entities, relationships, and available data in the source system before designing the Data Warehouse.

## 2. Source Database Structure

The Northwind database contains several tables related to:

* Customers
* Orders
* Order Details
* Products
* Categories
* Suppliers
* Employees
* Shippers
* Territories
* Region
* Customer Demographics

The database also contains several views and system-related objects. In this project, the main analysis focuses on the base tables that contain the operational data.

## 3. Main Business Entities

The most important business entities identified in the source database are:

| Entity        | Description                                         |
| ------------- | --------------------------------------------------- |
| Customers     | Contains customer information                       |
| Orders        | Contains order-level information                    |
| Order Details | Contains individual product lines within each order |
| Products      | Contains product information                        |
| Categories    | Contains product category information               |
| Suppliers     | Contains supplier information                       |
| Employees     | Contains employee information                       |
| Shippers      | Contains shipping company information               |
| Territories   | Contains sales territory information                |
| Region        | Contains regional information                       |

## 4. Key Relationships

The main relationships identified in the source database include:

* Customers → Orders
* Orders → Order Details
* Order Details → Products
* Products → Categories
* Products → Suppliers
* Orders → Employees
* Orders → Shippers
* Employees → Employee Territories
* Employee Territories → Territories
* Territories → Region

These relationships were identified using the foreign key definitions of the Northwind source database.

## 5. Source Data Characteristics

The main source tables contain the following approximate number of records:

| Table               | Row Count |
| ------------------- | --------: |
| Categories          |         8 |
| Customers           |        91 |
| Employees           |         9 |
| EmployeeTerritories |        49 |
| Order Details       |     2,155 |
| Orders              |       830 |
| Products            |        77 |
| Region              |         4 |
| Shippers            |         3 |
| Suppliers           |        29 |
| Territories         |        53 |

Some source tables, such as `CustomerCustomerDemo` and `CustomerDemographics`, currently contain no records.

## 6. Important Observations

The `Orders` table represents order-level information, while `Order Details` contains the individual products included in each order.

Therefore, an order can contain multiple order-detail records.

The `Products` table is connected to both `Categories` and `Suppliers`, while orders are connected to customers, employees, and shippers.

These relationships provide the foundation for the later design of the staging layer and Data Warehouse.

## 7. Source Database Diagram

The source database structure is also documented using a Database Diagram created in SQL Server Management Studio.

The diagram is stored in the project's `screenshots` directory.

## 8. Next Step

After completing the source database analysis, the next stage is to define a detailed Data Catalog for the relevant source tables before implementing the staging layer.
