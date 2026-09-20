# Staging Layer Design

## 1. Overview

The Staging Layer is the intermediate layer between the Northwind source database and the Data Warehouse.

Its main purpose is to receive and temporarily store data extracted from the source system before the data is processed and loaded into the final Data Warehouse.

In this project, the Staging Layer is implemented in a separate SQL Server database named:

`stage_dw`

A dedicated schema named `stage` is used to organize all staging-related tables and logging objects.

---

## 2. Staging Database Structure

The Staging database contains the following schema:

```text
stage_dw
└── stage
```

The `stage` schema contains the staging tables and the ETL execution log.

### Staging Tables

The following Northwind source tables were selected for the Staging Layer:

| Source Table  | Staging Table                   |
| ------------- | ------------------------------- |
| Categories    | `stage.northwind_categories`    |
| Customers     | `stage.northwind_customers`     |
| Employees     | `stage.northwind_employees`     |
| Suppliers     | `stage.northwind_suppliers`     |
| Products      | `stage.northwind_products`      |
| Shippers      | `stage.northwind_shippers`      |
| Orders        | `stage.northwind_orders`        |
| Order Details | `stage.northwind_order_details` |

Not all source tables were loaded into the Staging Layer. Only the entities required for the intended Data Warehouse and sales analysis were selected.

---

## 3. Naming Convention

The Staging Layer follows a `snake_case` naming convention.

For example:

```text
northwind_categories
northwind_customers
northwind_order_details
```

This provides a consistent naming structure and makes the staging layer easier to understand and maintain.

The original source table names are preserved conceptually, while spaces and naming inconsistencies are removed in the staging layer.

For example:

```text
Order Details
        ↓
northwind_order_details
```

---

## 4. Staging Table Design

The staging tables are designed to remain relatively close to the source structure.

The purpose of this layer is not to perform the final dimensional modeling. Instead, it provides a controlled intermediate area where source data can be loaded before further transformation and Data Warehouse processing.

Each staging table also contains the following technical column:

```text
dwh_inserted_at
```

This column stores the UTC timestamp at which the data was loaded into the Staging Layer.

The value is generated using:

```sql
SYSUTCDATETIME()
```

---

## 5. Loading Strategy

The Staging Layer uses a **Full Load** strategy.

The loading process follows:

```text
Northwind Source
       ↓
TRUNCATE Staging Table
       ↓
INSERT Source Data
       ↓
Staging Table
```

For each table, the existing staging data is first removed using:

```sql
TRUNCATE TABLE
```

The current source data is then inserted into the staging table.

This approach ensures that each execution creates a fresh copy of the selected source data.

---

## 6. Data Loading Process

The staging load process is implemented using a stored procedure:

```text
stage.usp_load_stg_northwind_full
```

The procedure performs the full load for the selected Northwind tables.

The procedure can be executed using:

```sql
USE stage_dw;
GO

EXEC stage.usp_load_stg_northwind_full;
```

Instead of manually executing separate `TRUNCATE` and `INSERT` statements for every table, the stored procedure provides a single entry point for the complete staging load process.

---

## 7. Error Handling and Logging

The staging load procedure uses `TRY...CATCH` blocks to handle errors during the loading of each table.

The execution information is stored in:

```text
stage.northwind_etrun_log
```

The log table contains information such as:

* `log_id`
* `package_name`
* `table_name`
* `rows_inserted`
* `status`
* `error_message`
* `start_time`
* `end_time`
* `dwh_inserted_at`

This allows the execution of the staging process to be monitored and audited.

For successful executions, the status is recorded as:

```text
SUCCESS
```

If an error occurs, the error message is recorded in:

```text
error_message
```

---

## 8. Staging Load Results

The Full Load process was successfully executed against the Northwind source database.

The following results were recorded:

| Staging Table             | Rows Inserted | Status  |
| ------------------------- | ------------: | ------- |
| `northwind_categories`    |             8 | SUCCESS |
| `northwind_customers`     |            91 | SUCCESS |
| `northwind_employees`     |             9 | SUCCESS |
| `northwind_suppliers`     |            29 | SUCCESS |
| `northwind_products`      |            77 | SUCCESS |
| `northwind_shippers`      |             3 | SUCCESS |
| `northwind_orders`        |           830 | SUCCESS |
| `northwind_order_details` |         2,155 | SUCCESS |

All eight staging tables were loaded successfully.

No error message was recorded for any of the loaded tables.

---

## 9. Source-to-Staging Flow

The overall data flow for the Staging Layer is:

```text
                    Northwind Source Database
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
         Categories       Customers         Employees
             │                │                │
             ▼                ▼                ▼
      stg_categories    stg_customers    stg_employees

             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
         Suppliers         Products         Shippers
             │                │                │
             ▼                ▼                ▼
      stg_suppliers     stg_products     stg_shippers

                       ┌──────────────┐
                       │    Orders    │
                       └──────┬───────┘
                              │
                              ▼
                        stg_orders

                       ┌──────────────┐
                       │ Order Details│
                       └──────┬───────┘
                              │
                              ▼
                    stg_order_details
```

The loaded staging data will be used as the input for the subsequent Data Warehouse transformation and loading process.

---

## 10. Role of the Staging Layer in the Project

The overall architecture currently follows:

```text
Northwind Source
       │
       ▼
   Staging Layer
       │
       ▼
Data Quality / Transformation
       │
       ▼
Data Warehouse
       │
       ▼
Fact & Dimension Tables
       │
       ▼
Analytical Queries
```

The Staging Layer therefore acts as the controlled intermediate layer between the operational source database and the final analytical Data Warehouse.

---

## 11. Conclusion

The Staging Layer for the Northwind project has been successfully created and populated.

The implementation includes:

* A separate `stage_dw` database
* A dedicated `stage` schema
* Eight selected staging tables
* `snake_case` naming convention
* UTC ingestion timestamps
* Full Load / Truncate-and-Load strategy
* Automated loading through a Stored Procedure
* TRY...CATCH error handling
* ETL execution logging
* Row-count tracking
* Successful loading of all selected source data

The next stage of the project is to design and implement the Data Warehouse model, including the appropriate Fact and Dimension tables.
