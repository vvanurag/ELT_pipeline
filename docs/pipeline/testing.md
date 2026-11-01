# Data Quality Testing & Macros

Data reliability is maintained through automated generic tests, singular tests, and reusable Jinja macros.

---

## 🧪 Testing Types in dbt

### 1. Generic Schema Tests (YAML)
Defined directly in `models/**/*.yml`:
- `unique`: Ensures no duplicate primary keys exist.
- `not_null`: Asserts that critical columns have non-null values.
- `accepted_values`: Validates status codes against allowed values (e.g., `['O', 'F', 'P']`).
- `relationships`: Enforces foreign key referential integrity between fact and dimension tables.

### 2. Singular Tests (Custom SQL)
Written as standalone SQL queries in the `tests/` directory. If the query returns **zero rows**, the test passes. If rows are returned, the test fails.

---

## ⚙️ Jinja Macros
Macros are reusable functions written in Jinja/SQL:
- Money and pricing calculations (e.g. discounting and tax functions).
- Date standardizations and timezone conversions.
