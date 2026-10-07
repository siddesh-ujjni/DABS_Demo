# DABs CI/CD Demo
<!-- Deployed from Databricks workspace -->

A minimal example of **Declarative Automation Bundles (DABs)** with **GitHub Actions** for automated multi-environment deployment on Databricks.

## Project Structure

```
DABS_Demo/
├── databricks.yml                    # Bundle configuration (targets, variables)
├── resources/
│   └── dabs_cicd_demo.job.yml        # Job resource definition
├── src/
│   └── notebooks/
│       └── sample_job.ipynb          # Notebook executed by the job
└── .github/
    └── workflows/
        └── deploy.yml                # GitHub Actions CI/CD pipeline
```

## Environments

| Target | Trigger | Catalog | Schema |
|--------|---------|---------|--------|
| **dev** | Push to `main` | `dev_catalog` | `dev_schema` |
| **prod** | Manual dispatch (with approval) | `prod_catalog` | `prod_schema` |

## GitHub Actions Pipeline

The CI/CD pipeline has three stages:

1. **Validate** — Runs `databricks bundle validate --strict` on every push and PR
2. **Deploy Dev** — Auto-deploys to dev on push to `main`, then runs the job
3. **Deploy Prod** — Manual trigger via `workflow_dispatch` with environment approval gate

## Setup

### 1. GitHub Secrets

Add these secrets to your GitHub repository (Settings → Secrets → Actions):

| Secret | Description |
|--------|-------------|
| `DATABRICKS_HOST` | Workspace URL (e.g. `https://adb-xxx.azuredatabricks.net`) |
| `DATABRICKS_TOKEN` | PAT or OAuth token for the service principal |

> **Recommended**: Use a Databricks service principal with GitHub OIDC for token-free authentication in production.

### 2. GitHub Environments

Create `dev` and `prod` environments in GitHub (Settings → Environments):
- **prod**: Add required reviewers for deployment approval gates

### 3. Databricks Setup

- Ensure `dev_catalog`/`dev_schema` and `prod_catalog`/`prod_schema` exist in Unity Catalog
- Update `run_as.service_principal_name` in `databricks.yml` for the prod target
- Update the email in `dabs_cicd_demo.job.yml` for failure notifications

## Local Development

```bash
# Validate the bundle
databricks bundle validate --strict -t dev

# Deploy to dev
databricks bundle deploy -t dev

# Run the job
databricks bundle run dabs_cicd_demo_job -t dev
```
