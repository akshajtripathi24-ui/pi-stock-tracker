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
- **🔔 Price alerts:** get a notification on your phone when a stock rises above or falls below a price, or moves more than a set % in a day. Uses the free [ntfy](https://ntfy.sh) app, with no account needed.
- **★ Favorites**, **light and dark mode**, and a layout that works on phones.

## What you need

- A Raspberry Pi (tested on a Pi 4 with 4GB running Raspberry Pi OS Bullseye, Python 3.9) connected to the internet
- Optional: a free API key from [finnhub.io](https://finnhub.io) for live prices

## Install

On the Pi, download the project and install the two Python packages it needs:

```bash
sudo apt install -y git
git clone https://github.com/akshajtripathi24-ui/pi-stock-tracker.git ~/stocktracker
cd ~/stocktracker
python3 -m venv venv
venv/bin/pip install requests websocket-client
```

Then make it run in the background and start whenever the Pi boots:

```bash
bash setup-autostart.sh
```

Open `http://<your-pi-ip>:8000` in a browser on any device on the same Wi-Fi. To find the IP address, run `hostname -I` on the Pi.

### Update to the latest version

```bash
cd ~/stocktracker && git pull && sudo systemctl restart stocktracker
```

Then refresh the website with **Ctrl+F5**. Your Finnhub key, alerts and downloaded history are kept.

<details>
<summary>Installed it before by copying the files by hand? Link the folder to GitHub once</summary>

```bash
cd ~/stocktracker
sudo apt install -y git
git init -q
git remote add origin https://github.com/akshajtripathi24-ui/pi-stock-tracker.git
git fetch origin
git checkout -f -B main origin/main
git branch -u origin/main
git config pull.ff only
sudo systemctl restart stocktracker
```

This replaces the code files with the latest version and keeps `venv`, `finnhub_key.txt`, `alerts.json` and the history files. If you never ran `setup-autostart.sh`, run `bash setup-autostart.sh` instead of the last line.
</details>

### Live prices (optional)

1. Sign up at [finnhub.io](https://finnhub.io) and copy the **API key** from your dashboard. Don't use the "Secret", which is for webhooks.
2. On the Pi, save the key in a file called `finnhub_key.txt` next to `server.py`. The file should contain only the key:
   ```bash
   nano finnhub_key.txt          # paste the key, then Ctrl+O, Enter, Ctrl+X
   chmod 600 finnhub_key.txt     # make it private
   ```
3. Restart the tracker with `sudo systemctl restart stocktracker`. The badge in the top bar shows **● LIVE** while the market is open.

`finnhub_key.txt` is listed in `.gitignore`, so it won't be committed to GitHub by accident. Keep your key private.

### Controlling the background service

After `bash setup-autostart.sh`, the tracker runs in the background, starts whenever the Pi boots, and restarts if it crashes. You don't need to run `python server.py` yourself.

| To do this | Run |
|---|---|
| See the log | `journalctl -u stocktracker -f` |
| Restart (e.g. after changing the key) | `sudo systemctl restart stocktracker` |
| Stop | `sudo systemctl stop stocktracker` |
| Turn off auto-start | `sudo systemctl disable --now stocktracker` |
| Run it by hand instead (stop the service first) | `cd ~/stocktracker && source venv/bin/activate && python server.py` |

### Price alerts on your phone

1. Install the free **ntfy** app from the App Store or Google Play ([ntfy phone app guide](https://docs.ntfy.sh/subscribe/phone/)).
2. Open the tracker's **🔔 Alerts** page. It shows a random topic name like `pi-stocks-1a2b3c4d5e6f7a8b`. In the app, tap **+** and subscribe to that topic.
3. Tap **Send test notification** to check it works.
4. Open any stock and use the **🔔 Price alerts** box to add an alert.

The Pi checks alerts every 10 seconds. A price alert fires once each time the price crosses your level, and a "% move" alert fires at most once a day. Alerts are saved in `alerts.json` on the Pi, which is in `.gitignore`. Keep your topic name private: anyone who knows it could read your alerts.

### Use it from anywhere, privately (Tailscale)

[Tailscale](https://tailscale.com) is a free app that connects your own devices to each other, so you can open the tracker away from home without making it public. Ask a parent if you need help creating the account.

On the Pi:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

`sudo tailscale up` prints a login link. Open it and sign in. Then install the **Tailscale** app on your phone and sign in with the **same account**.

On your phone, with Tailscale switched on, open `http://raspberrypi:8000`. If that doesn't work, run `tailscale ip -4` on the Pi and open `http://<that-ip>:8000`. Tailscale starts by itself when the Pi boots.

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
| `bash: python: command not found` | Use the background service (`bash setup-autostart.sh`). To run it by hand, first `cd ~/stocktracker && source venv/bin/activate`, then the line should start with `(venv)`. |
| `git pull` shows orange "hint:" messages | Harmless. Run `git config pull.ff only` once to hide them. |
| Badge says **● Connecting…** | Hover over it to see why. "Finnhub rejected the API key" means `finnhub_key.txt` is wrong. |
| Badge says **● Live (market quiet)** | That's normal outside US market hours (9:30am–4pm Eastern, Mon–Fri). |
| `Unit stocktracker.service not found` | Auto-start isn't set up yet. Run `cd ~/stocktracker && bash setup-autostart.sh`. |
| `pip install` fails on an old Pi OS | This project only needs `requests` and `websocket-client`, which install fine on Bullseye. |
| No alert notifications | Check the topic in the ntfy app matches the Alerts page exactly, then tap **Send test notification**. |
| `http://raspberrypi:8000` doesn't open away from home | Make sure Tailscale is switched on in the phone app, or use the IP from `tailscale ip -4`. |
| Pi running hot | Check with `vcgencmd measure_temp`. With a small fan on pins 4 (5V) and 6 (GND), a Pi 4 stays around 35–50°C. |

## How it works

- `server.py`: a small Python web server, using only the standard library plus `requests` and `websocket-client`. It downloads the stock list, crawls history in the background, streams live trades and serves a JSON API.
- `index.html`: the whole website in one file (HTML, CSS and JavaScript). It uses [Chart.js](https://www.chartjs.org/) for charts.

## License

[MIT](LICENSE). You can use, change and share this project, as long as you keep the copyright notice.
