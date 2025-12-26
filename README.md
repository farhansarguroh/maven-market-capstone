# Databricks capstone: Maven Market 

This repository contains the end-to-end data engineering pipeline for the **Maven Market** project, built using **Databricks Delta Live Tables (DLT)** and a **Medallion Architecture** (Bronze, Silver, and Gold layers). The project integrates multi-source data including CSV files from ADLS, NoSQL data from MongoDB, and real-time streaming data from Confluent Kafka.

## Project Architecture

The project follows the Medallion architecture to ensure data quality, reliability, and performance:

1. **Bronze Layer**: Raw data ingestion from multiple sources.
* **ADLS (CSV)**: Ingests `Transactions`, `Returns`, `Calendars`, `Stores`, and `Regions` using Auto Loader (`cloudFiles`).
* **MongoDB**: Ingests `Customers` and `Products` snapshots.
* **Kafka**: Real-time streaming for `Orders` and `Inventory` events.


2. **Silver Layer**: Data cleaning, schema enforcement, and standardization.
* Implements **SCD Type 2** tracking for `Customers` and `Products` to maintain historical changes.
* Standardizes date formats and casts data types for downstream analysis.


3. **Gold Layer**: Business-level aggregates and materialized views.
* **Fact Tables**: `fact_sales`, `fact_returns`, and `fact_inventory_events`.
* **Dimension Tables**: `dim_products`, `dim_customers`, `dim_stores`, etc.
* **Aggregated Tables**: Store performance, product profitability, and customer value.
* **Materialized Views**: Real-time KPIs such as inventory health and hourly order counts.



## Folder Structure

* **/bronze**: Scripts for raw data ingestion.
* `adls_dlt`: DLT pipeline for ADLS CSV ingestion.
* `kafka_topic`: Configuration and streaming setup for Kafka.
* `mongo_dlt`: Secure ingestion from MongoDB using Databricks Secrets.


* **/silver**: Data transformation and quality logic.
* `dlt_silver`: Main DLT pipeline for cleaning and standardization. 

* **/gold**: Business logic and reporting.
* `dlt_gold_facts`: Unifies online orders and POS transactions into a single fact_sales table and defines core business fact tables.
* `dlt_good_dimensions`: Creates standardized dimension tables for products, customers, stores, regions, and calendars with data quality checks.
* `gold_aggregates`: Calculates high-level business metrics including store performance, product profitability, and customer lifetime value.
* `gold_mvs`: Defines specialized materialized views for real-time monitoring, such as inventory health and per-minute order counts.

## Key Features

* **Unified Sales View**: Combines streaming Online Orders (Kafka) and Batch In-Store Transactions (ADLS) into a single `fact_sales` table.
* **Data Quality**: Uses `@dlt.expect_or_drop` and `@dlt.expect` to ensure only high-quality data reaches the Silver and Gold layers.
* **Slowly Changing Dimensions (SCD)**: Tracks history for customers and products, allowing for accurate point-in-time reporting.
* **Scalability**: Built entirely on Databricks DLT, enabling automatic scaling and lineage tracking.

## Setup Instructions

1. **Secrets**: Configure a Databricks Secret Scope (e.g., `capstone_team2_scope`) containing:
* `adls-access-key`
* `mongo-uri`
* `confluent-api-key` / `confluent-api-secret`


2. **DLT Pipeline**: Create a new Delta Live Tables pipeline in Databricks and add the relevant notebooks from the `bronze`, `silver`, and `gold` directories.
3. **Catalog**: Ensure the `maven_uc` catalog and respective schemas (`bronze_dlt`, `silver_dlt`, `gold_dlt`) are created in Unity Catalog.