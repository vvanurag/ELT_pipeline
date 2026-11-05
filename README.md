# Production-Grade ELT Pipeline (Snowflake + dbt + Airflow)

[![Python 3.11](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/)
[![dbt-core](https://img.shields.io/badge/dbt--core-1.13-orange.svg)](https://www.getdbt.com/)
[![Snowflake](https://img.shields.io/badge/Snowflake-Data%20Cloud-29B5E8.svg)](https://www.snowflake.com/)
[![Apache Airflow](https://img.shields.io/badge/Airflow-2.x%20%2F%20Astro-017CEE.svg)](https://airflow.apache.org/)
[![OrbStack](https://img.shields.io/badge/Docker-OrbStack-green.svg)](https://orbstack.dev/)
[![Documentation](https://img.shields.io/badge/Docs-MkDocs%20Material-894cfa.svg)](https://vvanurag.github.io/ELT_pipeline/)

A comprehensive, production-ready **Extract, Load, Transform (ELT)** data pipeline demonstrating modern data engineering architecture, role-based access control (RBAC), dimensional modeling, data quality testing, and scheduled orchestration.

---

## Architecture Overview

```mermaid
flowchart LR
    subgraph S1["1. Raw Data Layer (Snowflake)"]
        direction TB
        TPCH["SNOWFLAKE_SAMPLE_DATA<br/>(TPC-H SF1)"]
        SRC_CUST["CUSTOMER"]
        SRC_ORD["ORDERS"]
        SRC_LINE["LINEITEM"]
        TPCH --> SRC_CUST & SRC_ORD & SRC_LINE
    end

    subgraph S2["2. Transformation Layer (dbt Core)"]
        direction TB
        subgraph STG["Staging Models (Views)"]
            STG_C["stg_tpch_customers"]
            STG_O["stg_tpch_orders"]
            STG_L["stg_tpch_line_items"]
        end
        subgraph MARTS["Marts & Facts (Tables)"]
            FCT_ORD["fct_orders"]
            DIM_CUST["dim_customers"]
            REP_SALES["monthly_sales_summary"]
        end
        subgraph QA["Testing & Macros"]
            TESTS["Schema Tests & Singular SQL Tests"]
            MACROS["Jinja Reusable Calculations"]
        end
        STG --> MARTS
        MARTS -.-> QA
    end

    subgraph S3["3. Orchestration Layer (Apache Airflow)"]
        direction TB
        ASTRO["Astronomer CLI / Cosmos"]
        DAG["dbt_snowflake_dag"]
        SCHED["Cron Schedule & Alerts"]
        ASTRO --> DAG --> SCHED
    end

    SRC_CUST & SRC_ORD & SRC_LINE --> STG
    S3 -. triggers & monitors .-> S2
```

---

## Tech Stack & Tooling

| Component | Tool / Platform | Purpose |
| :--- | :--- | :--- |
| **Data Warehouse** | [Snowflake](https://www.snowflake.com/) | Cloud storage, virtual compute (`COMPUTE_WH`), and secure RBAC (`TRANSFORM_ROLE`). |
| **Transformations** | [dbt Core](https://docs.getdbt.com/) (`dbt-snowflake`) | Modular SQL data models, lineage tracking, Jinja macros, and schema validations. |
| **Orchestration** | [Apache Airflow](https://airflow.apache.org/) / [Astronomer Cosmos](https://astronomer.github.io/astronomer-cosmos/) | Automatic DAG rendering from dbt project, task dependency scheduling, and monitoring. |
| **Container Engine** | [OrbStack](https://orbstack.dev/) | Fast, lightweight macOS container runtime for local Docker & Airflow execution. |
| **Documentation** | [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) | Interactive static documentation site published via GitHub Pages. |

---

## Project Structure

```text
ELT_pipeline/
├── README.md                      # Project overview & architectural guide
├── mkdocs.yml                     # Documentation configuration (GitHub Pages)
├── docs/                          # Comprehensive Markdown learning & architecture guides
│   ├── index.md                   # Home & executive summary
│   ├── setup/
│   │   ├── snowflake.md           # Snowflake RBAC & DWH setup SQL
│   │   └── local_environment.md   # OrbStack, Conda & dbt setup
│   ├── concepts/
│   │   ├── elt_vs_etl.md          # Architectural patterns
│   │   └── dimensional_modeling.md# Kimball methodology, Facts & Dimensions
│   └── orchestration/
│       └── airflow_cosmos.md      # Astro & Cosmos configuration
├── dbt_project/                   # dbt transformation models
│   ├── dbt_project.yml
│   ├── packages.yml
│   ├── models/
│   │   ├── staging/
│   │   └── marts/
│   ├── macros/
│   └── tests/
└── airflow/                       # Airflow orchestration project (Docker/Astro)
    ├── Dockerfile
    └── dags/
```

---

## Quickstart

### 1. Activate Local Python Environment
```bash
conda activate elt_env
dbt --version
```

### 2. Verify Snowflake Connection
```bash
cd dbt_project
dbt debug
```

### 3. Run Pipeline Transformations & Tests
```bash
dbt run
dbt test
```

### 4. Run Orchestration with Docker & Astro CLI
```bash
cd airflow
astro dev start
```
Airflow UI will be available at `http://localhost:8080`.

