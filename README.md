# pnp-end-to-end-project

A production-grade end-to-end data engineering pipeline built on Microsoft Azure, 
ingesting and transforming competitor retail pricing data across 8 UK retailers into 
a clean, business-ready lakehouse for analysts, ML modelling, and Acuity Pricing integration.

---

## Architecture Overview
Ascential ADLS (Source)
↓
Azure Data Factory (Incremental Ingestion)
↓
Bronze Layer — Raw CSVs (ADLS Gen2)
↓
Silver Layer — Cleaned + Standardised (Parquet, partitioned by date)
↓
Gold Layer — Business-Ready (Delta, partitioned by date)
↓
Databricks SQL Warehouse (Single Source of Truth)
↓
┌──────────────┬──────────────┬──────────────┐
↓              ↓              ↓              ↓
Analysts    ML Team      Acuity Team    DE Dashboard
(SQL)      (Delta/Spark) (Integration)  (Monitoring)

---

## Retailers Covered

| Retailer     | Source File              |
|--------------|--------------------------|
| Argos        | pnp_raw_argos.csv        |
| Co-op        | pnp_raw_coop.csv         |
| Euro Giant   | pnp_raw_eurogiant.csv    |
| Fresh        | pnp_raw_fresh.csv        |
| John Lewis   | pnp_raw_johnlewis.csv    |
| M&S          | pnp_raw_mns.csv          |
| Sainsbury's  | pnp_raw_sainsburys.csv   |
| Tesco        | pnp_raw_tesco.csv        |

---

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
