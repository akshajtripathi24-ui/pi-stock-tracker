# 2. Install the tracker

This guide downloads the tracker onto your Pi, makes it run in the background, and opens the website. It takes about 5 minutes.

Before you start, you should be connected to your Pi over SSH. See [1. Get your Raspberry Pi ready](01-prepare-the-pi.md).

---

## Step 1: Install git

Git downloads the project from GitHub and lets you update it later with one command.

```bash
sudo apt install -y git
```

## Step 2: Download the project

```bash
git clone https://github.com/akshajtripathi24-ui/pi-stock-tracker.git ~/stocktracker
cd ~/stocktracker
```

This creates a folder called `stocktracker` in your home folder with these files:

| File | What it is |
|---|---|
| `server.py` | The program that runs on the Pi: downloads prices, sends alerts, and serves the website |
| `index.html` | The website itself |
| `setup-autostart.sh` | A script that makes the tracker start by itself |
| `README.md`, `docs/` | These instructions |

## Step 3: Install the two Python packages it needs

```bash
python3 -m venv venv
venv/bin/pip install requests websocket-client
```

- The first line makes a **venv** ("virtual environment"): a private folder for this project's Python packages, so they don't clash with the rest of the Pi.
- The second line installs `requests` (for downloading data) and `websocket-client` (for live prices).

It's fine if pip prints a notice about upgrading pip. You can ignore it.

## Step 4: Make it run in the background

```bash
bash setup-autostart.sh
```

It asks for your password (because it uses `sudo`), then prints **`Done!`** and the address to open.

From now on the tracker:
- runs in the background, so you can close the SSH window
- starts by itself every time the Pi turns on
- restarts by itself if it ever crashes

**Don't** also run `python server.py` by hand. Two copies running at once causes problems, such as the "too many symbols" error with live prices.

## Step 5: Open the website

On any phone, tablet or computer on the **same Wi-Fi**, open a browser and go to:

```
http://192.168.1.143:8000
```

Use your Pi's IP address (from `hostname -I`) instead of `192.168.1.143`. Make sure it starts with `http://`, not `https://`, and ends with `:8000`.

**What happens the first time:**
1. For a few seconds the page says it's downloading all US stocks.
2. Then the Market page fills in with today's prices.
3. In the background, the Pi slowly downloads a year of price history for every stock, biggest companies first. This takes **about 2–3 hours**. The grey line at the top shows progress, like `loading price history 1,234/6,999`. Until it finishes, some 1W, 1M, 6M, YTD and 1Y numbers show `–`. The history is saved, so this only happens once.

## Step 6: Check it's running

```bash
journalctl -u stocktracker -f
```

You should see lines like:

```
[2026-10-22 11:29] Prices refreshed: 6999 stocks from Nasdaq
```

Press **Ctrl + C** to stop watching. The tracker keeps running.

**Optional next steps:**
- [3. Live prices](03-live-prices.md): prices that update with every trade
- [4. Price alerts](04-price-alerts.md): notifications on your phone
- [5. Use it away from home](05-tailscale.md) with Tailscale

---

## Controlling the background service

| To do this | Run |
|---|---|
| Check if it's running | `systemctl is-active stocktracker` (says `active` if it is) |
| See what it's doing (live log) | `journalctl -u stocktracker -f` (Ctrl + C to stop watching) |
| Restart it (after an update or changing a setting) | `sudo systemctl restart stocktracker` |
| Stop it | `sudo systemctl stop stocktracker` |
| Start it again | `sudo systemctl start stocktracker` |
| Turn off auto-start completely | `sudo systemctl disable --now stocktracker` |

### Running it by hand instead (for testing)

Only do this with the background service **stopped**:

```bash
sudo systemctl stop stocktracker
cd ~/stocktracker
source venv/bin/activate
python server.py
```

After `source venv/bin/activate`, the line starts with `(venv)`. Without it, the Pi says `python: command not found`. Press **Ctrl + C** to stop, and `sudo systemctl start stocktracker` to go back to the background service.

---

## Updating to the latest version

When the project on GitHub gets new features or fixes:

```bash
cd ~/stocktracker && git pull && sudo systemctl restart stocktracker
```

Then refresh the website with **Ctrl + F5** (or pull down to refresh on a phone).

Your Finnhub key, alerts, alert topic and downloaded history are **kept**. They're in files that git ignores.

> If `git pull` prints orange `hint:` messages, they're harmless. Run `git config pull.ff only` once to hide them.

### Installed it before by copying files by hand?

Run this **once** to link your folder to GitHub. Afterwards, `git pull` works as above.

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

This replaces the code files with the latest versions and keeps `venv`, `finnhub_key.txt`, `alerts.json` and the history files. If it says `Unit stocktracker.service not found`, run `bash setup-autostart.sh` instead of the last line.

---

## Files the tracker creates on your Pi

These are made while it runs. They're listed in `.gitignore`, so they never get uploaded to GitHub.

| File | What's in it | Private? |
|---|---|---|
| `finnhub_key.txt` | Your Finnhub API key (you create it, see guide 3) | 🔒 Yes, never share |
| `alerts.json` | Your alerts and your ntfy topic | 🔒 Yes |
| `history.json` | One year of price history for each stock | No |
| `profiles.json` | Company info (CEO, employees, website) | No |

## Uninstalling

```bash
sudo systemctl disable --now stocktracker
sudo rm /etc/systemd/system/stocktracker.service
sudo systemctl daemon-reload
rm -rf ~/stocktracker
```

---

## Problems?

| Problem | Fix |
|---|---|
| `Unit stocktracker.service not found` | Auto-start isn't set up. Run `cd ~/stocktracker && bash setup-autostart.sh`. |
| `bash: python: command not found` | Use the background service. Or run `source venv/bin/activate` first (see "Running it by hand"). |
| `fatal: destination path ... already exists` | You already have a `stocktracker` folder. Use the "Installed it before by copying files by hand?" steps above. |
| The website doesn't load | Check `systemctl is-active stocktracker` says `active`, your device is on the same Wi-Fi, and the address is `http://<pi-ip>:8000`. |
| The page says "Can't reach the Pi" | The tracker stopped. Run `sudo systemctl restart stocktracker` and check the log with `journalctl -u stocktracker -f`. |
| 1W/1M/1Y columns show `–` | The Pi is still downloading history (2–3 hours the first time). Watch the progress in the grey line at the top. |

Next: [3. Live prices →](03-live-prices.md)
