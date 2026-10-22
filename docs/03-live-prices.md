# 3. Live prices (optional)

Without this step, all prices update **every minute**. With a free Finnhub key, the stocks you're looking at update **with every trade**, and the price flashes green or red when it changes.

---

## What gets live prices

The free Finnhub plan streams up to 50 stocks at once. The tracker uses up to **45** (keeping a few spare) and picks them in this order:

1. The 4 funds behind the index cards (SPY, QQQ, DIA, IWM), so the S&P 500, Nasdaq 100, Dow and Russell 2000 cards move live
2. The stock page you're looking at
3. The other stocks on your screen (the gainers and losers lists, the screener page, your favorites)
4. Stocks you have price alerts on

Everything else still updates every minute. When you stop looking at a stock, its slot goes to something else after 30 seconds.

These 45 slots are **shared by every device** that has the website open. See [6. Use it on more devices](06-more-devices.md).

---

## Step 1: Get a free Finnhub API key

1. Go to <https://finnhub.io> and click **Get free API key**.
2. Sign up with your email. Ask a parent if you need help making an account.
3. After signing in, your **Dashboard** shows your **API key**: a long string of letters and numbers (it may be 20 or 40 characters).
   - Use the one labelled **API key**.
   - **Don't** use the **Secret** (webhook secret). That's for a different feature.

Treat the key like a password. **Don't post it online, send it in a chat, or put it on GitHub.**

## Step 2: Save the key on the Pi

Connect to the Pi with SSH, then:

```bash
cd ~/stocktracker
nano finnhub_key.txt
```

`nano` is a simple text editor that opens inside the window.

1. Paste your key. **Right-click** to paste in the Windows SSH window. The file should contain **only the key**: no spaces, no quotes, nothing else.
2. Save: press **Ctrl + O**, then **Enter**.
3. Exit: press **Ctrl + X**.

The file name must be exactly `finnhub_key.txt`. The key goes **inside** the file, not in its name.

Make the file private, so other users on the Pi can't read it:

```bash
chmod 600 finnhub_key.txt
```

Check it's there:

```bash
ls ~/stocktracker
```

You should see `finnhub_key.txt` next to `server.py`.

## Step 3: Restart the tracker

```bash
sudo systemctl restart stocktracker
journalctl -u stocktracker -f
```

You should see:

```
Live prices connected (Finnhub)
```

Press **Ctrl + C** to stop watching.

## Step 4: Check the badge on the website

Refresh the website. The badge at the top right shows the live status:

| Badge | What it means |
|---|---|
| **● LIVE** (green, pulsing) | Working. Prices update with every trade. |
| **● Live (market quiet)** | Connected, but no trades are happening. Normal outside US market hours: 9:30am–4pm Eastern, Monday–Friday, not on market holidays. |
| **● Connecting…** | Trying to connect. Hover over it (on a computer) to see why. If it says "Finnhub rejected the API key", the key in `finnhub_key.txt` is wrong. |
| **● Live off** | No key file was found, or it's empty. Live prices are off and everything updates every minute. |

When live prices are on, you'll also notice:
- the price **flashes green** when it goes up and **red** when it goes down
- a **pulsing green dot** next to the index card names
- on a stock page with the **1D** chart selected, the chart grows as new trades come in

---

## Changing or removing the key

- **Change it:** `nano ~/stocktracker/finnhub_key.txt`, replace the key, save, then `sudo systemctl restart stocktracker`.
- **Turn live prices off:** `rm ~/stocktracker/finnhub_key.txt`, then `sudo systemctl restart stocktracker`.

---

## Problems?

| Problem | Fix |
|---|---|
| `Finnhub error: Subscribing to too many symbols` | Two copies of the tracker are using the same key. Make sure you're **not** running `python server.py` by hand while the background service runs: `sudo systemctl stop stocktracker; pkill -f server.py; sudo systemctl start stocktracker`. If one copy still sees it, it streams fewer stocks automatically and tries more again after 5 minutes. |
| `Finnhub rejected the API key` | The key is wrong or has extra spaces. Copy the **API key** (not the Secret) again into `finnhub_key.txt`. |
| `Run: pip install websocket-client` | Run `~/stocktracker/venv/bin/pip install websocket-client`, then restart. |
| Badge stays on **Live (market quiet)** during market hours | Wait a minute, then refresh. Check the log with `journalctl -u stocktracker -f`. |
| A stock's live % looks very different from another website | Nasdaq's minute-by-minute numbers and Finnhub's live trades can occasionally disagree, especially for small or rarely traded stocks. Compare with a site like Google Finance. |

Next: [4. Price alerts →](04-price-alerts.md)
