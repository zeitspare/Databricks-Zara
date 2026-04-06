%%markdown


# Lake House Architecture

```plaintext
┌───────────────┐     ┌────────────────┐             ┌───────────────┐          ┌──────────────┐           ┌──────────────┐
│ Data Source   │ ──▶ │ Bronze Tables  │  ────────▶  │ Silver Tables │      ──▶ │ Gold Tables  │    ──▶    │ platinum     │
│               │     │                │   (valid    │               │          │              │           │  Tables      │  
│(retail-source)│     │ (retail-bronze)│   records)  │(retail-silver)│          │(retail-gold) │           │              │
└───────────────┘     └────────────────┘             └───────────────┘          └──────────────┘           └──────────────┘
(landing_zone)          (raw_zone)                    (cleaned_zone)            (semantic_zone)               (row level logic)
  csv uploads           metadata (all tables)         normalized data           complex Aggregation data      not in this sample
                        + source & file name          + UTC -> timestamp        
                        + ingested time               + Duplicates (Window) 
                        + Auditable column names
                            │
                            │                        ┌───────────────┐
                            └─────────────────────▶  │ Quarantine    │
                               (invalid records)     │    Tables     │
                                                     │(retail-silver)│
                                                     └───────────────┘
```

## Silver Validation
- Duplicates - window function for duplicates. row number in discounts & date in products sold (same pieces sold number more than 2 entries in date)
- Duplicate count - rows / duplicates per table processing
- constratints - Product ID not null 
- constraints - ID matches among other tables
- Number Anamolies - Discount numbers
- Rount count & uniqueness with Null analysis

## Gold Validation
- Business Metrics
- as wide as possible = Reusuable Aggregations ( so we can filter woman, men , matrial etc.)
- trying to make as much as calculation happen in table tevel. so BI wont struggle
- Reducing the complex joins in table level. so BI wont struggle.
- value check (not done in this code)

## Approaches
- Generic Functions for validation step
- quarantine tables for invalid records.  
- Data Quality Daily records registration
- Data Auditability 
- ANSI SQL approach for aggregation (easy for multiple system and Business people logic checking)
- havent used widgets, but easy integration to help Databricks asset bundle (CI/CD)
- all tables are currently (overwrite and mergeschema) 

## Data Handling & Risk planning 
- no time in discount
- grain = ProductID + city + date (products sold is main table)
- 1 detailed silverver fact table + 4 reusing Gold Aggregate Tables

```plaintext
      ┌───────────────┐                             ┌───────────────┐
      │   Silver      │     ─────────────────────▶  │ Strict        │ window function - row number (picked the recent entry)
      │   Validation  │                             │ Deduplication │ removed same date sales. 
      │   layer       │                             │      (A)      │
      └───────────────┘                             └───────────────┘
                                                           |
                                                           |
                                                           |                         
                                                           |                        fact_sales
                                                    ┌───────────────┐             ┌───────────────┐             ┌───────────────┐
                              is null               │ Anamolies     │ ──────────▶ │   Silver      │   (Fact     │    Gold       │    
                              is duplicate          │   Fllag       │             │   Tables      │   Tables)   │    Tables     │
                              is out of range       │      (B)      │             │               │             │               │
                                                    └───────────────┘             └───────────────┘             └───────────────┘
                                                                                          |                             | 
                                                                                          |                             | 
                                                                                          |                             | 
                                                                                  ┌───────────────┐              ┌───────────────┐
                                                           product + city + date  │ Aggregation   │  ──────────▶ │   Business    │
                                                                 group by         │     Grain     │              │    Gold       │
                                                                                  │      (c)      │              │   Validation  │
                                                                                  └───────────────┘              └───────────────┘
                                                                                                          Business level + Analytics MEtrics Ready
                                                                                                          - section (MAN/WOMAN)
                                                                                                          - SEASON 
                                                                                                          - material
                                                                                                          - origin
```

