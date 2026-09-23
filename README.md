# AdventureWorks Medallion Data Pipeline (Azure + Databricks)

A bronze → silver → gold **medallion architecture** data pipeline built on Azure Data Lake Storage and Azure Databricks, using PySpark to clean, transform, and aggregate the AdventureWorks sales dataset.

## Architecture

```
Raw CSVs (AdventureWorks)
        │
        ▼
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │ BRONZE  │ ───▶ │ SILVER  │ ───▶ │  GOLD   │
   │ raw CSV │      │ cleaned │      │  Delta  │
   └─────────┘      └─────────┘      └─────────┘
   Azure Data Lake Storage Gen2 (ADLS) containers
```

- **Bronze** — raw AdventureWorks CSV files, loaded as-is (Calendar, Customers, Product Categories/Subcategories, Products, Returns, Sales 2015–2017, Territories).
- **Silver** — cleaned and type-inferred DataFrames written out per entity.
- **Gold** — curated, aggregated, business-ready dataset (`Sales_Gold`) written in **Delta format**, with KPIs such as total sales, gross profit, orders, and customers.

## Tech Stack

- **Azure Storage Account (ADLS Gen2)** — data lake with `bronze`, `silver`, and `gold` containers
- **Microsoft Entra ID (Azure AD)** — app registration used as a service principal for OAuth authentication
- **Azure Databricks** — notebook environment and Spark compute
- **PySpark / Delta Lake** — data reading, transformation, and Delta table writes

## Project Setup

1. **Create Azure Storage** with containers: `$logs`, `bronze`, `silver`, `gold`.
2. **Upload raw AdventureWorks CSVs** into the `bronze` container.
3. **Register an app** in Microsoft Entra ID to act as a service principal (client ID, tenant ID, client secret).
4. **Grant access**: assign the `Storage Blob Data Contributor` role to the service principal on the storage account.
5. **Create a Databricks workspace and cluster** (single-node, Databricks Runtime 14.3 LTS).
6. **Authenticate Spark to ADLS** via OAuth using the service principal credentials.
7. **Read bronze data, clean/transform it, and write to `silver`** as per-entity folders.
8. **Aggregate silver data and write to `gold`** as a Delta table (`Sales_Gold`).

## Notebook

`AdventureWorks_Data_Quality.ipynb` contains the full PySpark workflow:
- OAuth configuration for ADLS access
- Loading each bronze CSV into a DataFrame with schema inference
- Row counts and data previews for quality checks
- Silver-layer cleaning/writes
- Gold-layer aggregation (KPIs, sales by category, sales by region)

## ⚠️ Security Note

The notebook in this repo originally contained a **hardcoded service principal client secret** in a `spark.conf.set(...)` call. Before pushing to GitHub:

1. **Rotate the secret** in Azure Entra ID → App registration → Certificates & secrets (treat the old one as compromised).
2. **Remove the hardcoded secret** from the notebook and replace it with a reference to a **Databricks secret scope** backed by Azure Key Vault, e.g.:

   ```python
   client_secret = dbutils.secrets.get(scope="adls-scope", key="client-secret")
   ```

3. Double-check the notebook and any config files for other IDs/secrets (tenant ID, client ID, storage account keys) before making the repo public — consider using placeholders (`<CLIENT_ID>`, `<TENANT_ID>`) instead of real values.

## Repo Contents

- `AdventureWorks_Data_Quality.ipynb` — full PySpark notebook (bronze → silver → gold)
- `Azure_Databricks_Medallion_Pipeline.pdf` — step-by-step build documentation with screenshots
