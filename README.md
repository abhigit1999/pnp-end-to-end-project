# pnp-end-to-end-project

A production-grade end-to-end data engineering pipeline built on Microsoft Azure, 
ingesting and transforming competitor retail pricing data across 50+ UK & IE retailers into 
a clean, business-ready lakehouse for analysts, ML modelling, and Acuity Pricing integration.

---

---

## Medallion Architecture

### Bronze Layer
- Raw CSV files copied from Ascential ADLS source
- Incremental ingestion — files modified in last 24 hours only
- Stored at: `brozen/final/YYYY-MM-DD/`
- Format: CSV (as-is from source)
- No transformations applied

### Silver Layer
- Cleaned and standardised data
- Transformations applied:
  - Column name standardisation (lowercase, underscores)
  - Price cleaning (remove £, GBP, negatives, zero)
  - Boolean standardisation (true/True/Y/1/yes → true)
  - Category series concatenation (main >> l1 >> l2 >> l3)
  - SKU/ID cleaning (uppercase, remove dashes)
  - Null handling via fillna per column type
  - Deduplication on SKU + retailer + date
- Format: Parquet
- Partitioned by: `ingestion_date`, `retailer_name`

### Gold Layer
- Business-ready data — single source of truth
- Business rules applied:
  - Price validation (< £0.20 or > £5000 → null)
  - Promo price logic (null if not on promo or >= regular price)
  - Effective price (promo price if on promo, else regular price)
  - Price band (Budget / Mid-range / Premium / Luxury)
  - Data quality flags (valid / price_missing / no_upc)
- Format: Delta
- Partitioned by: `date`
- Loaded into SQL Warehouse via incremental MERGE

---

## Pipeline Flow



## Tech Stack

| Layer             | Technology                          |
|-------------------|-------------------------------------|
| Orchestration     | Azure Data Factory + Databricks     |
| Storage           | Azure Data Lake Storage Gen2 (ADLS) |
| Transformation    | Azure Databricks + PySpark          |
| Warehouse         | Databricks SQL Warehouse            |
| Secret Management | Azure Key Vault                     |
| Alerting          | Azure Logic Apps                    |
| Visualisation     | Power BI                            |
| Version Control   | GitHub + GitHub Actions             |
| Format            | Parquet (Silver) + Delta (Gold)     |


## Pipeline Flow
ADF Trigger (Daily, 11:00 AM IST)

│

▼

Copy Activity — Ascential → Bronze Staging

│

▼

Get Metadata + ForEach — filter files modified in last 24h

│

▼

Bronze Final — brozen/final/YYYY-MM-DD/

│

▼

Databricks Job (Scheduled, 11:30 AM IST)

├── Silver Notebook  → clean, standardise, dedupe → silver/

├── Gold Notebook    → business rules, effective_price → gold/prices/

└── Warehouse MERGE  → incremental upsert → asc_pnp.pnp_gold.prices_current

│

▼

Watermark Log — asc_pnp.pnp_gold.watermark (rows, status, errors)

│

├──→ Power BI (Scheduled refresh, 12:00 PM IST)

│     ├── Business dashboard (prices, prices_current)

│     └── DE Pipeline Health dashboard (watermark)

│

└──→ Logic App Alerts (success/failure email)


## Single Source of Truth — Unity Catalog

All Gold data is registered under Unity Catalog for governed, SQL-based access:
asc_pnp (catalog)

└── pnp_gold (schema)

├── prices           — full history, partitioned by date

├── prices_current   — latest snapshot per sku + retailer (MERGE target)

└── watermark        — pipeline run audit log
## Data Quality & Reliability

- Per-row `data_quality` flag (`valid` / `price_missing` / `sku_missing` / `no_upc` / etc.)
- Watermark table logs `silver_rows`, `gold_rows`, `status`, and `error_message` for every run — including failures
- Incremental writes via `partitionBy(date)` + `replaceWhere` — safe to rerun without duplication
- Incremental MERGE into `prices_current` keyed on `sku + retailer_name`, updating only changed rows
