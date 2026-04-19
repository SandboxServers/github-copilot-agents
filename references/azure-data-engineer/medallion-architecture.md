# Medallion Architecture

## Layer Design

```
┌─────────────────────────────────────────────────────────┐
│ LANDING ZONE (optional)                                  │
│ Raw files from external systems, event notifications     │
│ Format: source format (CSV, JSON, API responses)         │
└────────────────────┬────────────────────────────────────┘
                     │ ingestion (ADF, Auto Loader, Event Streams)
┌────────────────────▼────────────────────────────────────┐
│ BRONZE (Raw)                                             │
│ Source data preserved exactly as received                │
│ Format: Delta (append-only, schema-on-read)              │
│ Partition: ingestion date                                │
│ Retention: long (source of truth for replay)             │
│ Quality: none — raw is raw                               │
└────────────────────┬────────────────────────────────────┘
                     │ validation, dedup, type casting, standardization
┌────────────────────▼────────────────────────────────────┐
│ SILVER (Refined)                                         │
│ Cleansed, validated, deduplicated, standardized          │
│ Format: Delta (managed tables)                           │
│ Partition: business keys (date, region, entity)          │
│ Quality: schema enforced, nulls handled, types correct   │
│ Joins: cross-source integration happens here             │
└────────────────────┬────────────────────────────────────┘
                     │ aggregation, dimensional modeling, business logic
┌────────────────────▼────────────────────────────────────┐
│ GOLD (Business-Ready)                                    │
│ Optimized for consumption: BI, ML, APIs                  │
│ Format: Delta (optimized, Z-ordered, pre-aggregated)     │
│ Partition: consumer-specific (by report, by model)       │
│ Quality: business rules enforced, SLAs defined           │
│ Security: row-level, column-level access controls        │
└─────────────────────────────────────────────────────────┘
```

## Common Mistakes in Medallion Implementation

1. **Gold is just Silver with a rename** → Gold should be purpose-built for consumers. Different gold tables for different use cases.
2. **No quality gates between layers** → Data quality must INCREASE at each layer. Define expectations/constraints.
3. **Bronze isn't truly raw** → Don't transform in bronze. If you need to convert format (CSV → Delta), that's fine, but don't filter or modify data.
4. **Everything in one big lakehouse** → Separate by domain. Each domain team owns their bronze-silver-gold chain.
5. **No replay capability** → Bronze must support re-processing. Keep it immutable and append-only.
6. **Skipping silver** → Going straight from bronze to gold creates unmaintainable gold layers. Silver is where data debt gets paid.
