%%markdown


## 🔄 Lake House Architecture

```plaintext
┌───────────────┐     ┌────────────────┐     ┌───────────────┐     ┌──────────────┐
│ Data Source   │ ──▶ │ Bronze Tables  │ ──▶ │ Silver Tables │ ──▶ │ Gold Tables  │
│               │     │                │     │               │     │              │
│(retail-source)│     │ (retail-bronze)│     │(retail-silver)│     │(retail-gold) │
└───────────────┘     └────────────────┘     └───────────────┘     └──────────────┘
(landing_zone)          (raw_zone)   
  csv uploads       metadata source file
```