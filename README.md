# Chennai GTFS Transport Analytics

## 1. Project overview

This project downloads Chennai General Transit Feed Specification (GTFS) data, loads the raw files into PostgreSQL, creates a cleaned data layer, and connects the cleaned tables to Power BI for analysis.

The project answers five operational questions:

1. Which routes have the highest number of scheduled trips?
2. Which stops have the highest scheduled arrivals and departures?
3. At what hours is scheduled service highest or lowest?
4. Which routes run the longest scheduled journeys?
5. How many stops does each route serve?

## 2. Architecture

```text
GitHub GTFS ZIP
      |
      v
Data loading script
      |
      v
gtfs_chennai schema
(raw PostgreSQL tables)
      |
      v
ETL_transport.py
(cleaning and validation)
      |
      v
gtfs_chennai_clean schema
(clean PostgreSQL tables)
      |
      v
Power BI
(data model, DAX measures, visuals)
```

## 3. Source data

The data-loading script downloads the Chennai GTFS ZIP file from GitHub and reads these GTFS files:

- `stops.txt`: stop identifiers, names, and coordinates.
- `routes.txt`: route identifiers and route names.
- `trips.txt`: scheduled trips assigned to routes and service IDs.
- `stop_times.txt`: arrival times, departure times, and stop sequence for each trip.
- `calendar.txt`: recurring service-day patterns and validity dates.

The source data is loaded into the PostgreSQL schema:

```text
gtfs_chennai
```

The raw tables are:

```text
gtfs_chennai.stops
gtfs_chennai.routes
gtfs_chennai.trips
gtfs_chennai.stop_times
gtfs_chennai.calendar
```

## 4. Requirements

Install Python and PostgreSQL before running the project.

Install the Python packages from the VS Code terminal:

```bash
python -m pip install pandas requests sqlalchemy psycopg2-binary python-dotenv
```

On Windows, if `python` is not recognized, use:

```bash
py -m pip install pandas requests sqlalchemy psycopg2-binary python-dotenv
```

## 5. Environment configuration

Create a file named `.env` in the same folder as the Python scripts:

```env
DB_USER=postgres
DB_PASSWORD=your_postgresql_password
DB_HOST=localhost
DB_PORT=5432
DATABASE_NAME=transport

SOURCE_SCHEMA=gtfs_chennai
CLEAN_SCHEMA=gtfs_chennai_clean
```

Do not commit `.env` to GitHub. Add it to `.gitignore`:

```text
.env
.venv/
```

The scripts use `python-dotenv` to load these values. The database password should not be hard-coded in the Python source files.

## 6. Data-loading process

The data-loading script performs these steps:

1. Downloads the GTFS ZIP file with `requests`.
2. Opens the ZIP archive in memory.
3. Reads the five GTFS text files with pandas.
4. Displays the files found and the shape of each table.
5. Connects to PostgreSQL.
6. Creates the `gtfs_chennai` schema if it does not exist.
7. Writes the raw GTFS DataFrames to PostgreSQL.
8. Lists the tables created in the raw schema.

The original loading script uses:

```python
if_exists="replace"
```

This replaces the raw tables every time the script runs. That is appropriate when the purpose is to refresh the raw layer with a newly downloaded GTFS feed. If preservation of existing raw tables is required, change the behavior to `fail` or write each download into a dated schema or staging table.

## 7. ETL and cleaning process

`ETL_transport.py` performs the following operations:

1. Loads database configuration from `.env`.
2. Checks whether the `transport` database exists.
3. Creates the database if it does not exist.
4. Connects to the existing or newly created database.
5. Reads the raw tables from `gtfs_chennai`.
6. Creates the `gtfs_chennai_clean` schema if it does not exist.
7. Standardizes text values.
8. Removes duplicate stops, routes, trips, calendar records, and stop-time rows.
9. Removes rows missing required identifiers.
10. Validates stop-time formats.
11. Validates stop latitude and longitude.
12. Validates relationships between GTFS tables.
13. Creates cleaned PostgreSQL tables.
14. Verifies the tables in the clean schema.

The cleaned tables are:

```text
gtfs_chennai_clean.stops_clean
gtfs_chennai_clean.routes_clean
gtfs_chennai_clean.trips_clean
gtfs_chennai_clean.stop_times_clean
gtfs_chennai_clean.calendar_clean
```

The ETL script uses:

```python
if_exists="fail"
```

This prevents existing clean tables from being overwritten. Existing tables are skipped and missing tables are created.

## 8. Recommended database model

For Power BI, use a star-schema-style model where the operational tables filter the trip and stop-time tables.

Recommended relationships:

```text
routes_clean[route_id]       1 ---- * trips_clean[route_id]
calendar_clean[service_id]   1 ---- * trips_clean[service_id]
trips_clean[trip_id]         1 ---- * stop_times_clean[trip_id]
stops_clean[stop_id]         1 ---- * stop_times_clean[stop_id]
```

Recommended relationship directions are generally from the one-side dimension table to the many-side fact table.

Use the separate clean tables for modelling. The combined table is useful for exploration, validation, and simple report creation, but using every field from one very wide combined table can create ambiguity and duplicate-grain problems.

## 9. Suggested Power BI measures

Create measures in Power BI rather than relying on implicit sums for the business questions.


## 10. Run order

Run the scripts in this order:

### Step 1: Load raw GTFS data

Run the data-loading script. It downloads the ZIP and writes the raw tables to:

```text
gtfs_chennai
```

### Step 2: Run ETL cleaning

Run `ETL_transport.py`. It reads the raw tables and writes the clean tables to:

```text
gtfs_chennai_clean
```

### Step 3: Connect Power BI

In Power BI Desktop:

1. Select **Get data → PostgreSQL database**.
2. Enter the server and database details.
3. Select the clean schema tables.
4. Create relationships in Model view.
5. Create DAX measures.
6. Build the report visuals.

## 11. Reproducibility and safety

- Keep credentials in `.env`.
- Add `.env` to `.gitignore`.
- Keep raw and clean schemas separate.
- Use validation summaries to document removed or unmatched records.
- Avoid `if_exists="replace"` for clean tables unless a refresh is intentional.
- Use database backups before changing table-writing behavior.
- Record the GTFS download date because feeds may change over time.

## 12. Summary

This project creates a complete pipeline from public Chennai GTFS data to a Power BI analytics model:

```text
Download
  -> Load
  -> Validate
  -> Clean
  -> Store
  -> Model
  -> Measure
  -> Visualize
  -> Support decisions
```

The final Power BI report can show route supply, stop activity, hourly service patterns, journey duration, and route coverage while clearly distinguishing scheduled service from actual operational or passenger data.
