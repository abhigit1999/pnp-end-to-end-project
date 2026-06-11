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
| Orchestration     | Azure Data Factory (ADF)            |
| Storage           | Azure Data Lake Storage Gen2 (ADLS) |
| Transformation    | Azure Databricks + PySpark          |
| Warehouse         | Databricks SQL Warehouse            |
| Secret Management | Azure Key Vault                     |
| Alerting          | Azure Logic Apps                    |
| Monitoring        | Azure Monitor                       |
| Visualisation     | Power BI                            |
| Version Control   | GitHub + GitHub Actions             |
| Format            | Parquet (Silver) + Delta (Gold)     |
