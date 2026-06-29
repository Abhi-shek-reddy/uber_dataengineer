# Uber End-to-End Data Engineering Project

A real-time **and** batch data engineering pipeline that takes Uber-style trip data all the way from raw events to an analytics-ready dimensional model, built on Azure and Databricks.

![Architecture](docs/architecture.png)

## Overview

This project ingests two kinds of data — live trip events (streaming) and reference/dimension data (batch) — lands them in a data lake, and processes them through a **medallion architecture** (Bronze → Silver → Gold) inside Databricks. The Gold layer is modeled as a **star schema** with fact and dimension tables, with full historical tracking via **Slowly Changing Dimensions (Type 2)**.

## Tech stack

| Layer | Tools |
|-------|-------|
| Streaming ingestion | Azure Event Hubs (Kafka-compatible), FastAPI event producer |
| Batch ingestion | Azure Data Factory |
| Storage | Azure Data Lake Storage (ADLS Gen2), Delta Lake |
| Processing | Databricks, PySpark, Spark Structured Streaming, Spark Declarative Pipelines (SDP) |
| Modeling | Star schema (facts + dimensions), SCD Type 2 |

## Architecture

Data flows through two lanes that converge inside Databricks:

- **Real-time lane:** a FastAPI app produces live trip events into Azure Event Hubs, which Spark Structured Streaming reads continuously.
- **Batch lane:** Azure Data Factory ingests source and dimension data into Azure Data Lake.

Both lanes feed the medallion layers:

- **Bronze** — raw data landed as-is (streaming events + batch loads). Schema handled with SDP.
- **Silver** — cleaned, joined, and deduplicated tables built from streaming tables.
- **Gold** — analytics-ready star schema (fact + dimension tables) with SCD Type 2 history.

## Key concepts

- **Metadata-driven streaming** — a single, reusable pipeline processes many tables from a config/metadata table instead of hardcoded logic per table.
- **Slowly Changing Dimensions (Type 2)** — dimension changes create a new versioned row (with start/end dates and an `is_current` flag) instead of overwriting, preserving full history.
- **Star schema** — a central `Fact_Trips` table holding measures (fare, distance, tip) and surrogate keys, surrounded by dimension tables (`Dim_Driver`, `Dim_Rider`, `Dim_Location`, `Dim_Date`).
- **OBT (optional serving layer)** — wide, pre-joined tables materialized from the star for dashboards or ML features.

## Project structure

```
uber_dataengineer/
├── producer/              # FastAPI app that streams trip events to Event Hub
├── adf/                   # Azure Data Factory pipeline definitions / exports
├── notebooks/
│   ├── bronze/            # raw ingestion (structured streaming, SDP)
│   ├── silver/            # cleaning, joins, metadata-driven streaming
│   └── gold/              # dimensions (SCD2), fact table, star schema
├── config/                # metadata / control tables
├── docs/
│   └── architecture.png   # pipeline diagram
└── README.md
```

> Adjust the structure above to match how your own files are organized.

## Getting started

Prerequisites:

- An Azure account (free tier works) with Event Hubs, Data Factory, and ADLS Gen2
- A Databricks workspace
- Python 3.9+ for the event producer

High-level steps:

1. Deploy the Azure Event Hub and create the FastAPI producer to stream events.
2. Build the Azure Data Factory pipelines to ingest batch/dimension data into ADLS.
3. In Databricks, run the Bronze notebooks to land raw data.
4. Run the Silver notebooks for cleaning, joins, and the metadata-driven pipeline.
5. Run the Gold notebooks to build SCD2 dimensions and the star schema.

## Credits

Built as a learning project following the end-to-end tutorial by **Ansh Lamba**.
