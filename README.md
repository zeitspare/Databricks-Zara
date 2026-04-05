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

## Validation
- Duplicates - window function for duplicates. row number in discounts & date in products sold (same pieces sold number more than 2 entries in date)
- Duplicate count - rows / duplicates per table processing
- constratints - Product ID not null 
- constraints - ID matches among other tables
- Number Anamolies - Discount numbers
- Rount count & uniqueness with Null analysis

## Approaches
- Generic Functions for validation step
- quarantine tables for invalid records.  
- Data Quality Daily records registration
- Data Auditability 

## Data Risk  & Handling Reports
- no time in discount
- grain = ProductID + city + date
```plaintext
      ┌───────────────┐                             ┌───────────────┐
      │   Silver      │     ─────────────────────▶  │ Strict        │ window function - row number (picked the recent entry)
      │   Validation  │                             │ Deduplication │ removed same date sales. 
      │   layer       │                             │      (A)      │
      └───────────────┘                             └───────────────┘
                                                           |
                                                           |
                                                           |
                                                           |              
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
                                                                                                                  
```

## Further Optimization
- currently data skipping is not done 
- Star schema with aditions city and date for advanced analytics
- Discount table needs date  (start and end)
```plaintext
                    +----------------------+
                    |     Dim_Product      |
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
                    | Currency             |
                    | Terms                |
                    | Section              |
                    | Season               |
                    | Material             |
                    | Origin               |
                    +----------------------+

                           |
                           |
                           |
+------------------+       |       +----------------------+
|  Dim_Location    |       |       |      Dim_Date        |        Dataskipping. 
|------------------|       |       |----------------------| ( creating sales date time and month dimension )
| City (PK)        |       |       | Date (PK)            | (city with location id or sometihing added in tables will benifit)
+------------------+       |       +----------------------+
                           |
                           |
                    +----------------------+
                    |     Fact_Sales       |
                    |----------------------|
                    | Product_ID (FK)      |  (Autoloader Function will benift here using read stream and apppend write method)
                    | City (FK)            |
                    | Time (FK)            |
                    | Pieces_Sold          |
                    +----------------------+

                           |
                           |
                    +----------------------+
                    |   Fact_Discount      |
                    |----------------------|
                    | Product_ID (FK)      | ( slow changing type 2 tables will benift here.)
                    | City (FK)            |
                    | Discount             |
                    +----------------------+
```



