# **Naming Conventions**
This document outlines the naming conventions used for the tables, columns, and other objects in this project.

## **General Principles**
- **Naming convention**: Use snake_case, with lowercase letters and underscores (`_`) to separate words.
- **Language**: Use English for all names.
- **Query groups in Power Query**: Capitalize the names so they are visually distinct from the query names inside them.

## **Table Naming Conventions**
### **Rules for Raw group**
- All table names must start with `raw`, and table names must match their original entity names without renaming.
- `raw_<entity>`
  - `<entity>`: Exact entity name (e.g., orders, order_items).
  - Examples: `raw_orders`

### **Rules for Transformed group**
- All table names must start with `stg`, and table names must match their original entity names without renaming.
- `stg_<entity>`  
  - `<entity>`: Exact entity name (e.g., orders, order_items).
  - Example: `stg_orders`

### **Rules for Model group**
- All table names must use meaningful, business-aligned names for tables, starting with the category prefix.
- **`<category>_<entity>`**  
  - `<category>`: Describes the role of a table – `dim` (dimension) or `fact` (fact table).  
  - `<entity>`: Descriptive name of a table, aligned with the business domain (e.g., `customers`, `products`).
  - Examples:
    - `dim_customers` → Dimension table for customer data.
    - `fact_orders` → Fact table containing order transactions.

#### **Category Prefixes**

| Prefix  | Meaning         | Examples                          |
| ------- | --------------- | --------------------------------- |
| `dim_`  | Dimension table | `dim_customers`, `dim_products`   |
| `fact_` | Fact table      | `fact_orders`, `fact_order_items` |

## **Column Naming Conventions**

| Layer       | Convention                                                                                                                                                                       | Example                                               |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Raw         | Keep original as-is                                                                                                                                                              | `order_purchase_timestamp`                            |
| Transformed | snake_case                                                                                                                                                                       | `order_purchase_timestamp`                            |
| Model       | Title Case with spaces; needs to be readable for business users since the columns will surface directly in the Power BI field list and appear in visual tooltips and axis labels | `Order Purchase Date`, `Customer ID`, `Freight Value` |

## **Measure Naming Conventions**
- All measures must use plain, readable, and descriptive names with in proper case since these will be directly displayed in the visuals.
	- Examples: `Total Revenue`, `Total Orders`
- All measures must be kept in a dedicated measures table, which must be named `_Measures`, with an underscore prefix so that it will automatically sort to the top of the field list.