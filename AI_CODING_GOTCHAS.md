# AI-Assisted Coding - Gotchas for dbt & Data Engineering

When using AI coding assistants (Cursor, GitHub Copilot, etc.) for dbt and data engineering work, be aware of these common pitfalls.

## Branch Management Issues

### 1. Wrong Branch Creation

- AI tools may create branches without checking you're starting from `main`
- **Always verify:** Run `git branch` to confirm you're on `main` before creating a feature branch
- **Manual fix:** If you created a branch from the wrong base:
  ```bash
  git checkout feature/your-work
  git rebase --onto main <wrong-base-branch> feature/your-work
  ```

### 2. Stale Base Branch

- AI may create a branch when your local `main` is outdated
- **Always verify:** Pull latest `main` before branching: `git pull origin main`

### 3. Branch Name Inconsistencies

- AI might suggest branch names that don't follow our convention
- **Always verify:** Check branch name matches `feature/` or `bugfix/` prefix and is lowercase with hyphens

---

## dbt-Specific Gotchas

### 4. Hardcoded Table References

- **Issue:** AI generates SQL with hardcoded table names instead of `ref()` or `source()`
- **Always verify:** Every table reference uses `{{ ref('model_name') }}` or `{{ source('source_name', 'table_name') }}`
- ```sql
  -- BAD
  SELECT * FROM catalog.schema.customers

  -- GOOD
  SELECT * FROM {{ ref('stg_crm__customers') }}
  ```

### 5. Wrong Materialisation

- **Issue:** AI defaults to `table` materialisation for everything
- **Always verify:** Staging models should be `view`, intermediate should be `ephemeral` or `view`, only marts should be `table` or `incremental`
- ```sql
  -- Check the config block at the top of each model
  {{ config(materialized='view') }}  -- for staging
  {{ config(materialized='table') }} -- for marts
  ```

### 6. Missing or Incorrect Schema YAML Files

- **Issue:** AI generates models without corresponding schema YAML files, or generates YAML with incorrect syntax
- **Always verify:**
  - Every model has a corresponding entry in a `_schema.yml` or `schema.yml` file
  - YAML indentation is correct (2 spaces, no tabs)
  - Column descriptions and tests are present
  - Primary key columns have `unique` and `not_null` tests

### 7. Incorrect Jinja Syntax

- **Issue:** AI may generate invalid Jinja2 (e.g. wrong bracket types, missing endfor/endif)
- **Common mistakes:**
  - Using `{% ref('model') %}` instead of `{{ ref('model') }}`
  - Missing `{% endfor %}` or `{% endif %}`
  - Incorrect whitespace control (`{%-` vs `{%`)
- **Always verify:** Run `dbt compile` to check Jinja renders correctly before running

### 8. Ignoring Incremental Model Logic

- **Issue:** AI generates incremental models without proper `is_incremental()` logic
- **Always verify:** Incremental models include:
  ```sql
  {{ config(materialized='incremental', unique_key='id') }}

  SELECT ...
  FROM {{ source('source_name', 'table_name') }}
  {% if is_incremental() %}
    WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
  {% endif %}
  ```

### 9. Source Freshness and Data Quality

- **Issue:** AI skips adding `freshness` checks in source definitions or forgets `loaded_at_field`
- **Always verify:** Sources include freshness configuration where appropriate:
  ```yaml
  sources:
    - name: crm
      freshness:
        warn_after: {count: 24, period: hour}
        error_after: {count: 48, period: hour}
      loaded_at_field: _loaded_at
  ```

---

## Databricks-Specific Gotchas

### 10. Unity Catalog Namespace

- **Issue:** AI may use two-level namespace (`schema.table`) instead of three-level (`catalog.schema.table`)
- **Always verify:** `dbt_project.yml` and `profiles.yml` correctly specify catalog and schema
- Use variables for environment-specific values:
  ```yaml
  # dbt_project.yml
  vars:
    catalog_name: "{{ env_var('DBT_CATALOG', 'dev_catalog') }}"
  ```

### 11. Databricks SQL Dialect Differences

- **Issue:** AI generates SQL for Snowflake/BigQuery/Postgres syntax instead of Databricks SQL
- **Common mistakes:**
  - Using `ILIKE` (not supported in Databricks SQL - use `LOWER()` + `LIKE`)
  - Using `::` for casting instead of `CAST()`
  - Using `QUALIFY` incorrectly
  - Missing Delta-specific features like `MERGE INTO`
- **Always verify:** Run `dbt compile` and review the compiled SQL in `target/compiled/`

### 12. DAB Configuration Errors

- **Issue:** AI generates invalid `databricks.yml` for Asset Bundles
- **Always verify:**
  - YAML syntax is valid
  - Resource references are correct
  - Variables use the correct substitution syntax: `${var.name}`
  - Test with `databricks bundle validate` before deploying

### 13. Lakeflow Job Dependencies

- **Issue:** AI may not set up task dependencies correctly in job definitions
- **Always verify:**
  - Tasks run in the correct order (dbt deps before dbt run, dbt run before dbt test)
  - Retry policies are set appropriately
  - Timeouts are reasonable
  - Cluster/warehouse configuration is correct

---

## General Code Quality

### 14. Secrets and Credentials

- **Issue:** AI generates code with placeholder credentials that look real
- **Critical checks:**
  - No `dapi...` tokens anywhere in code
  - `profiles.yml` is gitignored
  - Environment variables used for all credentials
  - See [ACCESS_TOKEN_HANDLING.md](ACCESS_TOKEN_HANDLING.md)

### 15. Missing Documentation

- **Issue:** AI-generated models lack descriptions
- **Always verify:**
  - Model descriptions in schema YAML
  - Column descriptions for key fields
  - README in project folder with setup instructions

### 16. Overly Complex SQL

- **Issue:** AI generates overly complex SQL when simpler approaches exist
- **Best practices:**
  - Break complex transformations into multiple models (staging -> intermediate -> mart)
  - Use CTEs for readability
  - Keep individual models focused on a single transformation concern
  - If a model exceeds ~100 lines, consider splitting it

---

## Pre-PR Checklist (AI Coding Specific)

Before creating your PR, manually verify:

```bash
# 1. Check you're on the correct branch
git branch

# 2. Check all changes are intentional
git status
git diff main...HEAD

# 3. Verify no secrets or temporary files
git diff main...HEAD | grep -i "dapi\|password\|token\|key\|secret"

# 4. Compile and run dbt
cd your_project/
dbt compile    # Check Jinja renders correctly
dbt build      # Run models and tests

# 5. Review compiled SQL
ls target/compiled/  # Spot-check generated SQL for correctness

# 6. Ensure branch is up to date
git fetch origin
git log HEAD..origin/main --oneline
```

---

**Remember:** AI is a powerful assistant, but YOU are responsible for the code quality, SQL correctness, and Git hygiene. Always review, compile, and test before pushing!
