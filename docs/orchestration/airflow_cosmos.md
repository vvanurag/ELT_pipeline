# Airflow & Astronomer Cosmos Architecture

This document describes how Apache Airflow orchestrates dbt models using Astronomer Cosmos.

---

## 🚀 Why Astronomer Cosmos?
Traditionally, running dbt in Airflow involved either:
1. Running a monolithic `BashOperator(bash_command='dbt run')`, which hides model-level lineage and causes entire pipeline reruns on single failures.
2. Manually maintaining hundreds of individual Python operators.

**Astronomer Cosmos** parses the dbt manifest dynamically and converts each dbt model into an individual Airflow task, giving you:
* Native DAG visualization matching your dbt lineage graph.
* Granular task retries on failure.
* Direct logs per model inside the Airflow UI.

