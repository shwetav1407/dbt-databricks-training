# Projects

This document provides an overview of all projects hosted in the dbt & Databricks training repository. Each project is self-contained with its own documentation, dependencies, and setup instructions.

---

<!-- 
## Example Project Entry

### Project Title

**Owner:** Your Name ([@github-handle](https://github.com/github-handle))

Brief description of the project (2-3 sentences). What does it demonstrate? Who is the target customer/industry?

**Key Capabilities:**
- Capability 1
- Capability 2
- Capability 3

**Technologies:**
- dbt Core
- Databricks (Unity Catalog, SQL Warehouse)
- Lakeflow Jobs

**Location:** [`project_folder/`](project_folder/)  
**Full Documentation:** [View README](project_folder/README.md)

---
-->

## Jaffle Shop

**Owner:** Ash Sultan ([@ash-sultan_data](https://github.com/ash-sultan_data))

A sandbox dbt project based on the [dbt Labs Jaffle Shop](https://github.com/dbt-labs/jaffle-shop), adapted for Databricks using the `dbt-databricks` adapter. Demonstrates core dbt concepts -- staging/mart layers, data testing, macros, and seeds -- through a fictional sandwich shop's data.

**Key Capabilities:**
- 6 staging models (views) with source definitions and schema tests
- 7 mart models (tables) including customers, orders, products, and locations
- 33 data tests covering uniqueness, not-null, relationships, and expression validations
- Seed data: 935 customers, 61K orders, 90K order items, 10 products, 6 stores
- Databricks-specific macro dispatch (`cents_to_dollars`)

**Technologies:**
- dbt Core with `dbt-databricks` adapter
- Databricks Unity Catalog (`dbt_databricks_training_internal`)
- SQL Warehouse (Shared endpoint)

**Location:** [`jaffle_shop/`](jaffle_shop/)  
**Full Documentation:** [View README](jaffle_shop/README.md)

---

## Contributing a New Project

1. **Create your project folder** at the root level (see [README.md](README.md) for guidelines)
2. **Add a comprehensive README** inside your project folder
3. **Test your project** thoroughly (`dbt build` should pass)
4. **Create a feature branch** and submit a Pull Request
5. **Update this file** by adding a new section for your project
6. **Get it reviewed** by at least one team member

---

*Last Updated: February 2026*
