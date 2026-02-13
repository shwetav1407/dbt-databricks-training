# EMEA dbt SME Working Group - dbt & Databricks Training Repository

A centralized repository for dbt projects, Lakeflow pipelines, and related data engineering work on Databricks. Managed by the EMEA dbt SME Working Group (WG), this repo is purpose-built for data transformation and orchestration -- keeping things focused on SQL, dbt, and the Databricks Lakehouse.

---

## Working Group Information

**Slack Channel:** [#snowmelt-dbt-wg](https://databricks.slack.com/channels/snowmelt-dbt-wg)

### Repository Admin

**Ash Sultan** - WG Lead & Repository Admin  
- Email: ash.sultan@databricks.com
- GitHub: [@ash-sultan_data](https://github.com/ash-sultan_data)

### Getting Access

Access to this repository is managed via **Opal**:

1. Search for **`dbt-databricks-training-internal`** in [Opal](https://app.opal.dev/)
2. Request the permission level you need:
   - **READ** -- view the codebase
   - **WRITE** -- contribute code (create branches, push commits, raise PRs)
   - **MAINTAIN** -- admin-level access (manage settings, approve PRs)
3. Approval will be routed to the repository admin (Ash Sultan)

Also join the [#snowmelt-dbt-wg](https://databricks.slack.com/channels/snowmelt-dbt-wg) Slack channel for collaboration and updates.

---

## Databricks Workspace

| Setting | Value |
|---------|-------|
| **Workspace** | [e2-demo-field-eng](https://e2-demo-field-eng.cloud.databricks.com) (AWS) |
| **Catalog** | `dbt_databricks_training_internal` |
| **SQL Warehouse** | Shared endpoint (`8baced1ff014912d`) |
| **CLI Profile** | `aws-demo-field-eng` |

---

## Technology Stack

| Technology | Purpose |
|------------|---------|
| **dbt Core** | SQL-first data transformations |
| **Databricks** | Lakehouse platform (Unity Catalog, SQL Warehouses) |
| **Lakeflow Jobs** | Orchestrating dbt runs and pipeline tasks |
| **Databricks Asset Bundles (DABs)** | Deployment and infrastructure-as-code |
| **Spark Declarative Pipelines (SDP)** | Declarative ETL where applicable alongside dbt |
| **Python** | Utility scripts, custom dbt tests, and macros |

---

## Project Structure

Each project lives in its own directory at the root level. A typical dbt project follows this structure:

```
project_name/
  dbt_project.yml           # Project configuration
  packages.yml              # dbt package dependencies
  databricks.yml            # DAB bundle definition (if using DABs)
  models/
    staging/                # 1:1 with source tables, light transformations
      _sources.yml          # Source definitions
      stg_<source>__<table>.sql
    intermediate/           # Business logic transformations
      int_<description>.sql
    marts/                  # Final business-ready tables/views
      fct_<description>.sql # Fact tables
      dim_<description>.sql # Dimension tables
  macros/                   # Reusable SQL/Jinja macros
  tests/                    # Custom data tests
  seeds/                    # Static reference data (CSV)
  snapshots/                # SCD Type 2 tracking
  analyses/                 # Ad-hoc analytical queries
```

### Running Projects

**Important:** Each project is self-contained. Always navigate into the specific project directory before running commands.

```bash
# Navigate into your project
cd your_project_folder/

# Install dbt dependencies
dbt deps

# Test connection
dbt debug

# Run models
dbt run

# Run tests
dbt test

# Build everything (run + test)
dbt build
```

**Always refer to the README.md inside each project folder for specific setup and configuration instructions.**

---

## Contribution Guidelines

### Branch Strategy

- Always create a new branch from `main` for your work
- Never commit directly to `main`

**Branch Naming Convention:**
- `feature/<jira-ticket>-<description>` for new features or projects
- `bugfix/<jira-ticket>-<description>` for bug fixes
- All lowercase, hyphens for word separators
- Examples:
  - `feature/feip-2886-initial-setup`
  - `bugfix/feip-3001-fix-incremental-model`

### Creating a New Project

1. **Create a new folder** at the root level with a descriptive name (lowercase, underscores)
2. **Initialise dbt** with `dbt init` or set up manually following the structure above
3. **Add a README.md** inside your folder with:
   - Project title and description
   - Owner information
   - Target customer/industry (if applicable)
   - Setup instructions (required env vars, profiles config)
   - Model documentation
4. **Update [PROJECTS.md](PROJECTS.md)** with your project entry before creating a PR

### Pull Request Process

1. **Create PR:** When your work is complete, raise a Pull Request on GitHub
2. **Add Reviewers:** Add **at least ONE reviewer** from the team
3. **Rebase with Main:**
   ```bash
   git checkout main
   git pull origin main
   git checkout feature/your-work
   git rebase origin/main
   ```
4. **Squash Merge Only:** Use squash-merge to keep `main` history clean
5. **Verify:** Ensure `dbt build` passes before requesting review

### Code Review Checklist

Before requesting a review, ensure:
- [ ] `dbt build` passes (models run and tests pass)
- [ ] All models have schema YAML files with descriptions and tests
- [ ] No hardcoded credentials or tokens (see [ACCESS_TOKEN_HANDLING.md](ACCESS_TOKEN_HANDLING.md))
- [ ] No hardcoded catalog/schema names (use variables)
- [ ] Code follows the SQL style guide and naming conventions
- [ ] Branch is rebased with latest `main`
- [ ] Project added to [PROJECTS.md](PROJECTS.md)
- [ ] If using AI assistants: reviewed [AI_CODING_GOTCHAS.md](AI_CODING_GOTCHAS.md)

---

## dbt Conventions

### Naming
- **Staging:** `stg_<source>__<table>.sql` (double underscore separating source and table)
- **Intermediate:** `int_<description>.sql`
- **Facts:** `fct_<description>.sql`
- **Dimensions:** `dim_<description>.sql`
- **All names** in `lowercase_snake_case`

### SQL Style
- **UPPERCASE** for SQL keywords (`SELECT`, `FROM`, `WHERE`)
- **lowercase_snake_case** for column names
- Use CTEs (`WITH` clauses) over subqueries
- Explicit `JOIN` types always (`INNER JOIN`, `LEFT JOIN`)
- One column per line in `SELECT` statements
- Use `ref()` and `source()` -- never hardcode table names

### Materialisation Strategy
| Layer | Materialisation | Rationale |
|-------|----------------|-----------|
| Staging | `view` | Lightweight, always fresh |
| Intermediate | `ephemeral` or `view` | Depends on complexity and reuse |
| Marts (facts/dims) | `table` or `incremental` | Depends on data volume |
| Snapshots | `snapshot` | SCD Type 2 tracking |

### Testing Requirements
- Every model must have a schema YAML file
- Primary keys: `unique` + `not_null` tests
- Categorical columns: `accepted_values` where appropriate
- Foreign keys: `relationships` tests for integrity

---

## Databricks Token Handling

| Branch | Tokens Allowed? |
|--------|-----------------|
| Feature branches | Yes (during development) |
| Main/Master | No (must be redacted) |

**Best Practice:** Use a gitignored config file (`.env.local`) for your tokens.

Full policy: [ACCESS_TOKEN_HANDLING.md](ACCESS_TOKEN_HANDLING.md)

---

## Projects

View current projects: **[PROJECTS.md](PROJECTS.md)**

---

## Need Help?

- **Technical Questions:** Post in [#snowmelt-dbt-wg](https://databricks.slack.com/channels/snowmelt-dbt-wg)
- **Access Issues:** Request via [Opal](https://app.opal.dev/) (search `dbt-databricks-training-internal`)
- **Code Review:** Tag WG members in your PR
