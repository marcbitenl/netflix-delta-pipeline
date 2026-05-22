# Spotify Lakehouse Platform

Metadata-driven ingestion platform built on Azure and Databricks focused on incremental processing, orchestration reliability and scalable downstream transformations.

The platform combines Azure Data Factory for CDC-based extraction orchestration with Databricks Auto Loader and Delta Live Tables for incremental downstream processing.

---

# Architecture Overview

<img width="2302" height="1528" alt="architecture" src="https://github.com/user-attachments/assets/1d6a2816-e5fb-4764-95e7-5972a3c14ce3" />

---

# Platform Stack

| Domain | Technology |
|---|---|
| Source | Azure SQL Database |
| Orchestration | Azure Data Factory |
| Storage | ADLS Gen2 |
| Processing | Azure Databricks |
| Streaming | Auto Loader + Delta Live Tables |
| Format | Delta Lake / Parquet |
| Transformations | PySpark / SQL |
| Monitoring | Azure Functions |
| CI/CD | Databricks Asset Bundles |
| Source Control | GitHub |

---

# Ingestion Framework

The ingestion layer was designed as a reusable orchestration framework inside Azure Data Factory.

Instead of maintaining isolated pipelines per entity, ingestion is dynamically driven through metadata-based orchestration and CDC watermark persistence.

Example ingestion payload:

```json
[
  {
    "schema": "dbo",
    "table": "DimUser",
    "cdc_col": "updated_at",
    "from_date": ""
  },
  {
    "schema": "dbo",
    "table": "FactStream",
    "cdc_col": "stream_timestamp",
    "from_date": ""
  }
]
```

The orchestration dynamically iterates through entities using a `ForEach` activity and parameterized extraction logic.

---

# Incremental CDC Strategy

Azure Data Factory is responsible for source-level incremental consistency.

CDC state is persisted directly into the Data Lake using JSON watermark files.

Execution flow:

```text
1. Read watermark
2. Extract incremental delta
3. Persist parquet into ADLS
4. Calculate latest CDC value
5. Update watermark
```

Dynamic extraction query:

```sql
SELECT *
FROM @{item().schema}.@{item().table}
WHERE @{item().cdc_col} >
'@{activity('LAST_CDC').output.value[0].cdc}'
```

This strategy enables scalable onboarding of new entities without orchestration redesign while avoiding unnecessary full-load extraction.

---

# Raw Incremental Landing Zone

Incremental parquet files are persisted into ADLS Gen2 as immutable raw ingestion artifacts before downstream streaming processing.

Characteristics:

- Append-only ingestion
- Incremental parquet generation
- Historical traceability
- Snappy compression
- CDC watermark persistence
- Partition-ready storage structure

Example:

```text
abfss://bronze@<storage-account>.dfs.core.windows.net/DimUser
```

---

# Empty Load Optimization

The orchestration includes a validation layer to avoid empty parquet persistence during incremental execution windows.

Validation logic:

```text
IF dataRead > 0
    Continue processing
ELSE
    Delete generated parquet
```

This prevents unnecessary storage growth and downstream processing overhead.

---

# Hybrid Incremental Processing Model

The platform adopts a hybrid incremental architecture.

## Azure Data Factory Responsibilities

- Source-level incremental extraction
- CDC watermark persistence
- Incremental parquet landing
- Orchestration state management

## Databricks Auto Loader Responsibilities

Once parquet files arrive in ADLS, Databricks Auto Loader handles streaming-style incremental ingestion into downstream Delta tables.

Auto Loader manages processing consistency using:

- `cloudFiles`
- checkpoint state management
- incremental file discovery
- streaming micro-batches

Example:

```python
spark.readStream.format("cloudFiles")
```

This architecture separates extraction consistency from downstream file-processing consistency.

---

# Streaming Processing Layer — Databricks

Downstream workloads are executed inside Azure Databricks using Auto Loader, Delta Lake and Delta Live Tables.

Core responsibilities:

- Incremental streaming ingestion
- Schema normalization
- Standardization
- Deduplication
- Delta persistence
- Layer promotion
---

# Delta Live Tables (DLT)

Delta Live Tables are used to simplify declarative streaming and batch transformations.

Example:

```python
import dlt

@dlt.table
def dim_user_stg():
    return spark.readStream.table(
        "spotify_cata.silver.dimuser"
    )
```

DLT capabilities leveraged by the platform:

- Managed checkpoints
- Pipeline lineage
- Declarative transformations
- Dependency resolution
- Streaming orchestration

---

# Silver Layer

Validated and standardized Delta datasets generated through Auto Loader and DLT transformations.

Responsibilities include:

- Schema enforcement
- Data quality validation
- Deduplication
- Standardization
- Business rule enforcement

---

# Gold Layer

Curated Delta datasets generated from validated Silver transformations and downstream business aggregation logic.

---

# Monitoring & Operational Alerting

Operational observability is integrated directly into the orchestration layer.

If pipeline execution fails:

1. ADF triggers a Web Activity
2. Azure Functions receive the payload
3. Automated notifications are dispatched

Example payload:

```json
{
  "pipeline_name": "@{pipeline().Pipeline}",
  "pipeline_runId": "@{pipeline().RunId}"
}
```

This approach simulates production-style operational monitoring patterns for orchestration reliability.

---

# CI/CD & Deployment Lifecycle

The platform follows a Git-based workflow where orchestration artifacts, notebooks and deployment definitions are version-controlled and promoted across environments.

Deployment lifecycle components:

- GitHub source control
- Databricks Asset Bundles (DABs)
- Bundle deployment through CLI
- Environment promotion (Dev / Staging / Prod)

Deployment example:

```bash
databricks bundle deploy
```

ADF orchestration artifacts are managed through Git integration and publish workflows aligned with Azure deployment practices.

---

# Final Notes

The platform was designed around reusable ingestion patterns, incremental processing and operational reliability using Azure and Databricks technologies.

The architecture combines metadata-driven orchestration, CDC watermark persistence and streaming-style downstream processing through Auto Loader and Delta Live Tables.
