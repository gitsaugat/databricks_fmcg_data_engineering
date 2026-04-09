# Medallion Architecture Pipeline
### Databricks · Delta Lake · AWS S3 · PySpark

---

## Overview

A production-style data engineering pipeline implementing the Medallion Architecture (Bronze → Silver → Gold) on Databricks. Raw data ingested from AWS S3 is progressively refined across three layers — from untouched source records to validated, modeled, analytics-ready datasets — with full support for incremental processing, idempotent ingestion, and data quality enforcement.

---

## Stack

| Component | Technology |
|---|---|
| Compute & orchestration | Databricks |
| Storage format | Delta Lake |
| Source storage | AWS S3 |
| Transformation | PySpark · SQL |

---

## Architecture

The pipeline follows a domain-driven Medallion Architecture. Each business domain (customers, orders, products, product pricing) owns its full Bronze → Silver → Gold stack, keeping logic isolated and independently deployable.

```
AWS S3 (raw source files)
        │
        ▼
┌──────────────────────────────────────────────────────┐
│  BRONZE  —  raw ingestion, no transformation         │
│  Delta format, preserves source fidelity             │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│  SILVER  —  cleaning, validation, standardization    │
│  Schema enforcement · referential integrity checks   │
│  Invalid records quarantined to separate tables      │
└───────────────────────┬──────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│  GOLD  —  business-ready, analytics-ready            │
│  Fact tables · Dimension tables · Aggregated marts   │
└──────────────────────────────────────────────────────┘
```

---

## Project Structure

```
├── setup/
│   ├── setup_catalog_schema.py       # Unity Catalog + schema initialization
│   ├── setup_dim_dates.py            # Date dimension bootstrap
│   └── utilities.py                  # Shared helpers
├── customer/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── orders/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── products/
│   ├── bronze/
│   ├── silver/
│   └── gold/
└── product_pricing/
    ├── bronze/
    ├── silver/
    └── gold/
```

Each domain is fully self-contained. Shared setup logic lives separately and is invoked once at environment initialization.

---

## Layer Details

### Bronze — ingestion

Raw files are read from AWS S3 and written to Delta tables with no transformations applied. Source fidelity is preserved so any downstream issue can be debugged by replaying from bronze. All domains share the same pattern: read → write Delta → done.

### Silver — validation and standardization

Three operations applied consistently across all domains:

**Schema enforcement** — explicit types cast at ingestion, rejecting malformed records rather than silently coercing them.

**Referential integrity** — `left_anti` joins validate that foreign keys resolve correctly before records are promoted. Records that fail are written to quarantine tables (`invalid_customers`, `invalid_products`) rather than dropped, enabling auditing and root cause analysis without re-ingestion.

**Deduplication** — `ROW_NUMBER()` over a defined partition key selects the latest record per entity, ensuring silver tables represent current state.

### Gold — modeling

Fact and dimension tables are built from silver using standard dimensional modeling patterns.

**Fact table — `fact_sb_orders`**
- Grain: one row per order event
- Incremental strategy: append-only merge keyed on `order_id`
- Idempotent: re-running on the same data produces the same result with no duplicates

**Dimension tables — customers, products, product pricing**
- Built from latest silver records using `ROW_NUMBER()` deduplication
- Upsert via `MERGE` — new records inserted, changed records updated, deletions handled by convention

**Aggregated mart**
- Monthly sales aggregated per customer and product
- Pre-joined and pre-aggregated for direct BI consumption

---

## Incremental Processing

All pipelines are designed for incremental operation from the start, not retrofitted later. Staging tables act as a controlled buffer between ingestion and promotion — new records land in staging, are validated, then merged into the target. This gives three properties that matter in production:

- **Reliability** — failed runs leave target tables untouched; only successfully validated records are promoted
- **Idempotency** — re-running any pipeline step on the same input produces the same output
- **Auditability** — the path from raw S3 file to gold record is traceable at every step

---

## Data Quality

Invalid records are not dropped — they are quarantined. Every domain that performs referential integrity checks writes failing records to a dedicated table:

- `invalid_customers` — orders referencing unknown customer IDs
- `invalid_products` — orders or pricing referencing unknown product IDs

These tables support monitoring, alerting, and upstream debugging without requiring a full re-ingestion cycle.

---

## Delta Lake Features Used

| Feature | Purpose |
|---|---|
| ACID transactions | Safe concurrent reads and writes |
| Schema enforcement | Reject malformed records at write time |
| Time travel | Replay and audit any historical table state |
| MERGE | Upsert support for dimension tables and idempotent fact loads |

---

## Gold Outputs

| Table | Type | Grain |
|---|---|---|
| `fact_sb_orders` | Fact | Order-level events |
| `dim_customers` | Dimension | One row per customer (current) |
| `dim_products` | Dimension | One row per product (current) |
| `dim_product_pricing` | Dimension | One row per pricing record (current) |
| Monthly sales mart | Aggregated | Customer × product × month |

---

## What's Next

- Databricks Workflows or Apache Airflow for end-to-end orchestration
- Formal data quality framework (Great Expectations or Databricks DQ)
- Monitoring and alerting on quarantine table growth
- Partitioning and Z-ordering on high-volume fact tables
- dbt integration for SQL-based transformation layer
