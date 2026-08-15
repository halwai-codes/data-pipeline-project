# Crypto ETL Pipeline: API → Python → SQL Server

A lightweight, end-to-end ETL (Extract, Transform, Load) pipeline built as a first hands-on data engineering project. It pulls live cryptocurrency market data from a public API, cleans and reshapes it with pandas, and loads it into a SQL Server database as a growing time-series table.

## Overview

This project follows the classic ETL pattern used in real-world data platforms (Azure Data Factory, Databricks pipelines, etc.), scaled down into simple, readable Python scripts:

```
Extract  →  Raw JSON landing zone  →  Transform (pandas)  →  Load  →  SQL Server
```

Each stage is isolated in its own folder, so any part of the pipeline can be modified, tested, or swapped out independently.

## Architecture

```
┌─────────────┐     ┌───────────────┐     ┌──────────────┐     ┌────────────────┐
│  Public API │ --> │  raw_data/    │ --> │  Transform   │ --> │  SQL Server     │
│ (crypto     │     │  (untouched   │     │  (pandas –   │     │  crypto_prices  │
│  prices)    │     │  JSON dump)   │     │  clean/shape)│     │  table          │
└─────────────┘     └───────────────┘     └──────────────┘     └────────────────┘
```

- **Extract** — calls the API and saves the raw, untouched JSON response to disk first. This "landing zone" means a bug in the transform step never requires re-hitting the API.
- **Transform** — flattens the nested JSON into a clean, tabular pandas DataFrame with consistent types.
- **Load** — appends each run as a new snapshot into SQL Server, building a genuine time-series table you can query for trends over time.

## Tech Stack

- **Python 3.13**
- **pandas** — data cleaning and transformation
- **requests** — API extraction
- **SQLAlchemy + pyodbc** — SQL Server connectivity
- **SQL Server** (local instance, Windows Authentication)
- **ODBC Driver 17 for SQL Server**

## Project Structure

```
data-pipeline-project/
├── extract/
│   └── extract_api.py       # Pulls data from the API, saves raw JSON
├── transform/
│   └── transform.py         # Cleans and reshapes raw JSON into a DataFrame
├── load/
│   └── load_sql.py          # Loads the DataFrame into SQL Server
├── raw_data/                # Landing zone for raw JSON (gitignored)
├── config/
│   └── logging_config.py    # Shared logging setup
├── requirements.txt
└── README.md
```

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/YOUR_USERNAME/crypto-etl-pipeline.git
cd crypto-etl-pipeline
```

**2. Create and activate a virtual environment**
```bash
python -m venv .venv
.venv\Scripts\activate      # Windows
```

**3. Install dependencies**
```bash
pip install -r requirements.txt
```

**4. Set up SQL Server**

Create a local database (via SSMS or a query):
```sql
CREATE DATABASE DataPipelineDB;
```

Update the connection details in `load/load_sql.py` if your server name or driver differs:
```python
SERVER = r'YOUR_SERVER\INSTANCE_NAME'
DATABASE = 'DataPipelineDB'
DRIVER = 'ODBC Driver 17 for SQL Server'
```

## Usage

Run each stage in order:

```bash
python extract/extract_api.py
python transform/transform.py
python load/load_sql.py
```

`load_sql.py` will create the `crypto_prices` table automatically on first run, then append a new snapshot of coin prices every time it's executed.

## Example Output

| coin | price_usd | market_cap_usd | change_24h_pct | timestamp |
|---|---|---|---|---|
| bitcoin | 63039.00 | 1265237387918.64 | 0.1518 | 2026-08-15 23:22:02 |
| ethereum | 1883.02 | 227246121634.43 | 0.1571 | 2026-08-15 23:22:02 |
| solana | 75.46 | 43967335072.40 | -0.2086 | 2026-08-15 23:22:02 |
| dogecoin | 0.07 | 10883248582.13 | 0.2344 | 2026-08-15 23:22:02 |

## What This Project Demonstrates

- Structuring a pipeline using the Extract → Transform → Load pattern
- Working with nested JSON and flattening it for tabular storage
- Building a raw data landing zone for reproducibility and debugging
- Connecting Python to SQL Server via SQLAlchemy/pyodbc with Windows Authentication
- Appending time-series data instead of overwriting, to preserve historical snapshots

## Roadmap / Next Steps

- [ ] Add a CSV extractor as a second data source
- [ ] Add a web scraping extractor as a third data source
- [ ] Add error handling + retry logic for API rate limits
- [ ] Add scheduling (Windows Task Scheduler or Airflow) to run automatically
- [ ] Add unit tests for the transform logic
- [ ] Migrate to Azure SQL / Azure Data Factory as a next iteration

## License

MIT
