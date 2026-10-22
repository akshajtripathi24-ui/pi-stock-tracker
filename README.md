# 📈 Pi Stock Tracker

A web app that runs on a Raspberry Pi and tracks every US-listed stock (about 7,000). You only need the Pi. No sensors or add-on boards.

Open it from any phone or computer on your home Wi-Fi.

![Pi Stock Tracker market page with live index cards and the top gainers and losers](docs/screenshot.png)

## Features

- **Market page:** S&P 500, Nasdaq 100, Dow and Russell 2000, top gainers and losers (1D, 1W, 1M, 6M, YTD and 1Y), the most active stocks, and today's sector performance.
- **Screener:** all US stocks in one table. Filter by name, sector or company size, and sort by any column.
- **Search:** by company name, ticker or CEO name.
- **Stock pages:** an interactive chart from 1 day to the stock's whole history, performance, key stats, and a company profile (CEO, employees, headquarters, website).
- **Live prices (optional):** with a free [Finnhub](https://finnhub.io) API key, the stock you're viewing, what's on screen and your favorites update with every trade and flash green or red.
- **★ Favorites**, **light and dark mode**, and a layout that works on phones.

## What you need

- A Raspberry Pi (tested on a Pi 4 with 4GB running Raspberry Pi OS Bullseye, Python 3.9) connected to the internet
- Optional: a free API key from [finnhub.io](https://finnhub.io) for live prices

## Install

On the Pi:

```bash
mkdir -p ~/stocktracker && cd ~/stocktracker
python3 -m venv venv
source venv/bin/activate
pip install requests websocket-client
```

Copy `server.py`, `index.html` and `setup-autostart.sh` from this repo into `~/stocktracker`. Then start it:

```bash
python server.py
```

Open `http://<your-pi-ip>:8000` in a browser. To find the IP address, run `hostname -I` on the Pi.

### Live prices (optional)

1. Sign up at [finnhub.io](https://finnhub.io) and copy the **API key** from your dashboard. Don't use the "Secret", which is for webhooks.
2. On the Pi, save the key in a file called `finnhub_key.txt` next to `server.py`. The file should contain only the key:
   ```bash
   nano finnhub_key.txt          # paste the key, then Ctrl+O, Enter, Ctrl+X
   chmod 600 finnhub_key.txt     # make it private
   ```
3. Restart the tracker. The badge in the top bar shows **● LIVE** while the market is open.

`finnhub_key.txt` is listed in `.gitignore`, so it won't be committed to GitHub by accident. Keep your key private.

### Run it all the time

```bash
bash setup-autostart.sh
```

The tracker then runs in the background, starts whenever the Pi boots, and restarts if it crashes.

| To do this | Run |
|---|---|
| See the log | `journalctl -u stocktracker -f` |
| Restart (e.g. after changing the key) | `sudo systemctl restart stocktracker` |
| Stop | `sudo systemctl stop stocktracker` |
| Turn off auto-start | `sudo systemctl disable --now stocktracker` |

## Where the data comes from

| Data | Source | How often |
|---|---|---|
| Prices, sectors and market cap for all US stocks | Nasdaq's public stock screener | every minute |
| Charts, 1-year history and company profiles | Yahoo Finance | when you open a stock; history slowly in the background |
| Live trades (optional) | Finnhub WebSocket | as they happen, up to 45 stocks at once |

The first time it starts, the Pi spends about 2–3 hours slowly downloading a year of history for every stock, biggest companies first. Until then, the 1W, 1M and 1Y columns fill in gradually. The history is saved to `history.json`, so restarting doesn't lose it.

> **Personal use only.** The free data sources used here are meant for personal use. Run your own copy for yourself. Don't put it on the public internet. Nothing here is investment advice.

## Troubleshooting

| Problem | Fix |
|---|---|
| `Finnhub error: Subscribing to too many symbols` | Only run **one** copy per API key. Don't run `python server.py` while the background service is also running. |
| Badge says **● Connecting…** | Hover over it to see why. "Finnhub rejected the API key" means `finnhub_key.txt` is wrong. |
| Badge says **● Live (market quiet)** | That's normal outside US market hours (9:30am–4pm Eastern, Mon–Fri). |
| `Unit stocktracker.service not found` | Run `bash setup-autostart.sh` first. |
| `pip install` fails on an old Pi OS | This project only needs `requests` and `websocket-client`, which install fine on Bullseye. |
| Pi running hot | Check with `vcgencmd measure_temp`. With a small fan on pins 4 (5V) and 6 (GND), a Pi 4 stays around 35–50°C. |

## How it works

- `server.py`: a small Python web server, using only the standard library plus `requests` and `websocket-client`. It downloads the stock list, crawls history in the background, streams live trades and serves a JSON API.
- `index.html`: the whole website in one file (HTML, CSS and JavaScript). It uses [Chart.js](https://www.chartjs.org/) for charts.

## License

[MIT](LICENSE). You can use, change and share this project, as long as you keep the copyright notice.
