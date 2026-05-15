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
