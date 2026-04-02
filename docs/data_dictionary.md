# Data Dictionary: Semantic Model Layer
This document covers all tables and columns in the semantic model layer of the project. The semantic model is built on top of the staging layer (in Power Query) and serves as the business-ready layer consumed by the report pages in Power BI.

**Model group tables covered:**
1. `fact_orders`
2. `fact_order_items`
3. `dim_customers`
4. `dim_sellers`
5. `dim_products`
6. `dim_order_payments`
7. `dim_order_reviews`
8. `dim_customer_segments`
9. `dim_seller_segments`
10. `dim_date`

**Naming conventions:** Tables follow a `fact_` / `dim_` prefix convention. Column names use Title Case with spaces. Date and datetime columns include the word "Date" in their name.

**Data types used:**

|Type|Description|
|---|---|
|`string`|Text value|
|`int64`|Whole number|
|`double`|Decimal number|
|`dateTime`|Date and time value|
|`date`|Date value only (no time component)|

---
## Fact Tables
Fact tables store measurable, transactional events. Each row represents a business event at a 
specific level of granularity. Fact tables are linked to dimension tables via foreign keys.

### `fact_orders`
**Description:** Each row represents one customer order. Although this fact table doesn't contain any measures, it serves as a bridge fact table for the data model and a reference for order-level analysis. It contains order lifecycle dates, status information, and attributes of when and how the order was placed.

**Granularity:** One row per order.

**Source:** `stg_orders`

| Column                  | Data Type | Key                  | Description                                                                                                                                                                                                                |
| ----------------------- | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Order ID                | string    | PK                   | Unique identifier for each order. Maps to Order ID in `fact_order_items`, `dim_order_payments`, and `dim_order_reviews`.                                                                                                   |
| Customer ID             | string    | FK → `dim_customers` | Identifies the customer associated with this order. Each order has exactly one Customer ID, but a customer may have multiple orders.                                                                                       |
| Order Status            | string    |                      | Current status of the order. Values include: `delivered`, `shipped`, `canceled`, `invoiced`, `processing`, `unavailable`, `approved`, `created`. All revenue and delivery analysis is filtered to `delivered` orders only. |
| Order Date              | date      |                      | The date the order was placed by the customer. Used as the primary date key for linking to `dim_date`.                                                                                                                     |
| Approval Date           | dateTime  |                      | Timestamp when the order was approved by Olist. `NULL` for 160 orders, mostly canceled.                                                                                                                                    |
| Carrier Delivery Date   | dateTime  |                      | Timestamp when the carrier received the order from the seller. Also referred to as fulfillment date. Used to compute average fulfillment days.                                                                             |
| Customer Delivery Date  | dateTime  |                      | Timestamp when the order was actually delivered to the customer. `NULL` for non-delivered orders. Used to compute average delivery days.                                                                                   |
| Estimated Delivery Date | dateTime  |                      | Olist's estimated delivery date communicated to the customer at time of order. Used to determine on-time vs. late delivery status.                                                                                         |
| Delivery Status         | string    |                      | Derived field indicating whether the order was delivered on time or late, based on comparison of `Customer Delivery Date` vs. `Estimated Delivery Date`. Values: `On Time`, `Late`.                                        |
| Order Hour of Day       | int64     |                      | The hour of the day (0–23) when the order was placed, extracted from the original order purchase timestamp.                                                                                                                |
| Order Day Type          | string    |                      | Indicates whether the order was placed on a `Weekday` or `Weekend`.                                                                                                                                                        |

### `fact_order_items`
**Description:** This is the main fact table that contains the measures. Each row represents one line item within an order. Since a single order can contain multiple products from multiple sellers, this table has a many-to-one relationship with `fact_orders`. It stores the financial details of each line item including product price, freight cost, and computed GMV.

**Granularity:** One row per order item (one order may have multiple rows).

**Source:** `stg_order_items`

| Column                  | Data Type | Key                 | Description                                                                                                                                                              |
| ----------------------- | --------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Order ID                | string    | FK → `fact_orders`  | Links the line item to its parent order. Not unique in this table since one order can contain multiple items.                                                            |
| Order Item ID           | int64     |                     | Sequential identifier for items within the same order. Starts at 1 for the first item in each order. Not globally unique.                                                |
| Product ID              | string    | FK → `dim_products` | Identifies the product purchased. Links to the product category information in `dim_products`.                                                                           |
| Seller ID               | string    | FK → `dim_sellers`  | Identifies the seller who fulfilled this line item. Links to seller location and segment data.                                                                           |
| Shipping Deadline       | dateTime  |                     | The latest date by which the seller must ship the item to meet the estimated delivery date.                                                                              |
| Price                   | double    |                     | The product price for this line item in Brazilian Reais (R$). Excludes freight charges. Used as the basis for GMV calculations in most measures.                         |
| Freight Value           | double    |                     | The shipping cost for this line item in Brazilian Reais (R$). Zero values represent free shipping.                                                                       |
| Gross Merchandise Value | double    |                     | The total transaction value for this line item, calculated as `Price + Freight Value`. Represents the full amount paid by the customer for this item including shipping. |

