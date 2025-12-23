# Chicago Taxi Lakehouse (Databricks FREE)

Portfolio lakehouse project demonstrating **Databricks + Delta Lake** delivery patterns using a **medallion architecture** (bronze → silver → gold), **data quality gates**, and **pipeline orchestration** — designed to be **Databricks Free Edition**.

## What this demonstrates

* **Medallion architecture** (bronze → silver → gold) with clear layer contracts
* **Delta Lake reliability**: table history + time travel for debugging
* **Data quality gates**: fail-fast checks and sanity validations
* **Orchestration pattern**: multi-step pipeline with parameters + status propagation
* **Governance framing**: Unity Catalog concepts mapped to CE-compatible equivalents

## Architecture

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/16fbe94d-aa81-4484-a030-deb54e95d127" />

## Quickstart (Databricks FE)

Run notebooks in this order:

1. `notebooks/00_setup_project`
2. `notebooks/01_bronze_ingestion_autoloader`
3. `notebooks/02_silver_transformations`
4. `notebooks/03_gold_analytics`
5. `notebooks/04_quality_governance`
6. `notebooks/05_pipeline_ops_sim` (end-to-end orchestration)

### Parameters

Notebooks accept widgets (defaults shown):

* `catalog`: `taxi_catalog`
* `schema`: `taxi_schema`
* `volume`: `taxi_volume`

## Outputs

* **Bronze**: `/Volumes/<catalog>/<schema>/<volume>/bronze/...`
* **Silver**: `/Volumes/<catalog>/<schema>/<volume>/silver/...`
* **Gold**: `/Volumes/<catalog>/<schema>/<volume>/gold/...`

## Evidence

* Screenshots: `docs/screenshots/` (pipeline run + gold output + quality gate)
* Reusable utils: `src/utils/` (I/O, quality, logging)
* Tests: `tests/` (quality checks)

## Notes

* Deep exam mapping lives in `docs/exam-coverage.md` (kept separate to keep the README recruiter-friendly).
