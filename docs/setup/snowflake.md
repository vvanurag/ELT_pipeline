# Snowflake Role-Based Access Control (RBAC) & DWH Setup

In this section, we set up an isolated, secure environment inside Snowflake adhering to the **Principle of Least Privilege**.

---

## 🔐 Architecture Principles
1. **Never use `ACCOUNTADMIN` for automated pipelines**: Administrative roles have unrestricted access and can cause accidental data loss or security breaches.
2. **Dedicated Virtual Warehouse**: Isolated compute prevents long-running data transformations from blocking ad-hoc analytics or BI queries.
3. **Auto-Suspend & Auto-Resume**: Set warehouse auto-suspend to 60 seconds to minimize credit consumption during idle periods.
4. **Imported Privileges on Sample Data**: dbt requires read-only permissions on the raw source dataset (`SNOWFLAKE_SAMPLE_DATA`).

---

## 🛠️ Snowflake Worksheet Setup Script

Open your Snowflake Web Console, navigate to **Worksheets**, and execute the following SQL script as `ACCOUNTADMIN`:

```sql
-- 1. Use high-privileged role to provision infrastructure
USE ROLE ACCOUNTADMIN;

-- 2. Create dedicated virtual warehouse for dbt transformations
CREATE WAREHOUSE IF NOT EXISTS TRANSFORM_WH
    WITH 
    WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    INITIALLY_SUSPENDED = TRUE
    COMMENT = 'Virtual warehouse for dbt and Airflow transformations';

-- 3. Create target transformation database
CREATE DATABASE IF NOT EXISTS DEMO_DWH
    COMMENT = 'Destination analytics warehouse for dbt models';

-- 4. Create development schemas
CREATE SCHEMA IF NOT EXISTS DEMO_DWH.DEV_STAGING;
CREATE SCHEMA IF NOT EXISTS DEMO_DWH.DEV_MARTS;

-- 5. Create transformation role
CREATE ROLE IF NOT EXISTS DBT_ROLE
    COMMENT = 'Service role for dbt and Airflow ELT pipeline';

-- 6. Grant warehouse permissions
GRANT USAGE, OPERATE ON WAREHOUSE TRANSFORM_WH TO ROLE DBT_ROLE;

-- 7. Grant database and schema privileges
GRANT USAGE, CREATE SCHEMA ON DATABASE DEMO_DWH TO ROLE DBT_ROLE;
GRANT ALL PRIVILEGES ON ALL SCHEMAS IN DATABASE DEMO_DWH TO ROLE DBT_ROLE;
GRANT ALL PRIVILEGES ON FUTURE SCHEMAS IN DATABASE DEMO_DWH TO ROLE DBT_ROLE;
GRANT ALL PRIVILEGES ON ALL TABLES IN DATABASE DEMO_DWH TO ROLE DBT_ROLE;
GRANT ALL PRIVILEGES ON FUTURE TABLES IN DATABASE DEMO_DWH TO ROLE DBT_ROLE;
GRANT ALL PRIVILEGES ON ALL VIEWS IN DATABASE DEMO_DWH TO ROLE DBT_ROLE;
GRANT ALL PRIVILEGES ON FUTURE VIEWS IN DATABASE DEMO_DWH TO ROLE DBT_ROLE;

-- 8. Grant read-only access to TPC-H sample data
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE_SAMPLE_DATA TO ROLE DBT_ROLE;

-- 9. Create dedicated service user for dbt / Airflow
-- NOTE: Replace 'YourStrongPassword123!' with your chosen password
CREATE USER IF NOT EXISTS DBT_USER
    PASSWORD = 'YourStrongPassword123!'
    DEFAULT_ROLE = 'DBT_ROLE'
    DEFAULT_WAREHOUSE = 'TRANSFORM_WH'
    DEFAULT_NAMESPACE = 'DEMO_DWH.DEV_STAGING'
    MUST_CHANGE_PASSWORD = FALSE
    COMMENT = 'Service account for dbt and Airflow automation';

-- 10. Assign role to user
GRANT ROLE DBT_ROLE TO USER DBT_USER;

-- 11. Grant role to your current login user and sysadmin for UI visibility
GRANT ROLE DBT_ROLE TO ROLE SYSADMIN;
```

---

## 📋 Connection Parameters Reference

| Parameter | Value | Description |
| :--- | :--- | :--- |
| **Account Identifier** | `LSMPZXO-NZ29098` | Your unique Snowflake account locator |
| **User** | `DBT_USER` | Dedicated transformation user |
| **Role** | `DBT_ROLE` | Least-privilege transformation role |
| **Warehouse** | `TRANSFORM_WH` | Size: X-Small with 60s auto-suspend |
| **Database** | `DEMO_DWH` | Primary database for transformed data |
| **Schema** | `DEV_STAGING` | Default staging landing schema |