## Further Optimization
- currently data skipping is not done 
- Star schema with aditions city and date for advanced analytics
- Discount table needs date  (start and end)
```plaintext
                    +----------------------+
                    |  Dim_Product_info    |
                    |----------------------|
                    | Product_ID (PK)      |
                    | Product Position     | 
                    | Promotion            |
                    | Product Category     |
                    | Seasonal             |
                    | Brand                |
                    | URL                  |
                    | Name                 |
                    | Description          |
                    | Price                |
                    | Currency             |    +----------------------+     +----------------------+
                    | Terms                |    |   Dim_Products_sold  |     |     Dim_Discount     |
                    | Section              |    |----------------------|     |----------------------|
                    | Season               |    |     Product_ID (PK)  |     |   Product_ID (PK)    |
                    | Material             |    |     products_sold    |     |   city               |
                    | Origin               |    |     city             |     |   discount           |
                    |                      |    |     time             |     |                      |
                    +----------------------+    +----------------------+     +----------------------+
                           |                              |                            |
                           |                              |                            |
                           |────────────────────────────────────────────────────────────
                           |
+------------------+       |       +----------------------+
|  Dim_Location    |       |       |      Dim_Date        |        Dataskipping. 
|------------------|       |       |----------------------| ( creating sales date time and month dimension )
| City (PK)        |       |       | Date (PK)            | (city with location id or sometihing added in tables will benifit)
+------------------+       |       +----------------------+
                           |
                           |
                    +----------------------+
                    |     Fact_Sales       | (silver Table)
                    |----------------------|
                    | product_id (FK)      |  (Autoloader Function will benift here using read stream and apppend write method)
                    | city (FK)            |
                    | date (FK)            |
                    | pieces_sold          |                        (Sales Gold Tables with reusable Metrics) Aggregated tables
                    | unit_price           |──────────────────────────────────────────────────────────────────────────────────────────────────  
                    |                      |            |                            |                          |                             |
(Pieces sold *      | revenue              |            |                            |                          |                             |
   unit_price)      | discount             |  +----------------------+    +----------------------+   +----------------------+   +----------------------+
revenue * Discount  | discount_amount      |  |   Gold_city_sales    |    | Gold_Product_sales   |   | Gold_fact_sales      |   |Gold_product_attribute|
revenue - Discount  | net_revenue          |  +----------------------+    +----------------------+   +----------------------+   +----------------------+
                    +----------------------+  |   City               |    |  Product_ID          |   |  Product_ID          |   |    Product_ID        |
                           |                  |   total_volume       |    |  total_sold          |   |  City                |   |    City              |
                           |                  |   total_net_revenue  |    |  total_net_revenue   |   |  date                |   |    section           |
                           |                  |   avg_discount       |    |                      |   |  total_pieces_sold   |   |    season            |
                    +----------------------+  +----------------------+    +----------------------+   |  total_revenue       |   |    material          |
                    |   Fact_Discount      |      sum(pieces_sold)           sum(pieces_Sold)        |  total_discount      |   |    origin            |
                    |----------------------|      sum(net_revenue)           sum(net_Revenue)        |  total_net_revenue   |   |    total_sold        |
                    | Product_ID (FK)      |      AVG (discount)                                     |   avg_price          |   |    total_revenue     |
                    | City (FK)            |                                                         |   avg_discount       |   |    avg_discount      |
( slow changing     | Discount             |                                                         +----------------------+   +----------------------+
   type 2 tables    |                      |                                                            sum(pieces_sold)           sum(pieces_sold)
   will benifit     |                      |                                                            sum(revenue)               sum(net_revenue)
    here )          +----------------------+                                                            sum(discount_amount)       aug(discount)
                                                                                                        sum(net_revenue)
                                                                                                        avg(unit_price)
                                                                                                        avg(discount)
```

## Data quality Report
```plaintext
Discount Table 
----------------------------------------------------------------------------------------------------
| total_rows | non_duplicate_rows | duplicate_rows | null_rows | non_null_rows | out_of_range_rows |
|------------|-------------------|----------------|-----------|----------------|-------------------|
| 185962     | 184121            | 1841           | 0         | 185962         | 0                 |
----------------------------------------------------------------------------------------------------

Products sold Table 
--------------------------------------------------------------------------------
| total_rows | non_duplicate_rows | duplicate_rows | null_rows | non_null_rows | 
|------------|-------------------|----------------|-----------|----------------|
| 1010492    | 184121            | 826371         | 0         | 1010492        |
--------------------------------------------------------------------------------

Prodcut Information Table
--------------------------------------------------------------------------------
| total_rows | non_duplicate_rows | duplicate_rows | null_rows | non_null_rows |
|------------|-------------------|----------------|-----------|----------------|
| 20252      | 20252             | 0              | 0         | 20252          | 
--------------------------------------------------------------------------------
```
