# E-Commerce Retention Lakehouse
### End-to-End Delta Lake Pipeline with MLflow-Tracked Churn Prediction

![Databricks](https://img.shields.io/badge/Databricks-Community%20Edition-FF3621?style=flat&logo=databricks&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-3.4.1-E25A1C?style=flat&logo=apachespark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-Medallion%20Architecture-003366?style=flat)
![MLflow](https://img.shields.io/badge/MLflow-Experiment%20Tracking-0194E2?style=flat&logo=mlflow&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10-3776AB?style=flat&logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=flat)

---

## Overview

This project builds a production-style data lakehouse on **Databricks Community Edition** using the [Olist Brazilian E-Commerce dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — a real-world 9-table relational dataset covering 99,441 orders across September 2016 to October 2018.

The pipeline follows the **Medallion architecture** (Bronze → Silver → Gold), ending with an **MLflow-tracked Random Forest churn model (AUC 0.803)** that scores all 93,357 customers. Churn probability scores are written back to the Gold layer as a Delta table. A **Power BI executive dashboard** consuming the Gold layer is built as a sequel project.

**Business problem:** 59.1% of Olist's 93,357 customers never returned after their first purchase. This project identifies which customers are at risk of churning using RFM and delivery behavioral features — enabling targeted retention campaigns.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DATA SOURCES (9 CSVs)                          │
│   orders · customers · items · payments · reviews · products        │
│               sellers · geolocation · categories                    │
│           Unity Catalog Volume: /raw_data/                          │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        BRONZE LAYER                                 │
│   Raw ingestion to Delta format · Metadata stamping                 │
│   _ingested_at · _source_file · Schema inference                    │
│   9 Delta tables · No business logic applied                        │
│   /Volumes/workspace/default/raw_data/bronze/                       │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        SILVER LAYER                                 │
│   Filter delivered orders · Null checks · 4-table master join       │
│   RFM feature engineering · Churn label definition (180-day rule)   │
│   93,357 unique customer profiles · Data quality fixes              │
│   /Volumes/workspace/default/raw_data/silver/                       │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         GOLD LAYER                                  │
│   customer_churn_segments  · revenue_by_state                       │
│   monthly_revenue_trend    · customer_churn_scores (ML output)      │
│   /Volumes/workspace/default/raw_data/gold/                         │
└──────────────┬───────────────────────────────┬──────────────────────┘
               │                               │
               ▼                               ▼
┌──────────────────────────┐     ┌─────────────────────────────────┐
│         MLFLOW           │     │           POWER BI              │
│  Logistic Regression     │     │     Executive Dashboard         │
│  Random Forest (AUC 0.80)│     │     (Sequel Project)            │
│  Model Registry v1       │     │     5 pages · Gold CSV export   │
└──────────────────────────┘     └─────────────────────────────────┘

```

---

## Tech Stack

| Component | Technology |
|---|---|
| Data Platform | Databricks Community Edition (Serverless) |
| Storage Format | Delta Lake |
| Processing Engine | PySpark 3.4.1 |
| Pipeline Pattern | Medallion Architecture (Bronze / Silver / Gold) |
| Storage Layer | Unity Catalog Volumes |
| ML Experiment Tracking | MLflow |
| ML Model Registry | MLflow Model Registry |
| ML Libraries | scikit-learn (Logistic Regression, Random Forest) |
| Version Control | GitHub via Databricks Repos |
| BI Layer | Power BI (sequel project) |

---

## Dataset

**Source:** [Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) — Kaggle

| Table | Description | Rows |
|---|---|---|
| olist_orders_dataset | Order status and timestamps | 99,441 |
| olist_customers_dataset | Customer ID, city, state | 99,441 |
| olist_order_items_dataset | Products per order, price, freight | ~112,000 |
| olist_order_payments_dataset | Payment type, installments, value | ~103,000 |
| olist_order_reviews_dataset | Review scores and comments | ~100,000 |
| olist_products_dataset | Product category and dimensions | ~33,000 |
| olist_sellers_dataset | Seller city and state | ~3,000 |
| olist_geolocation_dataset | Zip code lat/long | ~1,000,000 |
| product_category_name_translation | Portuguese to English categories | 71 |

> **Important:** In the Olist dataset, `customer_id` is order-specific — a new ID is generated per order. `customer_unique_id` is the actual person identifier. Aggregations must be done at the `customer_unique_id` level to correctly compute frequency, lifespan, and monetary features.

---

## Notebooks

| # | Notebook | What it does |
|---|---|---|
| 01 | `01_bronze_ingestion` | Reads all 9 CSVs, adds `_ingested_at` and `_source_file` metadata, writes to Delta |
| 02 | `02_silver_transformation` | Filters delivered orders, joins 4 tables, engineers RFM features, labels churn |
| 03 | `03_gold_aggregations` | Builds 3 business-ready Gold tables for BI consumption |
| 04 | `04_mlflow_churn_model` | Trains LR and RF models, tracks with MLflow, registers best model to registry |
| 05 | `05_batch_inference` | Loads registered model, scores all 93,357 customers, writes scores to Gold |

---

## Key Results

### Pipeline Numbers

| Metric | Value |
|---|---|
| Total orders ingested | 99,441 |
| Delivered orders analysed | 96,478 |
| Unique customers profiled | 93,357 |
| Overall churn rate | **59.1%** |
| Churned customers | 55,132 |
| Active customers | 38,225 |
| Dataset date range | Sep 2016 — Oct 2018 |
| Peak revenue month | Nov 2017 — 1.15M BRL |
| Business growth | ~22x order volume over dataset period |
| Top revenue state | SP (Sao Paulo) — 5.77M BRL |

---

### ML Model Results

| Model | AUC | Accuracy | Precision | Recall |
|---|---|---|---|---|
| Logistic Regression (baseline) | 0.560 | 59.3% | 59.4% | 97.9% |
| **Random Forest (winner)** | **0.803** | **73.3%** | **71.9%** | **89.8%** |

> Random Forest registered to MLflow Model Registry as `workspace.default.olist_churn_model` — Version 1, Status: READY

---

### Batch Inference Output (Gold Layer)

| Churn Risk Segment | Customers | Avg Churn Probability |
|---|---|---|
| High Risk | 28,297 | 79.6% |
| Medium Risk | 52,061 | 56.6% |
| Low Risk | 12,999 | 24.0% |

---

### Customer Segments (Gold Layer)

| Churn Risk Segment | Definition | Count |
|---|---|---|
| Low Risk | Last order within 90 days | 18,520 |
| Medium Risk | Last order 91–180 days ago | 19,706 |
| High Risk | Last order 181–365 days ago | 34,452 |
| Lost | No order in 365+ days | 20,680 |

| RFM Segment | Avg Spend (BRL) | Count |
|---|---|---|
| Champions | 438.72 | 1,604 |
| New Customer | 163.42 | 17,904 |
| Loyal | 134.22 | 1,197 |
| Lost Customer | 160.11 | 72,653 |

> Champions are only 1.7% of customers but spend **2.7x more** than Lost Customers on average.

---

### Top States by Revenue

| State | Customers | Revenue (BRL) | Churn Rate |
|---|---|---|---|
| SP (Sao Paulo) | 39,156 | 5,774,380 | 55.8% |
| RJ (Rio de Janeiro) | 11,913 | 2,055,409 | 62.3% |
| MG (Minas Gerais) | 10,994 | 1,818,715 | 60.5% |

---

## ML Features Used

| Feature | Source | Description |
|---|---|---|
| `frequency` | Bronze orders | Number of orders placed per unique customer |
| `monetary_value` | Bronze payments | Total spend in BRL |
| `avg_order_value` | Bronze payments | Average spend per order |
| `stddev_order_value` | Bronze payments | Spend variance (0 for single-order customers) |
| `avg_installments` | Bronze payments | Average payment installments |
| `avg_delivery_delay` | Bronze orders | Avg days early/late vs estimated delivery |
| `max_delivery_delay` | Bronze orders | Worst delivery delay experienced |
| `customer_lifespan_days` | Bronze orders | Days between first and last order |
| `avg_freight_value` | Silver RFM | Average shipping cost paid |
| `customer_state_encoded` | Silver RFM | Label-encoded Brazilian state (27 states) |

> `recency_days` was initially included but removed after producing AUC=1.0 — identified as data leakage since it directly encodes the churn label definition (churned = recency > 180 days).

---

## Engineering Decisions & Lessons Learned

### 1. Unity Catalog Volumes over DBFS
DBFS public root is disabled on Databricks Community Edition. All Delta tables use Unity Catalog Volumes (`/Volumes/workspace/default/raw_data/`) — the current Databricks best practice for managed storage.

### 2. customer_id vs customer_unique_id
A critical Olist dataset characteristic: `customer_id` is generated per order, not per person. One customer can have up to 17 different `customer_id` values. Grouping by `customer_id` made every customer appear to have frequency=1 and lifespan=0. Fixed by resolving `customer_unique_id` via the customers table first, then aggregating at the true unique customer level.

### 3. Data Quality Issue — review_score Column
During Silver transformation, `avg_review_score` threw a `CAST_INVALID_INPUT` error. The review_score column contained misaligned timestamp strings — a source data corruption issue. Feature excluded rather than imputing bad values downstream.

### 4. Data Leakage Caught and Fixed
First MLflow run with `recency_days` as a feature produced perfect scores (AUC=1.0). Identified as data leakage — `recency_days` directly encodes the churn label (churned = recency > 180 days). Removed from feature set; AUC dropped to an honest 0.56 for LR and 0.80 for RF.

### 5. stddev_order_value Null Handling
Single-order customers (96%+ of the dataset) have no order value variance — `stddev_order_value` is NULL by definition for one data point. Filled with 0 rather than dropping rows, which would eliminate the vast majority of the dataset.

### 6. Stratified Train/Test Split
With a 59.1% churn rate, a stratified split ensures both training (74,685 rows) and test (18,672 rows) sets maintain the same churn distribution — preventing accidentally skewed evaluation.

### 7. MLflow Experiment Path
Databricks Community Edition restricts MLflow experiment creation to user directories. Experiment path dynamically constructed using `spark.sql("SELECT current_user()")` to avoid `PERMISSION_DENIED` errors.

---

## Project Structure

```
ecommerce-lakehouse-databricks/
│
├── 01_bronze_ingestion          # Raw ingestion — 9 CSVs to Delta
├── 02_silver_transformation     # RFM features + churn labeling
├── 03_gold_aggregations         # Business aggregation tables
├── 04_mlflow_churn_model        # ML training + model registry
├── 05_batch_inference           # Score all customers → Gold layer
└── README.md
```

---

## Gold Layer Tables

| Table | Rows | Description | Consumer |
|---|---|---|---|
| `customer_churn_segments` | 93,357 | RFM + churn risk segments per customer | Power BI |
| `revenue_by_state` | 27 | Revenue and churn rate by Brazilian state | Power BI |
| `monthly_revenue_trend` | 25 | Monthly orders and revenue time series | Power BI |
| `customer_churn_scores` | 93,357 | ML churn probability score per customer | Power BI |

---

## How to Reproduce

1. Sign up for [Databricks Community Edition](https://community.cloud.databricks.com) — free, no credit card
2. Download the [Olist dataset from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
3. Upload all 9 CSVs to a Unity Catalog Volume at `/Volumes/workspace/default/raw_data/`
4. Clone this repo into Databricks via **Workspace → Repos → Add Repo**
5. Run notebooks in order: `01` → `02` → `03` → `04` → `05`
6. View MLflow experiments under **Experiments** in the Databricks sidebar
7. View registered model under **Catalog → Models → olist_churn_model**

> All notebooks use Serverless compute — no cluster configuration required.

---

## Project Status

| Layer | Status | Output |
|---|---|---|
| Bronze Ingestion | Complete | 9 Delta tables |
| Silver Transformation | Complete | customer_rfm_features (93,357 rows) |
| Gold Aggregations | Complete | 3 business tables |
| MLflow Model Training | Complete | RF AUC 0.803 |
| Batch Inference | Complete | 93,357 customers scored |
| Power BI Dashboard | Sequel project | 5-page executive dashboard |

---

## Sequel Project — Power BI Dashboard

A 5-page executive dashboard consuming the Gold layer Delta tables built as a companion project:

| Page | Content |
|---|---|
| Executive Overview | KPI cards, monthly revenue trend, churn rate over time |
| Churn Risk Analysis | ML probability distribution, risk segments, churn by category |
| Customer Segmentation | RFM scatter plot, segment breakdown, avg order value by segment |
| Product & Seller Performance | Revenue by category treemap, top sellers, review score analysis |
| Order Fulfillment & Logistics | On-time delivery rate, avg delivery days by state (map visual) |

---

*Built entirely on Databricks Community Edition — zero cloud spend.*