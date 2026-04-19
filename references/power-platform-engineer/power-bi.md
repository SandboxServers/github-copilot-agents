## Power BI

### Data Modeling Best Practices
1. **Star schema** — fact tables surrounded by dimension tables, avoid snowflake unless necessary
2. **Avoid bidirectional relationships** unless absolutely required (RLS, security implications)
3. **Use calculated columns sparingly** — prefer calculated measures or Power Query transformations
4. **Define measures explicitly** — don't rely on implicit measures (SUM of column)
5. **Date table** — always create a dedicated date dimension, mark it as date table

### DAX Key Patterns
- **CALCULATE** — the most important function, modifies filter context
- **Time intelligence** — TOTALYTD, SAMEPERIODLASTYEAR, DATEADD
- **Iterator vs aggregator** — SUMX (row-by-row) vs SUM (column), choose based on need
- **Variables** — use VAR/RETURN for readability and performance (evaluated once)
- **ALL/ALLEXCEPT** — remove filters for percentage-of-total calculations

### Row-Level Security (RLS)
- Define roles with DAX filter expressions in Power BI Desktop
- Filters table rows — can't restrict access to tables, columns, or measures
- **Map security groups** to roles (not individual users) — easier management via Entra ID
- RLS applies only to Viewer role — Admins/Members/Contributors bypass RLS
- When user maps to multiple roles, filters are **additive** (union, not intersection)
- **Test with "View As"** in Desktop — validate both expected and unexpected values
- Apply RLS to **dimension tables** not fact tables for better performance
- **Dynamic RLS** — use `USERPRINCIPALNAME()` for per-user filtering

### Deployment Pipelines
- Promote content from Development → Test → Production
- Compare content across stages to identify differences
- Deploy semantic models, reports, dashboards between workspaces
- Support parameterized deployment rules for different environments

### Dataflows
- ETL/data prep in Power BI, outputs data to Dataverse or Azure Data Lake
- Reusable across multiple reports/datasets
- Schedule refresh independently from reports
- **Back up dataflow definitions** — deleted dataflows can't be recovered
