# Chicago Taxi Lakehouse (Databricks CE)

Hands-on project built on Databricks Community Edition using the Chicago Taxi dataset.
The goal is to simulate a Lakehouse with bronze/silver/gold layers by **path** and to cover the main topics of the **Databricks Certified Data Engineer Associate Exam Guide (2025-07-30)**.

Key ideas:

* Databricks CE only (no Unity Catalog, no Auto Loader, no Lakeflow Jobs).
* Medallion layout by path: `/Volumes/{catalog}/{schema}/{volume}/{bronze|silver|gold}`.
* Delta Lake tables written by path; SQL views used to simulate UC-like catalogs/schemas.
* Each notebook is aligned to at least one section of the official exam guide.

---

## 1. Notebooks Overview (Canonical Skeleton)

This is the canonical notebook structure for this repo.
When changing or adding notebooks, update this section first.

```text
notebooks/
  00_setup_project.py
  01_bronze_ingestion_autoloader.py
  02_silver_transformations.py
  03_gold_analytics.py
  04_quality_governance.py
  05_pipeline_ops_sim.py
```

High-level mapping to exam guide sections:

* **Section 1 – Databricks Intelligence Platform**
* **Section 2 – Development and Ingestion**
* **Section 3 – Data Processing & Transformations**
* **Section 4 – Productionizing Data Pipelines**
* **Section 5 – Data Governance & Quality**

---

## 2. Notebook Details and Exam Coverage

### 00_setup_project.py

**Purpose**

* Initialize widgets for `catalog`, `schema`, and `volume`.
* Build canonical paths for bronze/silver/gold layers under `/Volumes/{catalog}/{schema}/{volume}/...`.
* Create or validate the working database/schema in CE.
* Provide a quick sanity check that the environment is ready (simple write/read of a tiny Delta table or path listing).
* Document, via comments/markdown, how this setup would look in a full Unity Catalog environment (catalogs, schemas, volumes, workspace objects).

**Exam coverage**

* **Section 1 – Databricks Intelligence Platform**

  * Workspaces, clusters (conceptually), notebooks, jobs.
  * Unity Catalog concepts (simulated in CE with views and path-based tables).
  * Basic Lakehouse architecture: bronze/silver/gold layout, Delta storage by path.

---

### 01_bronze_ingestion_autoloader.py

**Purpose**

* Ingest the Chicago Taxi CSV file(s) into the **bronze** layer using an explicit schema.
* Read raw data from the upload location and write it as Delta by path:

  * `/Volumes/{catalog}/{schema}/{volume}/bronze/chicago_taxi/`
* Add ingestion metadata:

  * `ingestion_ts`, `_metadata.file_path` (when available), and other useful columns.
* Demonstrate idempotent ingestion for demo purposes (overwrite) and comment on incremental patterns.
* Simulate Auto Loader and COPY INTO behaviors in CE through comments and simple batch patterns.

**Exam coverage**

* **Section 2 – Development and Ingestion**

  * Reading data from files using explicit schemas.
  * Ingesting data into Delta tables (by path).
  * Using metadata columns and timestamps for ingestion tracking.
  * Conceptual understanding of Auto Loader and COPY INTO (described in comments, approximated in CE).

* Secondary:

  * **Section 1 – Platform** (basic workspace/paths usage in a practical context).

---

### 02_silver_transformations.py

**Purpose**

* Transform bronze data into a curated **silver** table.

* Clean and standardize columns (types, names, null handling).

* Apply filters, derived columns (e.g., trip duration, distance buckets), and basic business rules.

* Implement simple incremental logic (for example, filtering by `ingestion_ts` or date columns).

* Use reusable data quality checks from `src/utils/dq.py`:

  * Non-negative fare/amount columns, basic row-count checks between bronze and silver.

* Write silver data to:

  * `/Volumes/{catalog}/{schema}/{volume}/silver/chicago_taxi/`

* Optionally expose a SQL view to simulate a UC-managed table for silver.

**Exam coverage**

* **Section 3 – Data Processing & Transformations**

  * Transformations with Spark DataFrames and/or SQL.
  * Incremental processing concepts.
  * Data quality as part of transformation (basic checks).
  * Handling schema and type conversions between layers.

---

### 03_gold_analytics.py

**Purpose**

* Build **gold**-level aggregated tables for analytics and reporting.
* Aggregate over time grain (e.g., monthly) and dimensions (payment type, pickup zone, etc.).
* Ensure KPIs and metrics are computed on clean silver data only.
* Handle nulls and invalid values consistently.
* Write gold data to:

  * `/Volumes/{catalog}/{schema}/{volume}/gold/chicago_taxi_metrics/`
* Create SQL views to simulate serving tables for BI tools, using `catalog.schema.view_name` style where possible.

**Exam coverage**

* **Section 3 – Data Processing & Transformations** (analytics-focused)

  * Aggregations and groupings over large datasets.
  * Using gold tables as serving layer for downstream tools.
  * Performance-aware transformations (basic column pruning, avoiding unnecessary work).

* Secondary:

  * **Section 1 – Platform** (Lakehouse pattern and serving layer positioning).

