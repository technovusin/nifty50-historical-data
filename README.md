# NIFTY50 OHLC Data

Historical NIFTY50 OHLC intraday data, including 1-minute candles from 2017 and 15-second candles from 04 September 2026. All times are IST.

## Data Structure

```
nifty/
├── 1min/                 # 1-minute candles (CSV)
│   ├── 2017/             # One file per year; partial year (Apr 3 - Dec 31)
│   ├── 2018-2025/        # One file per year
│   └── 2026/             # One file per month, from 01 January 2026 (includes Volume)
├── 15sec/                # 15-second candles (CSV)
│   └── 2026/             # One file per month, from 04 September 2026
├── parquet/              # Generated Parquet files (not in git, see below)
├── scripts/
│   └── build_parquet.sql # Builds the Parquet files from the CSVs
└── README.md             # This file
```

File names are `NIFTY50_<timeframe>_<year>.csv` for 2017-2025 and `NIFTY50_<timeframe>_<year>-<month>.csv` for 2026, for example `NIFTY50_1min_2026-09.csv`.

## File Format

Each CSV file contains the following columns:

| Column | Description |
|--------|-------------|
| Epoch | Unix timestamp (seconds since 1970-01-01) |
| Timestamp | ISO 8601 format (YYYY-MM-DDTHH:MM:SS), IST |
| Open | Opening price for the candle |
| High | Highest price during the candle |
| Low | Lowest price during the candle |
| Close | Closing price for the candle |
| Volume | 2026 1-minute files only |

## Parquet Files

Parquet files are not stored in git. Build them from the CSVs with [DuckDB](https://duckdb.org) (run from the repo root):

```
duckdb < scripts/build_parquet.sql
```

On Windows PowerShell, the `<` redirect isn't supported, so use either of these instead (they also work in Command Prompt and Linux/macOS shells):

```
duckdb -c ".read scripts/build_parquet.sql"
Get-Content scripts/build_parquet.sql -Raw | duckdb
```

This creates the following files in `parquet/`, a columnar format for fast backtesting. Re-run it whenever the CSVs change:

| File | Timeframe |
|------|-----------|
| `NIFTY50_15sec.parquet` | 15 seconds (from 04 Sep 2026) |
| `NIFTY50_01min.parquet` | 1 minute (includes `volume`, NULL before 2026) |
| `NIFTY50_02min`, `03min`, `05min`, `10min`, `15min`, `30min`, `60min` | Resampled from 1-minute data |

Columns are `epoch` (BIGINT), `ts` (TIMESTAMP, IST), `open`, `high`, `low`, `close` (DOUBLE). Resampled bars start at the 09:15 session open, and `ts` is the bar start time.

```sql
-- DuckDB
SELECT * FROM 'parquet/NIFTY50_05min.parquet' WHERE ts >= '2024-01-01';
```

## Data Cleaning

The CSVs and Parquet files have been cleaned:

- Bars outside 09:15-15:29 IST removed, except the evening Diwali muhurat sessions (12 Nov 2023, 1 Nov 2024).
- Duplicate bars with stray seconds in the timestamp (Aug-Sep 2021) removed, keeping the on-the-minute bar.
- Three bad first candles in 2022 fixed (25 Mar, 30 Mar, 7 Apr): High/Low adjusted to include Open.
- Removed suspicious partial or spurious sessions: 7 Aug 2021, 21 Aug 2021, 13 Nov 2021, 8 Jan 2022, 22 Jan 2022, 5 Feb 2022, 9 Apr 2022, 30 Apr 2022.
- Prices rounded to 2 decimals; duplicate epochs removed.

Known remaining quirks: some single minutes are missing (374 bars on a few days), a few frozen-price stretches exist (e.g. 24 Feb 2021 NSE outage, March 2020 circuit breakers), 4 Nov 2021 has only 43 bars, and 9 Sep 2026 has 3 missing 15-second bars. Weekend rows are real special sessions (budget days, 20 Jan 2024, 2 Mar and 18 May 2024).

## Adding New Data

1. Add the new CSV to the matching folder (for 2026, the current month's file), with the same header as the existing files.
2. Rebuild the Parquet files with the command above. Duplicate epochs across files are removed automatically.

## SENSEX OHLC Data

If you are looking for SENSEX index OHLC data, it can be downloaded at [https://github.com/technovusin/sensex-historical-data](https://github.com/technovusin/sensex-historical-data) 
