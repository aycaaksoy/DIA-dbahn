# DIA-dbahn

TU Berlin — Data Integration and Large Scale Analysis (WiSe 2025/2026)

End-to-end data integration project using Deutsche Bahn open timetable data for the Berlin region. The project covers schema design, data ingestion into PostgreSQL, SQL-based analysis, and large-scale ETL and analysis using Apache Spark.

---

## Dataset

Deutsche Bahn open timetable data for Berlin, covering **7 weeks (September–October 2025)**. The data consists of XML files packed in `.tar.gz` archives across two folders:

- `DBahn-berlin/timetables/` — planned train timetables per week
- `DBahn-berlin/timetable_changes/` — real-time changes and cancellations per week
- `DBahn-berlin/station_data.json` — station metadata (name, EVA number, coordinates)

Run `extract_files.ipynb` once to unpack all archives before running any task notebook.

---

## Stack

| Tool | Version | Purpose |
|---|---|---|
| PostgreSQL | 17 | Relational database for structured storage |
| pgAdmin 4 | latest | Database management UI |
| Apache Spark | 3.5.1 | Large-scale ETL and distributed analysis |
| PySpark | 3.5.1 | Python API for Spark |
| Jupyter Notebooks | — | Task implementations |
| Docker Compose | — | Local environment orchestration |

---

## Setup

**Prerequisites:** Docker and Docker Compose installed.

```bash
# Start all services
docker compose up -d
```

| Service | URL | Credentials |
|---|---|---|
| pgAdmin | http://localhost:8081 | dia@gmail.com / admin |
| Spark Master UI | http://localhost:8080 | — |
| PostgreSQL | localhost:5434 | dia_user / dia / db_berlin |

Place your notebooks under `path/to/your/code/` and data under `path/to/your/data/` — these directories are mounted into the Spark containers automatically.

---

## Tasks

### Task 1 — Schema Design and Data Ingestion

Designs a **star schema** for the Deutsche Bahn timetable data and ingests all raw XML data into PostgreSQL.

**Schema:**
- `fact_train_stops` — central fact table with one row per train stop event, storing planned and actual arrival/departure times, delays, and cancellation flags
- `dim_stations` — station dimension with EVA number, name, and geographic coordinates
- `dim_trains` — train dimension with train ID and category
- `dim_time` — time dimension with date, hour, and derived time attributes

**Sub-tasks:**
- **1.1** — Star schema design and table creation
- **1.2** — XML parsing and bulk ingestion of timetable and timetable change files into the schema; merges planned timetables with real-time change events to produce a unified fact table

---

### Task 2 — Data Analysis

SQL-based analysis on the PostgreSQL database built in Task 1.

- **2.1** — Retrieve metadata (EVA number, coordinates) for a given station by name (e.g. *Tiergarten*)
- **2.2** — Find the station closest to a given latitude/longitude using Euclidean distance in degree space (approximate, suitable for city-scale comparisons)
- **2.3** — Count the total number of cancelled train stop events for a specific date and hour
- **2.4** — Compute average arrival and departure delay in minutes per station, excluding cancelled events to avoid distorting delay statistics with missing time data

---

### Task 3 — Large-Scale ETL and Analysis in Spark

Reimplements the ETL pipeline at scale using Apache Spark connected to the Docker Spark cluster, then runs distributed analyses on the resulting dataset.

- **3.1** — Spark ETL job: reads raw XML timetable and change files using the `spark-xml` library, parses and transforms the data (timestamps, delays, cancellation flags), and writes the output to **Parquet** format for efficient downstream querying
- **3.2** — Computes **average daily arrival and departure delay** per station by reading the Parquet output and grouping by station and date
- **3.3** — Computes the **average number of departures during peak hours** (07:00–09:00 and 17:00–19:00) per station per day, excluding cancelled trains

---

## Project Structure

```
.
├── docker-compose.yaml               # PostgreSQL, pgAdmin, Spark master + worker
├── extract_files.ipynb               # Run once to unpack raw .tar.gz archives
└── path/to/your/code/
    ├── task1_Schema_Design_and_Data_Ingestion.ipynb
    ├── task2_Data_Analysis.ipynb
    └── task3_Large-Scale_ETL_and_Analysis_in_Spark.ipynb
```
