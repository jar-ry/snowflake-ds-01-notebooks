# 01 - Snowflake Notebooks

**Maturity Level:** ⭐ Beginner | **Runs In:** Snowflake UI (Snowsight)

## Overview

Run the full ML workflow entirely inside a Snowflake Notebook. No local environment, no connection files, no package management — everything executes on Snowflake-managed compute in the browser.

## When to Use

- ✅ Learning Snowflake ML capabilities quickly
- ✅ Rapid prototyping and sharing results with stakeholders
- ✅ Zero local setup — packages, compute, and kernels managed by Snowflake
- ✅ Built-in collaboration via Snowsight
- ⚠️ Lighter IDE features than a local editor (no breakpoints, limited extensions)
- ⚠️ Not ideal for complex Git/CI workflows

## What the Notebook Does

The single notebook (`CLV_MODEL_NOTEBOOK.ipynb`) runs an end-to-end Expected Monthly Value regression pipeline:

| Step | What Happens |
|------|--------------|
| **Feature Store setup** | Creates `MODELLING` (Model Registry) and `FEATURE_STORE` schemas, registers a `CUSTOMER` entity |
| **Feature engineering** | Joins `CUSTOMERS` + `PURCHASE_BEHAVIOR`, engineers features (`AVERAGE_ORDER_PER_MONTH`, `DAYS_SINCE_LAST_PURCHASE`, etc.) via inline Snowpark functions |
| **FeatureView creation** | Registers a managed FeatureView backed by a Dynamic Table for automatic refresh |
| **Dataset generation** | Builds a versioned Snowflake Dataset from the FeatureView using a spine DataFrame |
| **HPO + Training** | Uses `tune.Tuner` with `RandomSearch` (10 trials) over an XGBoost regression pipeline, distributed across the notebook cluster via `scale_cluster(5)` |
| **Experiment tracking** | Logs params, metrics, and model artifacts per trial using `ExperimentTracking` |
| **Model promotion** | Selects best trial, sets default version, alias (`PROD`), tag, and copies to a `PROD_SCHEMA` |
| **Inference service** | Deploys the model as a container service on SPCS for real-time predictions |
| **Model monitoring** | Configures a `ModelMonitor` for ongoing performance tracking |

## Contents

```
01_snowflake_notebooks/
├── README.md                        # This file
└── CLV_MODEL_NOTEBOOK.ipynb         # Full pipeline (import into Snowsight)
```

All helper functions (versioning, Feature Store/Registry creation, SQL formatting, feature engineering) are defined inline in the notebook cells — no external `.py` files required.

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                       Snowflake UI                              │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                 Snowflake Notebook                        │  │
│  │                                                           │  │
│  │  Feature Store ──► FeatureView ──► Dataset ──► Tuner/HPO  │  │
│  │                                                           │  │
│  │  scale_cluster(5)  ←── distributes trials across nodes    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                           │                                     │
│            ┌──────────────┼──────────────┐                      │
│            ▼              ▼              ▼                       │
│     Warehouse      Container Pool   Model Registry              │
│     (queries)      (HPO + serve)    (versioned models)          │
└─────────────────────────────────────────────────────────────────┘
```

## Key Difference from 02_ml_jobs_notebook

| | 01 Snowflake Notebooks | 02 ML Jobs Notebook |
|-|------------------------|---------------------|
| **Where code lives** | Inline in Snowflake Notebook | Local `.ipynb` + `.py` files |
| **Session** | `get_active_session()` (automatic) | `Session.builder.configs(...)` via connection file |
| **HPO compute** | `scale_cluster(5)` — scales the notebook container cluster | `@remote("CLV_MODEL_POOL_CPU")` — submits an ML Job to a separate compute pool |
| **External files** | None — everything is self-contained | `feature_engineering_fns.py`, `helper/useful_fns.py` |
| **Best for** | Learning, demos, fast iteration | Teams with local IDE workflows who need SPCS-backed training |

## Prerequisites

- Completed `Step01_Setup.ipynb` (creates database, tables, mock data)
- Access to Snowsight
- A warehouse (e.g. `RETAIL_REGRESSION_DEMO_WH`)
- A CPU compute pool (e.g. `CLV_MODEL_POOL_CPU`) for HPO and inference service

## Quick Start

1. Log into Snowsight
2. Navigate to **Notebooks** → **Import .ipynb file**
3. Upload `CLV_MODEL_NOTEBOOK.ipynb`
4. Select your warehouse and compute pool, then run cells top-to-bottom

## Snowflake Services Used

- Feature Store (Entity, FeatureView, Dynamic Tables)
- Model Registry (versioning, aliases, tags)
- Datasets & DataConnectors
- Experiment Tracking
- Tuner / HPO (`tune.Tuner`, `RandomSearch`)
- `scale_cluster` (notebook container scaling)
- SPCS Model Service (real-time inference)
- Model Monitoring

## Related Repos

| Repo | Description |
|------|-------------|
| [snowflake-ds-setup](https://github.com/jar-ry/snowflake-ds-setup) | Environment setup, data generation, and helper utilities (run this first) |
| [snowflake-ds-02-ml-jobs-notebook](https://github.com/jar-ry/snowflake-ds-02-ml-jobs-notebook) | Same pipeline, but run locally with `@remote` ML Jobs |
| [snowflake-ds-03-ml-jobs-framework](https://github.com/jar-ry/snowflake-ds-03-ml-jobs-framework) | Production-grade modular framework using `submit_directory` |
| [snowflake-ds-04-feature-store](https://github.com/jar-ry/snowflake-ds-04-feature-store) | Split repo: Feature Store with FeatureViews and Versioned Datasets |
| [snowflake-ds-04-ml-training](https://github.com/jar-ry/snowflake-ds-04-ml-training) | Split repo: ML Training with training, promotion, inference, monitoring |
