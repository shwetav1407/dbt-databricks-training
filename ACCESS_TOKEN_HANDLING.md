# Access Token Handling Policy

**This document outlines the policy for handling Databricks Personal Access Tokens (DAPI tokens) and other secrets in this repository.**

---

## Policy Summary

| Branch Type | Tokens Allowed? | Notes |
|-------------|-----------------|-------|
| **Feature branches** | Yes | Acceptable during development |
| **Main/Master branch** | No | Must be redacted before merge |

---

## Guidelines

### During Development (Feature Branches)

When working on a feature branch, you **may** include Databricks tokens in configuration files for testing purposes. This is acceptable per Drawbridge security policy.

### Before Merging to Main/Master

**Before creating a Pull Request to main/master**, you **must**:

1. **Replace all hardcoded tokens** with placeholders:
   ```bash
   export DATABRICKS_TOKEN="<YOUR_DATABRICKS_TOKEN>"
   ```

2. **Use environment variable references** in dbt and Python code:
   ```yaml
   # profiles.yml (gitignored, but in case of templates)
   host: "{{ env_var('DATABRICKS_HOST') }}"
   token: "{{ env_var('DATABRICKS_TOKEN') }}"
   ```

   ```python
   import os
   token = os.environ.get('DATABRICKS_TOKEN')
   ```

3. **Add setup instructions** pointing to secure authentication methods:
   ```bash
   databricks auth login --host https://your-workspace.azuredatabricks.net
   ```

---

## Security Scanning

This repository is monitored by **Drawbridge**, which scans for secrets in:
- GitHub (code repositories)
- Jira (tickets and comments)
- Confluence (wiki pages)

If a token is detected in the **main/master branch**, you will be contacted to:
1. **Revoke the token** (can be done centrally by security team)
2. **Clean up the source** by removing the secret from main/master

**Note:** You do NOT need to remove secrets from all branches -- only from main/master.

---

## Recommended Authentication Methods

### 1. Gitignored Config File (Recommended for Local Dev)

**This is the safest approach** - tokens never get committed because the file is gitignored.

**Step 1:** Create a `.env.local` file (NOT committed):
```bash
# .env.local (gitignored - your actual secrets)
DATABRICKS_HOST=https://adb-123456789.11.azuredatabricks.net
DATABRICKS_TOKEN=dapi1234567890abcdef...
DBT_CATALOG=your_catalog
DBT_SCHEMA=your_schema
```

**Step 2:** Create a template file that IS committed:
```bash
# .env.template (committed - shows required variables)
DATABRICKS_HOST=<YOUR_DATABRICKS_HOST>
DATABRICKS_TOKEN=<YOUR_DATABRICKS_TOKEN>
DBT_CATALOG=<YOUR_CATALOG>
DBT_SCHEMA=<YOUR_SCHEMA>
```

**Step 3:** Reference in dbt profiles:
```yaml
# profiles.yml (gitignored)
your_project:
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

### 2. Databricks CLI Authentication
```bash
# One-time setup
databricks auth login --host https://your-workspace.azuredatabricks.net

# Token is stored securely in ~/.databrickscfg
```

### 3. Environment Variables
```bash
# Set in your shell profile (~/.zshrc or ~/.bashrc)
export DATABRICKS_HOST="https://your-workspace.azuredatabricks.net"
export DATABRICKS_TOKEN="your-token-here"
```

### 4. Databricks Secret Scopes (for Production)
```python
# In Databricks notebooks, use secret scopes
token = dbutils.secrets.get(scope="my-scope", key="databricks-token")
```

---

## Pre-Merge Checklist

Before merging to main/master, verify:

- [ ] No `dapi...` tokens in any files
- [ ] No other secrets (API keys, passwords) in code
- [ ] `profiles.yml` is NOT committed (should be gitignored)
- [ ] Configuration files use placeholders or environment variables
- [ ] README references environment variables, not actual values

### Quick Check Command

Run this command to scan for potential DAPI tokens before committing:

```bash
# From repository root
grep -r "dapi[a-zA-Z0-9]\{30,\}" --include="*.py" --include="*.sql" --include="*.md" --include="*.yaml" --include="*.yml" --include="*.json" .
```

If any matches are found, replace them with placeholders before merging.

---

## Related Documentation

- [Databricks CLI Authentication](https://docs.databricks.com/en/dev-tools/cli/authentication.html)
- [dbt Databricks Profile Setup](https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup)
- [Unity Catalog Secret Scopes](https://docs.databricks.com/en/security/secrets/secret-scopes.html)

---

**Last Updated:** February 2026  
**Policy Source:** Drawbridge Security Scanning Guidelines
