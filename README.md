%%markdown


## 🔄 Lake House Architecture

```plaintext
┌───────────────┐     ┌────────────────┐       ┌───────────────┐          ┌──────────────┐
│ Data Source   │ ──▶ │ Bronze Tables  │  ──▶  │ Silver Tables │      ──▶ │ Gold Tables  │
│               │     │                │       │               │          │              │
│(retail-source)│     │ (retail-bronze)│       │(retail-silver)│          │(retail-gold) │
└───────────────┘     └────────────────┘       └───────────────┘          └──────────────┘
(landing_zone)          (raw_zone)              (cleaned_zone)            (semantic_zone)
  csv uploads           metadata (all tables)   normalized data           complex Aggregation data
                        + source file name      + UTC -> timestamp
                        + ingested time         + Duplicates (Window) 
                           
```