---

### 04_quality_governance.py

**Purpose**

* Inspect Delta tables for metadata and history:

  * `DESCRIBE DETAIL` for storage, schema, and properties.
  * `DESCRIBE HISTORY` to understand operations and versions.
* Demonstrate basic **time travel** queries for debugging or point-in-time reads.
* Apply additional data quality checks using `src/utils/dq.py`:

  * Validate non-negative revenue, counts across layers, and any other business rule.
* Document, via comments, how table-level permissions, catalogs, schemas, and lineages would be managed in Unity Catalog.
* Introduce basic maintenance concepts:

  * Explain `OPTIMIZE` and `VACUUM` in comments, noting CE limitations when applicable.

**Exam coverage**

* **Section 5 – Data Governance & Quality**

  * Delta table metadata, history, and time travel.
  * Data quality validation and monitoring patterns.
  * Conceptual governance in Unity Catalog (permissions, lineage, auditing).

* Secondary:

  * **Section 4 – Productionizing Data Pipelines** (quality gates in pipelines).

---

### 05_pipeline_ops_sim.py

**Purpose**

* Simulate a production pipeline orchestration for the bronze → silver → gold flow.
* Use `dbutils.notebook.run()` to chain the notebooks:

  * `01_bronze_ingestion_autoloader.py`
  * `02_silver_transformations.py`
  * `03_gold_analytics.py`
* Pass parameters (catalog, schema, volume) between notebooks.
* Implement simple logging using `src/utils/log.py`:

  * Start/end messages, basic status tracking.
* Enforce “fail fast” behavior:

  * Use asserts or explicit checks; if something fails, the orchestration notebook stops quickly.
* Each notebook should end with `dbutils.notebook.exit("OK")` to signal success.
* Include comments explaining how this pattern relates to:

  * Jobs, tasks, and job clusters.
  * Lakeflow Jobs and task dependencies in a full workspace (conceptual only).

**Exam coverage**

* **Section 4 – Productionizing Data Pipelines**

  * Orchestrating multiple steps into a pipeline.
  * Parameterization and reuse of notebooks.
  * Operational concerns: logging, failure handling, idempotency at a high level.

* Secondary:

  * **Sections 2–3** (because it invokes the ingestion and transformation notebooks).

---

## 3. Supporting Code (src/, tests, conf, scripts)

Although the exam focuses mainly on the notebooks and platform concepts, the supporting code reflects how a cloud data engineer would structure a real project.

### src/utils

* `io.py`

  * Centralizes path construction for bronze/silver/gold:

    * `/Volumes/{catalog}/{schema}/{volume}/{layer}/...`
  * Provides helpers to read/write Delta tables by path.
  * Aligns with:

    * **Section 1** (workspace and objects)
    * **Section 2** (ingestion helpers).

* `dq.py`

  * Provides reusable data quality checks:

    * Non-negative columns, null ratios, row count comparisons, etc.
  * Used in notebooks 02, 03, and 04.
  * Aligns with:

    * **Section 5** (Governance & Quality).

* `log.py`

  * Tiny logging helper for pipelines (print wrapper with consistent format).
  * Used by `05_pipeline_ops_sim.py`.
  * Aligns with:

    * **Section 4** (Productionizing / Ops).

### src/pipelines

* `bronze.py`, `silver.py`, `gold.py`

  * Optional programmatic versions of the notebooks for local or job-style execution.
  * Reinforce the same logic implemented in 01, 02, and 03.
  * Map to:

    * **Sections 2 and 3**, and partially **Section 4**.

### tests

* `test_dq.py`

  * Unit tests for `dq.py` functions.
  * Reinforces the idea that data quality checks should be testable and reliable.
  * Supports:

    * **Section 5** (quality mindset).

### conf

* `default.yml`, `dev.yml`

  * Store default values for catalog, schema, and volume.
  * Can be used to drive widgets or local scripts.
  * Support:

    * **Section 1** (environment/configuration patterns).

### scripts and tooling

* `scripts/export_notebooks.sh`, `scripts/run_local.sh`

  * Utilities for exporting notebooks as `.py` and running local checks.
* `.github/workflows/ci.yml`, `.pre-commit-config.yaml`, `pyproject.toml`

  * Standard Python tooling for formatting (black, isort), linting (flake8), and tests (pytest).
  * Not exam content, but demonstrate modern engineering practices expected from a cloud data engineer.

---

## 4. How to Use This Project for Exam Prep

1. **Read the Exam Guide (2025-07-30)** and map each bullet to at least one notebook above.
2. Use:

   * `00` for platform concepts and Lakehouse layout.
   * `01–03` to practice ingestion and transformations on a real dataset.
   * `04` to explore Delta metadata, history, and time travel.
   * `05` to understand the basics of pipeline orchestration and operational patterns.
3. When practicing, focus on:

   * What the code does.
   * Which exam section it illustrates.
   * How it would change in a full Unity Catalog + Lakeflow environment.

This README is the canonical description of the project structure and exam coverage.
Whenever the project changes, update this document first and use it as the single source of truth for new study prompts.
