# Stock Analysis Pipeline

A Python service that screens equities, scores them, backtests the signals, stores results in
Airtable, and fires alerts by email and Zapier webhook. A Nuxt frontend provides the run
history, live logs, and configuration screens.

## What it does

1. Pulls market data from Polygon.io and Yahoo Finance, and pulls screeners from TradingView.
2. Scores candidates against configurable rules and applies technical filters with ATR and
   volume thresholds.
3. Backtests the rules against historical data and records the results.
4. Pushes runs, scans, and alerts into Airtable over its REST API.
5. Sends alerts through a Zapier webhook and by email.
6. Exposes a FastAPI server with a `/health` endpoint for the deploy platform.

## Stack

| Layer | Technology |
|---|---|
| API | FastAPI, uvicorn, APScheduler |
| Data | pandas, numpy, yfinance, Polygon.io |
| Screening | tradingview-screener, Beautiful Soup, lxml, tenacity |
| Storage | Airtable REST API, JSON run artefacts |
| Alerting | Zapier webhook, SMTP email |
| Frontend | Nuxt 3, Vue, TypeScript |
| Packaging | Docker (python:3.11-slim), Railway |

## Layout

```
backend/
  server.py              FastAPI app and routes
  main.py                pipeline entry point and scheduling
  market_data.py         Polygon and Yahoo fetchers
  scraper.py             HTML scraping helpers
  technical_scanner.py   ATR and volume rule engine
  scorer.py              scoring rules
  backtester.py          historical backtest engine
  airtable_push.py       Airtable REST writer
  alerts.py              Zapier and email delivery
  health_check.py        service health probe
  test_*.py              six test modules
frontend/                Nuxt 3 UI: runs, live logs, config
scripts/                 shell runners for the pipeline and scanner
Dockerfile               backend image
railway.json             two service deploy definition
```

## Run it

```bash
cp .env.example .env
# fill in the keys, see the table below

pip install -r backend/requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8000
```

Frontend:

```bash
cd frontend
npm install
npm run dev
```

Tests:

```bash
cd backend
python -m pytest test_pipeline.py test_technical_scanner.py test_alerts.py
```

## Configuration

All secrets come from the environment. Copy `.env.example` and fill in your own values. Never
commit a filled `.env`.

| Variable | Purpose |
|---|---|
| `AIRTABLE_TOKEN` | Airtable personal access token |
| `AIRTABLE_BASE_ID` | Target base |
| `AIRTABLE_TABLE_*` | Table names for scans, runs, and alerts |
| `POLYGON_API_KEY` | Market data feed |
| `ZAPIER_WEBHOOK_URL` | Alert destination |
| `ALERT_EMAIL_TO` | Alert recipient |
| `NUXT_PUBLIC_API_BASE` | API base URL used by the frontend |
| `MIN_SCAN_SCORE`, `V1_*`, `V2_*`, `REPEAT_BUY_DAYS` | Scoring and filter thresholds |

## Deploy

The Dockerfile builds the backend and starts uvicorn under the platform's `$PORT`. `railway.json`
defines the backend service with a `/health` health check and the frontend as a second service.

## License

MIT. See [LICENSE](LICENSE).