---
## Dimension Tables
Dimension tables store descriptive attributes used to slice, filter, and group data in a fact table. Each dimension table has a primary key that links to a foreign key in one or more fact tables.

### `dim_customers`
**Description:** Stores the geographic and identity attributes of customers. In the Olist dataset, a customer may place multiple orders but each order is associated with a separate `Customer ID`. The `Customer Unique ID` field is used to identify the actual unique individual across multiple orders.

**Granularity:** One row per Customer ID (one row per order-customer pair, not per unique person).

**Source:** `stg_customers`

| Column             | Data Type | Key | Description                                                                                                                                               |
| ------------------ | --------- | --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Customer ID        | string    | PK  | Order-level customer identifier. Each order has a unique Customer ID even if the same person placed multiple orders. Links to `fact_orders[Customer ID]`. |
| Customer Unique ID | string    |     | The actual unique customer identifier that remains consistent across multiple orders from the same person. Used for repeat vs. one-time buyer analysis.   |
| Customer ZIP Code  | string    |     | Five-digit ZIP code prefix of the customer's delivery address.                                                                                            |
| Customer City      | string    |     | City of the customer's delivery address.                                                                                                                  |
| Customer State     | string    |     | Two-letter Brazilian state code of the customer's delivery address (e.g., SP, RJ, MG).                                                                    |

### `dim_sellers`
**Description:** Stores the geographic attributes of sellers registered on the Olist platform. Each seller is uniquely identified by a Seller ID and is associated with a physical location used for logistics and regional analysis.

**Granularity:** One row per seller.

**Source:** `stg_sellers`

| Column          | Data Type | Key | Description                                                                             |
| --------------- | --------- | --- | --------------------------------------------------------------------------------------- |
| Seller ID       | string    | PK  | Unique identifier for each seller. Links to `fact_order_items[Seller ID]`.              |
| Seller ZIP Code | string    |     | Five-digit ZIP code prefix of the seller's registered location.                         |
| Seller City     | string    |     | City of the seller's registered location.                                               |
| Seller State    | string    |     | Two-letter Brazilian state code of the seller's registered location (e.g., SP, RJ, MG). |

### dim_products
**Description:** Stores product-level attributes. The dataset contains 32,951 unique products. Physical attributes (weight, dimensions, description length) were excluded from this table during modeling as they were not used in the analysis. Products with no category in the source data were assigned the label `Uncategorized`.

**Granularity:** One row per product.

**Source:** `stg_products`

| Column           | Data Type | Key | Description                                                                                                                                                                         |
| ---------------- | --------- | --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Product ID       | string    | PK  | Unique identifier for each product. Links to `fact_order_items[Product ID]`.                                                                                                        |
| Product Category | string    |     | English translation of the product category name. 610 products with no category in the source data are labeled `Uncategorized`. Used for all category-level analysis in the report. |

### dim_order_payments
**Description:** Stores payment details for each order. An order can be paid using multiple payment methods or across multiple installments, resulting in multiple rows per order. This table has a many-to-one relationship with `fact_orders`.

**Granularity:** One row per payment method per order (multiple rows possible per order).

**Source:** `stg_order_payments`

| Column            | Data Type | Key                | Description                                                                                                               |
| ----------------- | --------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| Order ID          | string    | FK → `fact_orders` | Links the payment record to its parent order. Not unique in this table since one order may have multiple payment records. |
| Payment Sequence  | int64     |                    | Sequential number identifying each payment method used within an order. Starts at 1.                                      |
| Payment Type      | string    |                    | The payment method used. Values include: `credit_card`, `boleto`, `voucher`, `debit_card`, `not_defined`.                 |
| Installment Count | int64     |                    | The number of installments selected by the customer for this payment. Relevant for credit card payments.                  |
| Payment Value     | double    |                    | The amount paid via this payment method in Brazilian Reais (R$).                                                          |

### dim_order_reviews

