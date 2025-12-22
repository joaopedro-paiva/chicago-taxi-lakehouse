# Chicago Taxi Lakehouse (Databricks CE)

Portfolio lakehouse project demonstrating **Databricks + Delta Lake** delivery patterns: **bronze/silver/gold**, **quality gates**, and **pipeline orchestration** (Community Edition compatible).

## What this demonstrates

* Medallion architecture (bronze → silver → gold) with clear layer contracts
* Delta Lake reliability: history + time travel for debugging
* Data quality gates (fail-fast checks)
* Orchestration pattern (multi-step pipeline with parameters + status)
* Governance framing: Unity Catalog concepts mapped to CE-compatible equivalents

## Architecture

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/16fbe94d-aa81-4484-a030-deb54e95d127" />



## Quickstart

1. Run `notebooks/00_setup_project.py`
2. Run `notebooks/01_bronze_ingestion_autoloader.py`
3. Run `notebooks/02_silver_transformations.py`
4. Run `notebooks/03_gold_analytics.py`
5. Run `notebooks/05_pipeline_ops_sim.py` (end-to-end)

## Outputs

* Bronze: `/Volumes/.../bronze/chicago_taxi/`
* Silver: `/Volumes/.../silver/chicago_taxi/`
* Gold: `/Volumes/.../gold/chicago_taxi_metrics/`

## Evidence

* Screenshots: `docs/screenshots/` (pipeline run + gold output + quality gate)
* Reusable utils: `src/utils/` (I/O, quality, logging)
* Tests: `tests/` (quality checks)

## Notes

Deep exam mapping lives in `docs/exam-coverage.md` (kept separate to keep the README recruiter-friendly).
