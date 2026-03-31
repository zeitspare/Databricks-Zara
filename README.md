%%markdown


## 🔄 Lake House Architecture

```plaintext
┌───────────────┐     ┌────────────────┐       ┌───────────────┐      ┌──────────────┐
│ Data Source   │ ──▶ │ Bronze Tables  │  ──▶  │ Silver Tables │  ──▶ │ Gold Tables  │
│               │     │                │       │               │      │              │
│(retail-source)│     │ (retail-bronze)│       │(retail-silver)│      │(retail-gold) │
└───────────────┘     └────────────────┘       └───────────────┘      └──────────────┘
(landing_zone)          (raw_zone)              (cleaned_zone)         (semantic_zone)
  csv uploads           metadata                normalized data     complex Aggregation data
                        + source file name
                        + ingested time
                           
```