**Description:** Stores customer review scores for delivered orders. Reviews were collected by Olist after delivery. Not all delivered orders have a corresponding review. The dataset contains duplicate review IDs, likely due to the same review being associated with multiple orders or recording errors.

**Granularity:** One row per review.

**Source:** `stg_order_reviews`

| Column       | Data Type | Key                | Description                                                                                                                                                                              |
| ------------ | --------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Review ID    | string    | PK                 | Unique identifier for the review. Note: duplicate Review IDs exist in the source data, suggesting the same review may be associated with multiple orders or was recorded more than once. |
| Order ID     | string    | FK → `fact_orders` | Links the review to its corresponding order.                                                                                                                                             |
| Review Score | int64     |                    | Customer satisfaction rating on a scale of 1 to 5, where 1 is the lowest and 5 is the highest. Used for all satisfaction analysis in the report.                                         |

### `dim_customer_segments`
**Description:** A derived dimension that classifies each unique customer as either a one-time or returning buyer, based on the total number of delivered orders associated with their Customer Unique ID. This table is built by aggregating the order data in the staging layer.

**Granularity:** One row per unique customer.

**Source:** `stg_customer_segments`

| Column             | Data Type | Key | Description                                                                                                       |
| ------------------ | --------- | --- | ----------------------------------------------------------------------------------------------------------------- |
| Customer Unique ID | string    | PK  | The unique customer identifier consistent across multiple orders. Links to `dim_customers[Customer Unique ID]`.   |
| Order Count        | int64     |     | Total number of orders placed by this customer across the dataset period. Used to derive Customer Type.           |
| Customer Type      | string    |     | Segmentation label derived from Order Count. Values: `One-Time` (Order Count = 1), `Returning` (Order Count > 1). |

### `dim_seller_segments`
**Description:** A derived dimension that classifies each seller into a rating tier based on the average review score across all their delivered orders. This table is built by aggregating the review and order data in the staging layer.

**Granularity:** One row per seller.

**Source:** `stg_seller_segments`

| Column               | Data Type | Key | Description                                                                                                                                               |
| -------------------- | --------- | --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Seller ID            | string    | PK  | Unique identifier for each seller. Links to `dim_sellers[Seller ID]` and `fact_order_items[Seller ID]`.                                                   |
| Average Review Score | double    |     | The mean review score across all delivered orders fulfilled by this seller. Ranges from 1.0 to 5.0.                                                       |
| Seller Rating Tier   | string    |     | Segmentation label derived from Average Review Score. Values: `Low (1-2)`, `Medium (3)`, `High (4-5)`. Used to group sellers by satisfaction performance. |

### `dim_date`
**Description:** A calculated date dimension table generated from DAX using the `CALENDAR()` function, spanning the full range of order dates in `fact_orders`. It provides a complete, contiguous date timeframe with time intelligence attributes used for period-over-period comparisons and time-based filtering across all report pages.

**Granularity:** One row per calendar date.

**Source:** Calculated table — generated via DAX `CALENDAR(MIN(fact_orders[Order Date]), MAX(fact_orders[Order Date]))`. Not sourced from any staging table.

| Column             | Data Type | Key | Description                                                                                                       |
| ------------------ | --------- | --- | ----------------------------------------------------------------------------------------------------------------- |
| Date               | date      | PK  | The calendar date. Serves as the primary key for the date dimension and links to the Order Date in `fact_orders`. |
| Year               | int64     |     | Calendar year extracted from the date (e.g., 2017, 2018).                                                         |
| Month Number       | int64     |     | Calendar month number (1–12).                                                                                     |
| Month Name         | string    |     | Full name of the month (e.g., January, February).                                                                 |
| Month Short        | string    |     | Abbreviated month name (e.g., Jan, Feb).                                                                          |
| Quarter Number     | int64     |     | Calendar quarter number (1–4).                                                                                    |
| Quarter            | string    |     | Quarter label (e.g., Q1, Q2, Q3, Q4).                                                                             |
| Year Month Number  | string    |     | Year and month combined as YYYYMM (e.g., 201701). Used for chronological sorting of month labels in visuals.      |
| Year Month         | string    |     | Human-readable year-month label (e.g., Jan 2017). Used as the x-axis label in time series visuals.                |
| Day of Week Number | int64     |     | Day of the week as a number (1 = Monday, 7 = Sunday).                                                             |
| Day Name           | string    |     | Full name of the day (e.g., Monday, Tuesday).                                                                     |
| Week Number        | int64     |     | ISO week number of the year (1–53).                                                                               |