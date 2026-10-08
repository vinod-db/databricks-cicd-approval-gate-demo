# Governed change → approval gate → production

A tiny, working demo of how a change to a **governed data asset** reaches production **only after a
human approval** — the automated version of a "design doc → approval → then build the production
table" control.

## The flow

```
 Open PR            CI validates          Merge to main         Required reviewer         Service principal
 (edit the     →    the change      →     (change accepted)  →   approves "production"  →  applies it to
  definition)       automatically                                 environment               Unity Catalog
```

- **Change-as-code:** the governed schema and volume are declared in `databricks.yml` (a Databricks
  Asset Bundle). You change the *definition*, not production directly.
- **Automated validation:** every pull request runs `databricks bundle validate`.
- **The approval gate:** the production deploy runs only after a required reviewer approves the
  `production` environment — it literally pauses until someone signs off.
- **No hand-built prod:** the deploy runs as a **service principal** that owns the schema; people
  don't create or alter production objects by hand.
- **Audit:** who approved, when it deployed, and who touched the data afterward are all in the
  platform's immutable audit log.

## How to record the demo

1. Create a branch and edit `databricks.yml` — e.g. bump the schema comment `(v1)` -> `(v2)`.
2. Open a pull request -> `validate` runs and passes.
3. Merge to `main`.
4. Open the **Actions** tab: `deploy-prod` shows **"Review required"** and waits.
5. Click **Review deployments -> production -> Approve and deploy**.
6. The deploy runs; the change is live in Unity Catalog.

## Setup

- `production` environment with a **required reviewer**.
- Repo secrets `DATABRICKS_HOST` and `DATABRICKS_TOKEN` (the deploy service principal).
- Production-grade upgrade: swap the token for **OIDC / workload identity federation** (no stored secret).
