# 10. How it works

For anyone who wants to understand or change the code.

---

## The big picture

```
                     ┌──────────────── Raspberry Pi ─────────────────┐
Nasdaq screener ───▶ │ all ~7,000 prices, every minute               │
Yahoo Finance   ───▶ │ charts, 1-year history, company profiles      │──▶ browsers (index.html)
Finnhub (opt.)  ───▶ │ live trades for up to 45 stocks               │      on any device
                     │ price alerts, checked every 10 s              │──▶ ntfy.sh ──▶ phones
                     └───────────────────────────────────────────────┘
```

There are only two code files:

| File | What it does |
|---|---|
| `server.py` | A Python web server using the standard library plus `requests` and `websocket-client`. It downloads data, streams live trades, checks alerts, and serves the website and a JSON API on port 8000. |
| `index.html` | The whole website in one file (HTML, CSS and JavaScript). It uses [Chart.js](https://www.chartjs.org/) for charts. |

And one helper: `setup-autostart.sh` creates a **systemd** service (`/etc/systemd/system/stocktracker.service`) so the tracker starts on boot and restarts if it crashes.

## What runs inside `server.py`

`server.py` runs several jobs at the same time (threads):

| Job | What it does | How often |
|---|---|---|
| **Prices** | Downloads every US stock's price, change, market cap, sector and volume from Nasdaq's public screener in one request. If that fails on the first start, it falls back to Nasdaq's symbol lists (names only). | Every 60 seconds |
| **History crawl** | Downloads one year of daily prices from Yahoo for each stock, biggest companies first, plus the company profile (CEO, employees, website). It saves to `history.json` and `profiles.json` so it resumes after restarts, and pauses 2 minutes if Yahoo says to slow down. | One stock every ~0.6 s; each stock refreshed about once a day |
| **Live prices** | Keeps a WebSocket open to Finnhub and subscribes to the stocks people are looking at: index funds first, then what's on screen, then alert stocks. Drops stocks 30 s after nobody looks at them, and reconnects automatically. | Continuous |
| **Alerts** | Checks every alert against the latest price and sends notifications through ntfy.sh | Every 10 seconds |
| **Web server** | Serves `index.html` and the API below | On request |

## How the % changes are worked out

- **1D:** from Nasdaq's numbers. With live prices, it's `live price ÷ yesterday's close − 1`, where yesterday's close = Nasdaq price − Nasdaq change.
- **1W, 1M, 6M:** compared with the close 5, 21 and 126 trading days ago, from the saved history.
- **YTD:** compared with the last close of the previous year.
- **1Y:** compared with the first close in the 1-year history.

## The API

All endpoints return JSON. Only `/api/alerts` accepts POST, and only with `Content-Type: application/json`, so other websites can't change your alerts from someone's browser.

| Endpoint | Returns |
|---|---|
| `GET /` | The website (`index.html`) |
| `GET /api/status` | Number of stocks, last update, history progress, live status |
| `GET /api/screener?q=&sector=&mincap=&maxcap=&sort=&order=&offset=&limit=&symbols=` | Filtered and sorted stocks (max 200 per page) |
| `GET /api/sectors` | Today's change per sector, weighted by market cap |
| `GET /api/market` | The four index cards |
| `GET /api/search?q=` | Up to 15 matches by ticker, name or CEO |
| `GET /api/quote?symbol=` | One stock: price, changes, stats and profile |
| `GET /api/chart?symbol=&range=` | Chart points for `1d`, `5d`, `1m`, `6m`, `ytd`, `1y`, `5y` or `max` |
| `GET /api/live?symbols=` | Latest prices for the stocks on screen. Also tells the Pi which stocks to stream. |
| `GET /api/alerts?symbol=` | Your alerts, recent notifications and the ntfy topic |
| `POST /api/alerts` | `{"action": "add", "symbol", "kind": "above"\|"below"\|"move", "value"}`, `{"action": "delete", "id"}` or `{"action": "test"}` |

## Settings you can change

Near the top of `server.py`:

| Setting | Default | Meaning |
|---|---|---|
| `PORT` | `8000` | The website's port |
| `SCREENER_REFRESH` | `60` | Seconds between full price downloads |
| `LIVE_MAX` | `45` | Max stocks to stream live (Finnhub's free limit is 50) |
| `LIVE_IDLE` | `30` | Seconds before an unwatched stock stops streaming |
| `CRAWL_DELAY` | `0.6` | Seconds between history downloads |
| `HISTORY_MAX_AGE` | `20 * 3600` | Re-download a stock's history after this many seconds |
| `ALERT_CHECK` | `10` | Seconds between alert checks |

After changing one, run `sudo systemctl restart stocktracker`. Note that `git pull` may conflict with your changes; run `git stash` before pulling and `git stash pop` after.

## Files the Pi creates

| File | Contents | In git? |
|---|---|---|
| `finnhub_key.txt` | Your Finnhub API key | ❌ Ignored, private |
| `alerts.json` | Alerts, notification log, ntfy topic | ❌ Ignored, private |
| `history.json` | Reference prices for each stock | ❌ Ignored |
| `profiles.json` | Company profiles | ❌ Ignored |
| `venv/` | Python packages | ❌ Ignored |

## Data sources and rules

| Data | Source |
|---|---|
| Prices for all US stocks | Nasdaq's public stock screener |
| Charts, history, company profiles, index values | Yahoo Finance |
| Live trades (optional) | [Finnhub](https://finnhub.io) WebSocket (free plan) |
| Phone notifications | [ntfy.sh](https://ntfy.sh) |

These free sources are meant for **personal use**. Run your own copy for yourself and your household, and don't put it on the public internet.
