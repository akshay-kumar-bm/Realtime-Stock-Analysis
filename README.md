# Realtime-Stock-Analysis

A Flask web app that screens the Nifty 200 stocks with trend indicators (EMA, MACD, Supertrend) and shortlists those showing positive momentum.

## What it does
For every company in `stock_data_nifty200.xlsx`, the app downloads daily price history from Yahoo Finance (`yfinance`), computes indicators, takes the latest row as a snapshot, and serves two tables in the browser: the full snapshot of all stocks and a filtered shortlist of momentum candidates. Results are cached to `data.xlsx` and can be refreshed on demand.

## Key features
- Daily data from 2000-08-01 to today for each Nifty 200 symbol (`yfinance`).
- Indicators (`ta` library plus a hand-written Supertrend): MACD histogram and its slope, EMA 50 and EMA 100 with percentage slopes and second differences, Supertrend (ATR period 10, multiplier 2), all-time-high flag, and a Supertrend-derived target price.
- Shortlist filter (`preprocess`): last candle green, EMA-50 slope positive and rising, MACD-histogram slope positive; sorted by EMA-50 slope descending.
- Background refresh every 900 seconds using APScheduler; last refresh time stored in `time.txt` (IST).
- Tables rendered from `data.xlsx`; the `/view` page has an "Export table to excel" button (client-side SheetJS).
- Elastic Beanstalk config provided (`.ebextensions/python.config`, WSGI path `application:application`).

## Routes
| Route | Behaviour |
|---|---|
| `/` | landing page |
| `/result` | recompute all indicators now (slow), then show tables |
| `/view` | show tables from cached `data.xlsx` with last-update time |

## Tech stack
Python, Flask, yfinance, ta, pandas, numpy, openpyxl, APScheduler, pytz; Jinja templates with static CSS; AWS Elastic Beanstalk config.

## Project structure
```
application.py            # Flask app, indicators, scheduler
stock_data_nifty200.xlsx  # symbol/company list (input)
data.xlsx, time.txt       # cached output and last refresh time
templates/                # index, result, view, demo
static/                   # css, background image, loader
.ebextensions/python.config
requirements.txt
```

## Run locally
```bash
pip install -r requirements.txt
python application.py          # serves on 0.0.0.0:5000
```
The first full computation takes a long time (the original README says about 15 minutes) because ~200 tickers are downloaded sequentially.

## Limitations
- Sequential downloads and a scheduler that re-downloads full history every 15 minutes (even on `/result`); no caching of raw prices.
- Despite the "real-time" title, data is daily bars refreshed on a timer, not streaming.
- Analytics are screening heuristics only; no backtest, no investment advice.
- `requirements.txt` includes `websocket`, `requests` and `datetime` which are not used by the code; versions are unpinned.
- Indicators rely on `yfinance` column layout, which has changed across versions.
