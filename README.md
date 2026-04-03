%%markdown


# Lake House Architecture

```plaintext
┌───────────────┐     ┌────────────────┐             ┌───────────────┐          ┌──────────────┐
│ Data Source   │ ──▶ │ Bronze Tables  │  ────────▶  │ Silver Tables │      ──▶ │ Gold Tables  │
│               │     │                │   (valid    │               │          │              │
│(retail-source)│     │ (retail-bronze)│   records)  │(retail-silver)│          │(retail-gold) │
└───────────────┘     └────────────────┘             └───────────────┘          └──────────────┘
(landing_zone)          (raw_zone)                    (cleaned_zone)            (semantic_zone)
  csv uploads           metadata (all tables)         normalized data           complex Aggregation data
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

## Data Risk  & Handling Reports
```plaintext
                                                    ┌───────────────┐
                            ─────────────────────▶  │ Strict        │
                                                    │ Deduplication │
                                                    │      (A)      │
                                                    └───────────────┘
                                                    ┌───────────────┐
                            ─────────────────────▶  │ Aggregation   │
                                                    │               │
                                                    │      (B)      │
                                                    └───────────────┘
                                                    ┌───────────────┐
                            ─────────────────────▶  │ Anamolies     │
                                                    │  Flag         │
                                                    │      (c)      │
                                                    └───────────────┘
```
