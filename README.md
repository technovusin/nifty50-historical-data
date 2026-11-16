# NIFTY50 OHLC Data

Historical NIFTY50 intraday OHLC data in CSV format. One-minute candles start on 3 April 2017; 15-second candles start on 4 September 2026. All Timestamp values are in Indian Standard Time (IST, UTC+05:30).

## Coverage

| Timeframe | Available from | Layout |
|-----------|----------------|--------|
| 1 minute | 3 April 2017 | Yearly files through 2025; monthly files from 2026 |
| 15 seconds | 4 September 2026 | Monthly files |

## Data Structure

```text
nifty50-historical-data/
├── 1min/                 # 1-minute CSV candles, grouped by year
├── 15sec/                # 15-second CSV candles, grouped by year
├── parquet/              # Generated Parquet files (not stored in git)
├── scripts/
│   └── build_parquet.sql # Builds Parquet files from CSVs
├── LICENSE
└── README.md
```

Files through 2025 use the name `NIFTY50_<timeframe>_<year>.csv`; monthly files use `NIFTY50_<timeframe>_<year>-<month>.csv`, for example `NIFTY50_1min_2026-09.csv`.

## CSV Format

| Column | Description |
|--------|-------------|
| `Epoch` | Unix timestamp in seconds |
| `Timestamp` | Candle start time in IST, formatted as `YYYY-MM-DDTHH:MM:SS` without a timezone offset |
| `Open` | Opening price for the candle |
| `High` | Highest price during the candle |
| `Low` | Lowest price during the candle |
| `Close` | Closing price for the candle |
| `Volume` | Present in 2026 one-minute CSV files only |

`Timestamp` is a local wall-clock time, so localize it to `Asia/Kolkata` when using timezone-aware tools. `Epoch` identifies the same instant as a Unix timestamp. `Volume` is the index volume calculated by aggregating the traded volumes of all the index's constituent stocks.

## Parquet Files

Parquet files are generated locally and are not stored in git. Install the [DuckDB CLI](https://duckdb.org/docs/installation/) and run this command from the repository root:

```sh
duckdb < scripts/build_parquet.sql
```

In Windows PowerShell, use one of the following instead:

```powershell
duckdb -c ".read scripts/build_parquet.sql"
# Or:
Get-Content scripts/build_parquet.sql -Raw | duckdb
```

Re-run the script whenever the CSVs change. It creates these files in `parquet/`:

| File | Timeframe and columns |
|------|-----------------------|
| `NIFTY50_15sec.parquet` | 15-second OHLC |
| `NIFTY50_01min.parquet` | 1-minute OHLC, plus `volume` (NULL before 2026) |
| `NIFTY50_02min.parquet`, `NIFTY50_03min.parquet`, `NIFTY50_05min.parquet`, `NIFTY50_10min.parquet`, `NIFTY50_15min.parquet`, `NIFTY50_30min.parquet`, `NIFTY50_60min.parquet` | OHLC resampled from one-minute data |

All files contain `epoch` (BIGINT), `ts` (TIMESTAMP, IST wall time), and `open`, `high`, `low`, `close` (DOUBLE). The one-minute file also contains `volume` (BIGINT). The Parquet timestamp has no timezone attached.

Resampled buckets are aligned to 09:15 IST, and `ts` is the bucket start. Open and close come from the first and last available source candles; high and low are their maximum and minimum. Missing minutes are not filled, and incomplete buckets are retained. A resampled `epoch` is the earliest source candle's epoch, so it can be later than `ts` if the first minute of a bucket is missing. Duplicate epochs are removed when building the source Parquet files.

```sql
-- Query a generated file with DuckDB.
SELECT *
FROM 'parquet/NIFTY50_05min.parquet'
WHERE ts >= '2024-01-01';
```

## Adding Data or Reporting Issues

1. Add a CSV to the matching timeframe and year folder, using the same header as the existing files for that timeframe and period.
2. Rebuild the Parquet files using the command above.
3. Report suspected gaps or corrections through the repository's issue tracker, including the timeframe, date and supporting details.

## SENSEX OHLC Data

For SENSEX index OHLC data, see [sensex-historical-data](https://github.com/technovusin/sensex-historical-data).
