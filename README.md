# NIFTY 50 OHLC Data

Historical NIFTY 50 OHLC intraday data, including 1-minute candles from 2017 and 15-second candles from 04 September 2026.

## Data Structure

```
nifty/
├── 1min/          # 1-minute candlestick data
│   ├── 2017/      # Partial year (Apr 3 - Dec 31)
│   ├── 2018-2025/ # Full years
│   └── 2026/      # Partial year (Jan 1 - Sep 10)
├── 15sec/         # 15-second candlestick data
│   └── 2026/      # Partial year (Sep 4 - Sep 10)
└── README.md      # This file
```

## File Format

Each CSV file contains the following columns:

| Column | Description |
|--------|-------------|
| Epoch | Unix timestamp (seconds since 1970-01-01) |
| Timestamp | ISO 8601 format (YYYY-MM-DDTHH:MM:SS) |
| Open | Opening price for the candle |
| High | Highest price during the candle |
| Low | Lowest price during the candle |
| Close | Closing price for the candle |
