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

The pipeline follows the **Medallion architecture** (Bronze → Silver → Gold), culminating in an **MLflow-tracked binary classification model** that predicts customer churn. A **Power BI executive dashboard** consuming the Gold layer tables is planned as a sequel project.

**Business problem:** 59.1% of Olist's 93,358 customers never returned after their first purchase. This project identifies which customers are at churn risk using RFM behavioral features, enabling targeted retention campaigns.

---
