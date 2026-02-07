Disclaimer

This is a learning/portfolio prototype using synthetic data. It is not connected to any real utility systems and should not be used for operational decisions without proper data, validation, and governance.

# Grid Reliability & Failure Prevention (Azure Prototype)

A small but realistic prototype that shows how to build an **operational reliability** analytics pipeline for an electricity distribution network. The system produces a daily **feeder risk view** and **actionable alerts** designed for operational decision-making (inspection, prioritisation, monitoring), not vanity reporting.

> **Note:** This project uses **synthetic/sample data** to demonstrate architecture, data modeling, data quality patterns, and decision logic. The same structure can be wired to real OMS/CMMS/GIS/SCADA sources.

---

## What this does

Every run:
1. Ingests raw CSV files from **ADLS Gen2** using **Azure Data Factory**
2. Loads them into **Azure SQL** `stg` tables (raw-as-text landing pattern)
3. Cleans/casts data into `silver` tables (typed, conformed)
4. Builds a daily analytical backbone: `silver.feeder_day`
5. Generates:
   - `gold.feeder_risk_daily` (risk score + explainable reasons)
   - `gold.feeder_alerts` (alerts with evidence + recommended action)
6. Records basic **data quality checks** in `ops.data_quality_daily`

---

## Why this matters (business context)

Grid operators and planners need to answer:
- Which feeders are showing patterns of **repeat failures**?
- Where is customer impact increasing?
- Which areas/assets look riskier due to age and exposure?
- What should we **inspect or prioritise** before the next failure?

This prototype models that workflow with:
- a daily feeder-level feature table
- explainable rule-based risk scoring
- operational alerts built for action, not just reporting

---

## Architecture

**ADLS Gen2 (raw CSVs)** → **ADF Copy** → **Azure SQL (stg)** → **SQL transforms (silver)** → **feeder-day (silver)** → **risk + alerts (gold)**  
Plus: **ops** schema for pipeline/data quality tracking.

Schemas:
- `stg` – raw landing tables (strings to avoid ingestion conversion issues)
- `silver` – typed, cleaned, conformed tables
- `gold` – outputs used by ops/planning (risk table + alerts)
- `ops` – data quality results and pipeline logs

---

## Data model (core tables)

### Inputs (stg/silver)
- `silver.outages`  
  Unplanned vs planned outages, durations, customer impact.
- `silver.assets`  
  Feeder attributes (region, substation, age proxy, overhead exposure).

### Backbone (silver)
- `silver.feeder_day`  
  One row per **feeder per day**, with rolling reliability features (e.g., outages last 30 days).

### Outputs (gold)
- `gold.feeder_risk_daily`  
  Daily risk score, risk level, and human-readable reasons.
- `gold.feeder_alerts`  
  Actionable alerts with severity, evidence, and recommended actions.

### Quality checks (ops)
- `ops.data_quality_daily`  
  Examples: missing keys, invalid durations, sanity checks.

---


---

## Setup (Azure)

### Prerequisites
- Azure Storage Account with **Hierarchical namespace enabled** (ADLS Gen2)
- Azure Data Factory
- Azure SQL Database

### 1) Create ADLS paths
Example:
- `raw/outages/dt=2024-01-02/outages.csv`
- `raw/assets/dt=2024-01-02/assets.csv`

### 2) Create SQL schemas/tables
Run scripts in `/sql` in order:
1. `00_schemas_ops.sql`
2. `01_stg_tables.sql`

### 3) ADF pipelines (Copy activity)
Create linked services:
- ADLS Gen2
- Azure SQL DB

Create datasets:
- `ds_outages_csv` → `stg.outages`
- `ds_assets_csv` → `stg.assets`

Run Copy pipelines to load raw CSVs into `stg.*`.

### 4) Build silver and gold layers
Run:
- `02_silver_outages.sql`
- `03_silver_assets.sql`
- `04_silver_feeder_day.sql`
- `05_gold_risk_daily.sql`
- `06_gold_alerts.sql`
- `07_ops_quality_checks.sql`

---


...


