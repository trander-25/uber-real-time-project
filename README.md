# Uber Real-Time Data Engineering Project

A real-time data engineering project that simulates Uber ride booking events, streams them through Azure Event Hubs, and processes them in Databricks using Spark Structured Streaming and Lakeflow Declarative Pipelines.

The project demonstrates an end-to-end medallion-style pipeline from event generation to analytical dimensional tables.

![Architecture](architecture.png)

## Project Overview

This project includes:

- A FastAPI application that generates synthetic Uber ride events.
- Azure Event Hubs as the streaming message broker.
- Databricks ingestion from Event Hubs using the Kafka-compatible endpoint.
- Bronze, Silver, and Gold processing layers.
- A dimensional model with passenger, driver, vehicle, payment, booking, location, and fact tables.

## Architecture

```text
FastAPI App
    |
    | generates synthetic ride event
    v
Azure Event Hubs
    |
    | Spark Structured Streaming
    v
Bronze Layer: rides_raw
    |
    v
Silver Layer: stg_rides + silver_obt
    |
    v
Gold Layer: dimension tables + fact table
```

## Tech Stack

- Python 3.12+
- FastAPI
- Jinja2
- Faker
- Azure Event Hubs
- Databricks
- PySpark Structured Streaming
- Databricks Lakeflow Declarative Pipelines / Delta Live Tables
- SQL

## Repository Structure

```text
.
+-- api.py                         # FastAPI app
+-- connection.py                  # Sends ride events to Azure Event Hubs
+-- data.py                        # Synthetic ride data generator
+-- requirements.txt               # Python dependencies
+-- pyproject.toml                 # Project metadata and uv dependencies
+-- architecture.png               # Architecture diagram
+-- Uber_Project.svg               # Project diagram/source asset
+-- Data/
|   +-- bulk_rides.json            # Synthetic initial load dataset
|   +-- map_cancellation_reasons.json
|   +-- map_cities.json
|   +-- map_payment_methods.json
|   +-- map_ride_statuses.json
|   +-- map_vehicle_makes.json
|   +-- map_vehicle_types.json
+-- templates/
|   +-- home.html
|   +-- confirmation.html
+-- Code_Files/
    +-- ingest.py                  # Bronze ingestion from Event Hubs
    +-- silver.py                  # Streaming and bulk staging logic
    +-- silver_obt.sql             # Silver one-big-table transformation
    +-- model.py                   # Dimensional model
    +-- bronze_adls.ipynb
    +-- silver_obt.ipynb
```

## Data Pipeline

### 1. Event Producer

The local FastAPI app exposes two routes:

- `GET /` - ride booking page
- `GET /book` - generates a synthetic ride and sends it to Event Hubs

Ride events are generated in `data.py` using `Faker` and include ride identifiers, passenger and driver details, pickup/dropoff locations, vehicle details, payment method, fare metrics, status, and rating.

### 2. Bronze Layer

`Code_Files/ingest.py` reads from Azure Event Hubs through the Kafka-compatible endpoint and creates the raw streaming table:

- `rides_raw`

### 3. Silver Layer

`Code_Files/silver.py` creates `stg_rides` from:

- `bulk_rides` for initial load
- `rides_raw` for real-time streaming events

`Code_Files/silver_obt.sql` enriches ride records with lookup tables and creates:

- `silver_obt`

### 4. Gold Layer

`Code_Files/model.py` builds the analytical model:

- `dim_passenger`
- `dim_driver`
- `dim_vehicle`
- `dim_payment`
- `dim_booking`
- `dim_location`
- `fact`

## Local Setup

Create and activate a virtual environment:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Or with `uv`:

```bash
uv sync
```

Create a `.env` file in the project root:

```env
CONNECTION_STRING="<azure-event-hubs-connection-string>"
EVENT_HUBNAME="<event-hub-name>"
```

Run the API:

```bash
uvicorn api:app --reload --host 0.0.0.0 --port 8000
```

Open:

```text
http://localhost:8000
```

## Databricks Setup

1. Upload the JSON files in `Data/` to a location accessible by Databricks.
2. Create the mapping tables:
   - `map_cancellation_reasons`
   - `map_cities`
   - `map_payment_methods`
   - `map_ride_statuses`
   - `map_vehicle_makes`
   - `map_vehicle_types`
3. Create the `bulk_rides` table from `Data/bulk_rides.json`.
4. Store the Event Hubs connection string in Spark configuration or a Databricks secret scope as `connection_string`.
5. Run the pipeline files in this order:
   - `Code_Files/ingest.py`
   - `Code_Files/silver.py`
   - `Code_Files/silver_obt.sql`
   - `Code_Files/model.py`

## Data and Privacy Notes

- The ride records in this repository are synthetic and generated with `Faker`.
- Names, emails, phone numbers, addresses, license plates, and driver licenses are not real user data.
- No real Azure connection string, token, password, or API key is stored in the source code.
- Runtime secrets are expected to come from `.env`, Spark configuration, or Databricks secrets.

## Expected Output

After the pipeline runs successfully, the project produces:

- A real-time ride event producer.
- Raw streaming data in the Bronze layer.
- Cleaned and enriched ride data in the Silver layer.
- Dimension and fact tables in the Gold layer for analytics.
