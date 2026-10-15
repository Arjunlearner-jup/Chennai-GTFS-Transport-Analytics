# Chennai GTFS Data Loading and Master Dataset

## 1. Dataloading.ipynb

Downloads the GTFS archive, opens it in memory, reads its CSV tables as text, and writes them to PostgreSQL.

### Functions, methods, and constructors used

- `requests.get()`
- `response.raise_for_status()`
- `zipfile.ZipFile()`
- `io.BytesIO()`
- `print()`
- `z.namelist()`
- `read_gtfs_file()`
- `pd.read_csv()`
- `z.open()`
- `create_engine()`
- `engine.begin()`
- `conn.execute()`
- `text()`
- `tables.items()`
- `df.to_sql()`
- `len()`
- `engine.connect()`

### Source tables

| GTFS file | PostgreSQL table |
| :--- | :--- |
| `stops.txt` | `gtfs_chennai.stops` |
| `routes.txt` | `gtfs_chennai.routes` |
| `trips.txt` | `gtfs_chennai.trips` |
| `stop_times.txt` | `gtfs_chennai.stop_times` |
| `calendar.txt` | `gtfs_chennai.calendar` |

### Loading behaviour

- Reads CSV columns with `dtype=str`.
- Uses `on_bad_lines="warn"` when parsing files.
- Writes records in batches using `method="multi"` and `chunksize=10000`.
- Uses `if_exists="replace"`, so rerunning the loading step replaces existing source tables.
- Lists tables in the source schema after loading.

## 2. Data cleaning and master dset.ipynb

Loads configuration, establishes the database connection, reads the source tables, and writes them into a separate destination schema.

### Functions, methods, and constructors used

- `Path()`
- `.resolve()`
- `load_dotenv()`
- `get_required_env()`
- `os.getenv()`
- `.strip()`
- `RuntimeError()`
- `int()`
- `make_postgres_url()`
- `URL.create()`
- `ensure_database_exists()`
- `create_engine()`
- `.connect()`
- `.execute()`
- `text()`
- `.scalar()`
- `print()`
- `.replace()`
- `.dispose()`
- `.mappings()`
- `.one()`
- `read_table()`
- `pd.read_sql()`
- `len()`
- `.begin()`
- `.items()`
- `.to_sql()`
- `str()`
- `.lower()`
- `.fetchall()`

### Configuration

The supplied code expects these environment variables:

```dotenv
DB_USER=your_database_user
DB_PASSWORD=your_database_password
DB_HOST=localhost
DB_PORT=5432
DATABASE_NAME=transport
SOURCE_SCHEMA=gtfs_chennai
CLEAN_SCHEMA=gtfs_chennai_clean
```

`CLEAN_SCHEMA=gtfs_chennai_clean` is an example destination name; choose the schema required for your project.

### Destination tables

| Source table | Destination table |
| :--- | :--- |
| `stops` | `<CLEAN_SCHEMA>.stops_clean` |
| `routes` | `<CLEAN_SCHEMA>.routes_clean` |
| `trips` | `<CLEAN_SCHEMA>.trips_clean` |
| `stop_times` | `<CLEAN_SCHEMA>.stop_times_clean` |
| `calendar` | `<CLEAN_SCHEMA>.calendar_clean` |

### Writing behaviour

- Uses `if_exists="fail"` to avoid replacing existing destination tables.
- Handles an existing-table error by printing a skip message.
- Prints row counts for source tables and newly written destination tables.
- Lists tables in the destination schema after writing.

