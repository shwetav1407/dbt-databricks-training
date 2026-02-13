# Jaffle Shop - dbt on Databricks

A sandbox dbt project based on the [dbt Labs Jaffle Shop](https://github.com/dbt-labs/jaffle-shop), adapted for **Databricks** using the `dbt-databricks` adapter. This project demonstrates core dbt concepts through a fictional sandwich shop's data.

---

## Project Overview

The Jaffle Shop is a fictional restaurant that processes customer orders for jaffles (toasted sandwiches) and beverages. The project showcases:

- **Staging models** - Light transformations on raw source data (views)
- **Mart models** - Business-ready tables with aggregations and joins
- **Data testing** - Schema tests, expression tests, and relationship tests
- **Macros** - Reusable SQL transformations (e.g. `cents_to_dollars`)
- **Seeds** - Sample data loaded via CSV files

### DAG Overview

```
Sources (raw)          Staging (views)         Marts (tables)
--------------         ---------------         --------------
raw_customers    -->   stg_customers     -->   customers
raw_orders       -->   stg_orders        -->   orders
raw_items        -->   stg_order_items   -->   order_items
raw_products     -->   stg_products      -->   products
raw_stores       -->   stg_locations     -->   locations
raw_supplies     -->   stg_supplies      -->   supplies
```

---

## Prerequisites

- Python 3.9+
- A Databricks workspace with a SQL Warehouse
- A Unity Catalog catalog and schema to build into

---

## Setup

### 1. Create a virtual environment

```bash
cd jaffle_shop/
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Configure environment variables

```bash
# Copy the template and fill in your values
cp .env.template .env.local

# Edit .env.local with your Databricks connection details
```

### 3. Create profiles.yml

Create a `profiles.yml` in this directory (it's gitignored):

```yaml
jaffle_shop:
  target: dev
  outputs:
    dev:
      type: databricks
      host: "{{ env_var('DATABRICKS_HOST') }}"
      http_path: "{{ env_var('DATABRICKS_HTTP_PATH') }}"
      token: "{{ env_var('DATABRICKS_TOKEN') }}"
      catalog: "{{ env_var('DBT_CATALOG') }}"
      schema: "{{ env_var('DBT_SCHEMA') }}"
      threads: 4
```

### 4. Install dbt packages

```bash
dbt deps
```

### 5. Verify connection

```bash
dbt debug
```

### 6. Load seed data

```bash
dbt seed --full-refresh --vars '{"load_source_data": true}'
```

### 7. Build the project

```bash
dbt build
```

---

## Project Structure

```
jaffle_shop/
  dbt_project.yml           # Project configuration
  packages.yml              # dbt package dependencies (dbt_utils, dbt_date)
  requirements.txt          # Python dependencies (dbt-databricks)
  .env.template             # Template for required environment variables
  models/
    staging/                # 1:1 with source tables, materialised as views
      __sources.yml         # Source definitions (ecom schema)
      stg_customers.sql
      stg_orders.sql
      stg_order_items.sql
      stg_products.sql
      stg_locations.sql
      stg_supplies.sql
    marts/                  # Business-ready tables
      customers.sql         # Customer lifetime metrics
      orders.sql            # Order details with food/drink breakdown
      order_items.sql       # Order items with product and supply data
      products.sql          # Product sales metrics
      locations.sql         # Location performance metrics
      supplies.sql          # Supply cost data
      metricflow_time_spine.sql
  macros/
    cents_to_dollars.sql    # Convert cents to dollars (Databricks-aware)
    generate_schema_name.sql
  seeds/
    jaffle-data/            # Raw CSV data (customers, orders, items, etc.)
  analyses/
  data-tests/
  snapshots/
```

---

## Owner

**Ash Sultan** (ash.sultan@databricks.com)
