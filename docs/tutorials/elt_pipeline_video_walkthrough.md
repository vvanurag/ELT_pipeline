# ELT Pipeline with dbt, Snowflake and Airflow: Guided Walkthrough

A study guide for the video **"Code along – build an ELT Pipeline in 1 Hour (dbt, Snowflake, Airflow)"** by Jay (jayzern), about 36 minutes.

- Video: <https://www.youtube.com/watch?v=OLXkGB7krGo>
- Code notes: [Notion code-along page](https://bittersweet-mall-f00.notion.site/Code-along-build-an-ELT-Pipeline-in-1-Hour-dbt-Snowflake-Airflow-cffab118a21b40b8acd3d595a4db7c15)

> **Source note:** This guide is based on the video's official chapter list and the code in the Notion notes. It is not based on the spoken transcript. Timestamps refer to the video chapters.

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
    - ELT vs ETL: transform inside the warehouse
    - Why the video uses the TPC-H sample data
2. [Lesson 1: Setting Up Snowflake and dbt](#lesson-1-setting-up-snowflake-and-dbt) (1:35–7:52)
    - Warehouse, database, schema and role
    - Separating compute from storage
    - RBAC grants and least privilege
    - `profiles.yml` vs `dbt_project.yml`
3. [Lesson 2: dbt Project Config and Packages](#lesson-2-dbt-project-config-and-packages) (7:52–11:45)
    - Materializations: view vs table
    - Folder-level config
    - Packages: `dbt_utils` and `dbt deps`
4. [Lesson 3: Sources and Staging Models](#lesson-3-sources-and-staging-models) (11:45–17:41)
    - `source()` vs hard-coded table names
    - What the staging layer is for
    - Grain and surrogate keys
    - Source tests
5. [Lesson 4: Transformed Models and Macros](#lesson-4-transformed-models-and-macros) (17:41–24:23)
    - Layers: intermediate, fact and mart
    - `ref()` and the DAG
    - Aggregating before joining
    - Macros, Jinja and DRY
6. [Lesson 5: Generic and Singular Tests](#lesson-5-generic-and-singular-tests) (24:23–29:35)
    - Tests return bad rows
    - Generic tests in YAML
    - Singular tests in SQL
    - `severity: warn` and `dbt build`
7. [Lesson 6: Deploying with Airflow](#lesson-6-deploying-with-airflow) (29:35–end)
    - Why orchestrate
    - Astro CLI and Docker
    - A separate virtual environment for dbt
    - Cosmos: one Airflow task per model
    - Airflow connections and DAG settings
8. [Key Takeaways](#key-takeaways)
    - [Gotchas in the Notion code](#gotchas-in-the-notion-code)
9. [Homework](#homework)
10. [Self-Check Questions](#self-check-questions)

### Video Chapters

| Time | Chapter |
|---|---|
| 0:00 | Intro |
| 1:35 | Setup dbt + Snowflake |
| 7:52 | Configure dbt_project.yml and packages |
| 11:45 | Create source and staging tables |
| 17:41 | Transformed models (fact tables, data marts) |
| 20:25 | Macro functions |
| 24:23 | Generic and singular tests |
| 29:35 | Deploy models using Airflow |

---

## The Big Picture

Before any code, keep this model in your head:

```
 Snowflake sample data  ──►  dbt (transform inside Snowflake)  ──►  Airflow (runs dbt on a schedule)
   [Extract + Load:            staging → intermediate → fact         [Orchestration]
    already done for you]      + tests + macros
```

- **ELT, not ETL.** The data is loaded into the warehouse *raw*, and the transformation happens *inside* the warehouse using SQL. dbt never moves data. It only sends SQL to Snowflake, and Snowflake does the work.
- The video skips the "E" and "L" by using Snowflake's built-in **TPC-H** sample dataset (`SNOWFLAKE_SAMPLE_DATA.TPCH_SF1`), a fake business with customers, orders and line items. That lets you focus on the "T".

---

## Lesson 1: Setting Up Snowflake and dbt

**Video: 1:35–7:52**

**What happens:** Jay runs a SQL worksheet in Snowflake, then runs `dbt init` in the terminal.

```sql
use role accountadmin;

create warehouse dbt_wh with warehouse_size='x-small';
create database if not exists dbt_db;
create role if not exists dbt_role;

grant role dbt_role to user <your_user>;
grant usage on warehouse dbt_wh to role dbt_role;
grant all on database dbt_db to role dbt_role;

use role dbt_role;
create schema if not exists dbt_db.dbt_schema;
```

### Concepts to learn

| Snowflake object | What it is | Analogy |
|---|---|---|
| **Warehouse** (`dbt_wh`) | **Compute** that runs queries and is billed while running | The engine |
| **Database / schema** (`dbt_db.dbt_schema`) | **Storage**, a container for tables and views | The garage |
| **Role** (`dbt_role`) | A bundle of permissions | A key card |

1. **Compute and storage are separate.** You can resize or pause the warehouse without touching the data. `x-small` is the cheapest size, which is plenty for learning.
2. **RBAC (role-based access control).** Permissions are granted to *roles*, and roles are granted to *users*.
    - `grant usage on warehouse` lets the role run queries on that compute.
    - `grant all on database` lets the role create schemas and tables there.
    - `grant role ... to user` lets you act as that role.
3. **Least privilege.** dbt runs as `dbt_role`, not `accountadmin`. A mistake in dbt then can't drop the whole account.
4. **Two dbt config files people often confuse:**
    - `~/.dbt/profiles.yml` holds **how to connect**: account, user, password, role, warehouse. It stays on your machine and is never committed.
    - `dbt_project.yml` holds **how to build the project**. It lives in the repo and is shared.

**Check yourself**

- **Q1.** If you run `drop warehouse dbt_wh`, is any data lost?
- **Q2.** Why grant permissions to a role instead of directly to the user?

---

## Lesson 2: dbt Project Config and Packages

**Video: 7:52–11:45**

**What happens:** Jay deletes the example models, sets materializations per folder, and adds the `dbt_utils` package.

```yaml
# dbt_project.yml
models:
  snowflake_workshop:
    staging:
      materialized: view
      snowflake_warehouse: dbt_wh
    marts:
      materialized: table
      snowflake_warehouse: dbt_wh
```

```yaml
# packages.yml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
```

Then run `dbt deps`.

> The Notion page labels the first block "dbt_profile.yaml", but it belongs in `dbt_project.yml`. It also leaves out `packages.yml`, which the next lesson needs.

### Concepts to learn

1. **Materialization** is what dbt creates in Snowflake from your `SELECT`.
    - `view` is just saved SQL. It is cheap to build and always fresh, but it recomputes every time someone queries it.
    - `table` stores the results physically. Queries against it are fast, but you have to rebuild it to refresh it.
2. **Why staging uses views and marts use tables.** Staging models are light renames that only feed other models. Marts are what analysts and BI tools query heavily, so they are worth storing.
3. **Folder-level config.** Setting `staging: materialized: view` once applies to every model in that folder. It is the same idea as the DRY macro in Lesson 4, applied to config.
4. **Packages.** `packages.yml` plus `dbt deps` works like `pip install` for dbt. `dbt_utils` is the most common package, and you'll use its `generate_surrogate_key` macro next.

**Check yourself**

- **Q3.** A model is queried 500 times a day by a dashboard. Should it be a view or a table, and why?

---

## Lesson 3: Sources and Staging Models

**Video: 11:45–17:41**

**What happens:** Jay writes `tpch_sources.yml`, `stg_tpch_orders.sql` and `stg_tpch_line_items.sql`.

```yaml
# models/staging/tpch_sources.yml
version: 2

sources:
  - name: tpch
    database: snowflake_sample_data
    schema: tpch_sf1
    tables:
      - name: orders
        columns:
          - name: o_orderkey
            tests:
              - unique
              - not_null
      - name: lineitem
        columns:
          - name: l_orderkey
            tests:
              - relationships:
                  to: source('tpch', 'orders')
                  field: o_orderkey
```

```sql
-- models/staging/stg_tpch_line_items.sql
select
    {{ dbt_utils.generate_surrogate_key(['l_orderkey', 'l_linenumber']) }} as order_item_key,
    l_orderkey as order_key,
    l_partkey as part_key,
    l_linenumber as line_number,
    l_quantity as quantity,
    l_extendedprice as extended_price,
    l_discount as discount_percentage,
    l_tax as tax_rate
from {{ source('tpch', 'lineitem') }}
```

### Concepts to learn

1. **`source()` vs a hard-coded table name.** Writing `{{ source('tpch','orders') }}` instead of `snowflake_sample_data.tpch_sf1.orders` has three benefits:
    - dbt knows the raw input, so it appears in the **lineage graph** (`dbt docs generate`).
    - You can change the database or schema in one YAML file.
    - You can attach **tests** to raw data before you build anything on it.
2. **What the staging layer is for.** It mirrors the source one-to-one and cleans it up: renames columns (`o_custkey` → `customer_key`) and does light casts, with **no joins and no business logic**. Every later model reads from staging and never from the raw tables.
3. **Grain and surrogate keys.** This is the most important modeling idea in the video.
    - *Grain* means "what does one row represent?" In `orders`, one row is one order. In `lineitem`, one row is one item within an order.
    - `l_orderkey` alone is **not unique** in line items, because one order has many items. The unique combination is (`l_orderkey`, `l_linenumber`).
    - `generate_surrogate_key([...])` hashes those columns into a single unique ID, `order_item_key`, which makes joins and tests simple.
4. **Source tests.** `unique` and `not_null` on `o_orderkey` check the primary key. `relationships` checks that every line item points to an order that actually exists, which is a foreign-key check.

**Try it:** run `dbt run -s stg_tpch_orders`, then look for the **view** in Snowflake under `dbt_db.dbt_schema`.

**Check yourself**

- **Q4.** Why would a `unique` test on `l_orderkey` fail?

---

## Lesson 4: Transformed Models and Macros

**Video: 17:41–24:23** (macros start at 20:25)

**What happens:** Jay builds `int_order_items`, writes the `discounted_amount` macro, then builds `int_order_items_summary` and `fct_orders`. The chapter order (models at 17:41, macros at 20:25) suggests he writes the join first and then pulls the discount math out into a macro.

```sql
-- macros/pricing.sql
{% macro discounted_amount(extended_price, discount_percentage, scale=2) %}
    (-1 * {{extended_price}} * {{discount_percentage}})::decimal(16, {{ scale }})
{% endmacro %}
```

```sql
-- models/marts/int_order_items_summary.sql
select
    order_key,
    sum(extended_price) as gross_item_sales_amount,
    sum(item_discount_amount) as item_discount_amount
from {{ ref('int_order_items') }}
group by order_key
```

### Concepts to learn

1. **The layered flow:**

    ```
    stg_tpch_orders ─────┬──────────────────────────────────────────────┐
                         ├─► int_order_items ─► int_order_items_summary ─┴─► fct_orders
    stg_tpch_line_items ─┘   (grain: item)      (grain: order)               (grain: order)
    ```

    - **Intermediate** models do joins and calculations, and they can change the grain. Here, the summary goes from item level to order level with `group by order_key`.
    - **Fact table** (`fct_`): one row per business event (an order) with numeric *measures* (sales, discount) and *keys* to dimensions (`customer_key`).
    - A **data mart** is the set of final, business-ready tables for one area, such as sales.

2. **`ref()` builds the DAG.** `{{ ref('stg_tpch_orders') }}` resolves to the real table name *and* tells dbt the dependency. You never set the run order yourself. dbt works it out, and this is what lets Airflow draw the graph in Lesson 6.
3. **Why aggregate before joining.** Joining orders directly to line items gives many rows per order. Summarizing to one row per order first means `fct_orders` keeps the same grain as `orders`, so the numbers aren't inflated.
4. **Macros, Jinja and DRY.** A macro is a reusable SQL snippet with parameters. If the discount formula appears in five models and the business changes it, you edit one file. Open `target/compiled/...` to see the plain SQL that dbt actually sends to Snowflake. This is the best way to understand Jinja.
5. **A detail to notice:** the macro returns a **negative** number (`-1 * price * discount`). That is a deliberate convention, discounts as negative money, and Lesson 5 tests it.

**Check yourself**

- **Q5.** What would go wrong if `fct_orders` joined `stg_tpch_line_items` directly instead of the summary?
- **Q6.** You never told dbt to build `int_order_items` before `fct_orders`. How did it know?

---

## Lesson 5: Generic and Singular Tests

**Video: 24:23–29:35**

**What happens:** Jay writes `generic_tests.yml` plus two SQL test files, then runs `dbt test`.

```yaml
# models/marts/generic_tests.yml
models:
  - name: fct_orders
    columns:
      - name: order_key
        tests:
          - unique
          - not_null
          - relationships:
              to: ref('stg_tpch_orders')
              field: order_key
              severity: warn
      - name: status_code
        tests:
          - accepted_values:
              values: ['P', 'O', 'F']
```

```sql
-- tests/fct_orders_date_valid.sql
select *
from {{ ref('fct_orders') }}
where date(order_date) > current_date()
   or date(order_date) < date('1990-01-01')
```

### Concepts to learn

1. **How every dbt test works:** a test is a query that returns the **bad rows**. Zero rows means pass, and any rows means fail.
2. **Generic tests** are reusable and declared in YAML: `unique`, `not_null`, `relationships`, `accepted_values`. The `accepted_values: ['P','O','F']` test catches new or unexpected status codes coming from upstream.
3. **Singular tests** are one-off SQL files in `tests/` for business rules. In this project:
    - `fct_orders_discount.sql`: discounts must not be positive (this is why the macro returns negative values).
    - `fct_orders_date_valid.sql`: order dates must not be in the future or before 1990.
4. **`severity: warn`** reports the problem without failing the run. Use it for checks that are worth knowing about but shouldn't block the pipeline.
5. **`dbt build`** runs models and tests in DAG order, so a failing test stops the models downstream of it. That is what you want in production.

**Check yourself**

- **Q7.** Write a singular test asserting that `gross_item_sales_amount` is never negative.

---

## Lesson 6: Deploying with Airflow

**Video: 29:35–end**

**What happens:** Jay creates an Astronomer project, installs dbt in a virtual environment inside the Docker image, adds a Snowflake connection in the Airflow UI, writes `dbt_dag.py`, and triggers it.

```bash
brew install astro
mkdir dbt-dag && cd dbt-dag
astro dev init
# copy the dbt project to dags/dbt/data_pipeline/
astro dev start   # UI at http://localhost:8080 (admin / admin)
```

```dockerfile
# Dockerfile (added line)
RUN python -m venv dbt_venv && source dbt_venv/bin/activate && \
    pip install --no-cache-dir dbt-snowflake && deactivate
```

```text
# requirements.txt
astronomer-cosmos
apache-airflow-providers-snowflake
```

```python
# dags/dbt_dag.py
import os
from datetime import datetime

from cosmos import DbtDag, ProjectConfig, ProfileConfig, ExecutionConfig
from cosmos.profiles import SnowflakeUserPasswordProfileMapping

profile_config = ProfileConfig(
    profile_name="default",
    target_name="dev",
    profile_mapping=SnowflakeUserPasswordProfileMapping(
        conn_id="snowflake_conn",
        profile_args={"database": "dbt_db", "schema": "dbt_schema"},
    ),
)

dbt_snowflake_dag = DbtDag(
    project_config=ProjectConfig("/usr/local/airflow/dags/dbt/data_pipeline"),
    operator_args={"install_deps": True},
    profile_config=profile_config,
    execution_config=ExecutionConfig(
        dbt_executable_path=f"{os.environ['AIRFLOW_HOME']}/dbt_venv/bin/dbt",
    ),
    schedule_interval="@daily",   # use schedule="@daily" on Airflow 2.4+ / 3
    start_date=datetime(2023, 9, 10),
    catchup=False,
    dag_id="dbt_dag",
)
```

### Concepts to learn

1. **Why orchestrate?** `dbt run` on your laptop isn't a pipeline. Airflow adds scheduling (`@daily`), retries, alerting, logs, and a place to put dbt alongside ingestion jobs.
2. **Astro CLI.** `astro dev init` and `astro dev start` run a full Airflow setup locally in Docker (scheduler, webserver and database).
3. **A separate virtual environment for dbt** (the Dockerfile step). Airflow and dbt pin conflicting Python packages. Installing dbt in `dbt_venv` keeps them apart, and Cosmos calls dbt at that path (`ExecutionConfig`).
4. **Cosmos** is the main idea here. Without it, you'd have one Airflow task running `dbt build`, which is all or nothing. Cosmos reads your dbt project and creates **one Airflow task per model, with its tests**, in the same DAG that `ref()` defined. You can see which model failed and retry just that one.
5. **Credentials from an Airflow connection.** `SnowflakeUserPasswordProfileMapping(conn_id="snowflake_conn")` builds `profiles.yml` at runtime from the Airflow connection (**Admin → Connections**, type Snowflake). Secrets stay in Airflow, not in your code.
6. **DAG settings:**
    - `start_date` is when the schedule begins.
    - `catchup=False` stops Airflow from back-filling a run for every day since 2023.
    - `install_deps=True` runs `dbt deps` automatically.

**Check yourself**

- **Q8.** What does Cosmos give you that a single `BashOperator` running `dbt build` does not?

---

## Key Takeaways

1. **ELT:** load raw data, then transform it in the warehouse with SQL.
2. **Snowflake:** compute (warehouses) is separate from storage, and access is granted through roles with least privilege.
3. **Layers:** source → staging (clean, 1:1) → intermediate (join, aggregate) → fact/mart (business-ready).
4. **Grain:** always know what one row means, and use surrogate keys when no single column is unique.
5. **dbt basics:** `source()`, `ref()` and the DAG, materializations, macros and Jinja, packages.
6. **Testing:** tests return bad rows. Generic tests go in YAML, singular tests go in SQL, and severity controls whether a failure blocks the run.
7. **Orchestration:** Airflow plus Cosmos turns the dbt DAG into observable, retryable tasks.

### Gotchas in the Notion code

- `stg_tpch_line_items.sql` is listed under `models/staging/tpch/`, but the other staging files are directly in `models/staging/`. dbt finds models in either place, so pick one location and use it everywhere.
- Replace the hard-coded `grant role dbt_role to user jayzern;` with your own username.
- `schedule_interval` is deprecated in Airflow 2.4+ and removed in Airflow 3, so use `schedule=` there.
- `packages.yml` is missing from the notes. Add it, otherwise `generate_surrogate_key` fails.

---

## Homework

1. Add a **dimension** model, `dim_customers`, built from `tpch.customer`, and join it to `fct_orders` on `customer_key`.
2. Add a `net_item_sales_amount` column (gross + discount) using a new macro.
3. Add **source freshness** checks to `tpch_sources.yml`, and find out why they don't make sense on sample data.
4. Make one test fail on purpose, then watch how Cosmos shows the failure in the Airflow graph.

---

## Self-Check Questions

| # | Lesson | Question |
|---|---|---|
| Q1 | 1 | If you run `drop warehouse dbt_wh`, is any data lost? |
| Q2 | 1 | Why grant permissions to a role instead of directly to the user? |
| Q3 | 2 | A model is queried 500 times a day by a dashboard. Should it be a view or a table, and why? |
| Q4 | 3 | Why would a `unique` test on `l_orderkey` fail? |
| Q5 | 4 | What would go wrong if `fct_orders` joined `stg_tpch_line_items` directly instead of the summary? |
| Q6 | 4 | You never told dbt to build `int_order_items` before `fct_orders`. How did it know? |
| Q7 | 5 | Write a singular test asserting that `gross_item_sales_amount` is never negative. |
| Q8 | 6 | What does Cosmos give you that a single `BashOperator` running `dbt build` does not? |
