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
- **Works on every device at once:** open it on your laptop, your phone and your family's phones. They all get live prices, with nothing to install. See [Use it on more devices](#use-it-on-more-devices-family-and-friends).

## What you need

- A Raspberry Pi (tested on a Pi 4 with 4GB running Raspberry Pi OS Bullseye, Python 3.9) connected to the internet
- Optional: a free API key from [finnhub.io](https://finnhub.io) for live prices
- Optional: the free **ntfy** app for price alerts ([iPhone](https://apps.apple.com/app/ntfy/id1625396347), [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy))
- Optional: the free **Tailscale** app to use it away from home ([iPhone](https://apps.apple.com/app/tailscale/id1470499037), [Android](https://play.google.com/store/apps/details?id=com.tailscale.ipn), [Windows, Mac and more](https://tailscale.com/download))

## Connect to your Pi (SSH)

You run the commands below on the Pi. The easiest way is from your computer over SSH, so you can copy and paste them:

1. On Windows, press the **Windows key**, type `cmd` and press **Enter**. On a Mac, open **Terminal**.
2. Connect, using your Pi's username (often `pi`) and IP address (run `hostname -I` on the Pi to find it). For example:
   ```
   ssh pi@192.168.1.143
   ```
3. Type the Pi's password. Nothing appears while you type, which is normal.

You're connected when you see `pi@raspberrypi:~ $`. In the Windows window, **right-click** to paste. Type `exit` to leave.

If SSH is turned off on the Pi, turn it on with `sudo systemctl enable --now ssh` (run on the Pi itself).

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

1. Install the free **ntfy** app. No account needed.
   - iPhone: [ntfy on the App Store](https://apps.apple.com/app/ntfy/id1625396347)
   - Android: [ntfy on Google Play](https://play.google.com/store/apps/details?id=io.heckel.ntfy)
2. Open the tracker's **🔔 Alerts** page. It shows a random topic name like `pi-stocks-1a2b3c4d5e6f7a8b`. Click **Copy**.
3. In the ntfy app, tap **+**, paste the topic **exactly**, and tap **Subscribe**. Allow notifications if asked.
4. On the Alerts page, tap **Send test notification**. Your phone should buzz within a few seconds.
5. Open any stock and use the **🔔 Price alerts** box to add an alert. For example, "moves more than **3**% in a day".

The Pi checks alerts every 10 seconds. A price alert fires once each time the price **crosses** your level, so an "above $1" alert on a $500 stock waits until the price goes below $1 and back up. A "% move" alert fires at most once a day. Alerts only fire while the market is open, because that's when prices move. Alerts are saved in `alerts.json` on the Pi, which is in `.gitignore`. Keep your topic name private: anyone who knows it could read your alerts.

### Use it from anywhere, privately (Tailscale)

[Tailscale](https://tailscale.com) is a free app that connects your own devices to each other, so you can open the tracker, and SSH into the Pi, away from home without making anything public. Only devices signed in to **your** account, or people you share the Pi with, can reach it. You need an account: sign in with GitHub, Google, Microsoft or Apple. Ask a parent if you need help creating it.

**1. On the Pi** (over SSH):

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

`sudo tailscale up` prints a login link. Open it on your computer, sign in, and click **Connect**. Then run `tailscale ip -4` and note the address. It starts with `100.`.

**2. On your phone:** install Tailscale, sign in with the **same account**, tap **Allow** when it asks to add a VPN configuration, and switch it on.
- iPhone: [Tailscale on the App Store](https://apps.apple.com/app/tailscale/id1470499037)
- Android: [Tailscale on Google Play](https://play.google.com/store/apps/details?id=com.tailscale.ipn)

**3. On your computer (optional, for SSH from anywhere):** install Tailscale from the [Tailscale download page](https://tailscale.com/download) and sign in with the same account.

**4. Test it:** turn your phone's Wi-Fi off (so you're on mobile data) and open `http://raspberrypi:8000`. If that doesn't load, use `http://100.x.x.x:8000` with the address from step 1. From your computer, `ssh pi@raspberrypi` works from anywhere too.

**5. Keep the Pi signed in:** Tailscale signs devices out after about 6 months by default. In the [Tailscale admin console](https://login.tailscale.com/admin/machines), click **⋯** next to `raspberrypi` and choose **Disable key expiry**.

Tailscale starts by itself when the Pi boots.

**"It says I'm on a VPN"**: your phone and computer call Tailscale a VPN, but it only carries traffic between your own devices. Everything else goes to the internet as normal. You don't have to keep it on:
- **Turn it off:** in the phone app, flip the switch off. On Windows, click the Tailscale icon by the clock and choose **Disconnect**. Turn it back on only when you want the tracker away from home. At home, just use `http://<your-pi-ip>:8000`.
- **Remove it:** delete the phone app. On Windows, go to **Settings → Apps → Installed apps → Tailscale → Uninstall**. On the Pi, run `sudo tailscale down && sudo apt remove -y tailscale`.

The tracker and your ntfy alerts keep working either way, because alerts don't need Tailscale.

### Use it on more devices (family and friends)

**You don't install anything for live prices.** There's no "stock tracker app". The Pi does all the work, and any browser that opens the website gets live updates:

```
Finnhub ──live trades──▶  Your Pi  ──website──▶  your laptop
(your key stays            (server.py)  ──website──▶  your phone
 on the Pi)                             ──website──▶  family phones…
```

The Pi is the only thing that talks to Finnhub. Every open page asks the Pi for new prices every second, so every device sees prices tick with the **● LIVE** badge. Nobody else needs your key or any account.

| What you want on the other device | What it needs |
|---|---|
| The website and live prices **at home** | Nothing. Open `http://<your-pi-ip>:8000` in any browser on the same Wi-Fi. |
| The website **away from home** | Tailscale (see the options below) |
| Alert notifications | The ntfy app, subscribed to the same topic ([iPhone](https://apps.apple.com/app/ntfy/id1625396347) · [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy)) |

**Tip:** in Safari, tap **Share → Add to Home Screen**. In Chrome, tap **⋮ → Add to Home screen**. The tracker then gets its own icon like an app.

#### Adding a device away from home: pick an option

| Option | How | Safe? | Use it for |
|---|---|---|---|
| **A. Your own Tailscale account** | Install Tailscale on the device and sign in **yourself** with your account. | ✅ Safe on **your own** devices | Your second phone, tablet or laptop |
| **B. Share just the Pi** | They make their **own** free Tailscale account, and you send them a share link for the Pi only. | ✅ **Safest** for other people | Family members and friends |
| **C. Give someone your login** | Tell them the email and password you use for Tailscale. | ❌ **Unsafe** | Never. That hands over your whole email account, not just the Pi. |
| **D. Make the Pi public** | Open router ports or use a public tunnel. | ❌ **Unsafe** | Never. Anyone on the internet could reach a computer in your home, and the free data is for personal use only. |

**Option A: your own devices**
1. Install Tailscale: [iPhone](https://apps.apple.com/app/tailscale/id1470499037) · [Android](https://play.google.com/store/apps/details?id=com.tailscale.ipn) · [Windows, Mac and more](https://tailscale.com/download)
2. Sign in with the **same** account the Pi uses, tap **Allow** for the VPN configuration, and switch it on.
3. Open `http://raspberrypi:8000`, or `http://100.x.x.x:8000` using the address from `tailscale ip -4` on the Pi.

This device can now reach **all** devices on your Tailscale account, so only use it on devices you own.

**Option B: family and friends (recommended for anyone else)**
1. **They** install Tailscale (links above) and create their **own** free account. Kids may need a parent's help.
2. **You** open the [Tailscale admin console](https://login.tailscale.com/admin/machines), click **⋯** next to `raspberrypi`, choose **Share…**, and copy the invite link.
3. Send them the link. **They** open it, sign in and accept.
4. Your Pi appears in their Tailscale app. They switch Tailscale on and open `http://raspberrypi:8000`, or the `100.x.x.x` address.

They can only reach the Pi, not your other devices. To remove someone, open the same **⋯ → Share…** menu in the admin console and revoke their access.

#### Good to know with more devices

- **Alerts are shared:** they're stored on the Pi, so every device sees and can change the same alerts. Give the ntfy topic only to people you trust, because anyone with it can read your alerts.
- **Favorites are not shared:** ★ favorites are saved in each browser separately, so a new device starts with an empty list.
- **Live slots are shared:** the free Finnhub key streams up to 45 stocks across **all** devices together. If several people look at different stocks at once, some update every minute instead of live.
- **Your Pi handles a few devices easily.** It's built for a household, not hundreds of visitors.
- **Never share** your Finnhub key, your account passwords, or `finnhub_key.txt`.

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
| A family member's phone can't open it at home | Their phone must be on the **same Wi-Fi** as the Pi, not mobile data or a guest network. Use `http://<your-pi-ip>:8000`, not `https`. |
| A shared person doesn't see the Pi in Tailscale | Check they accepted the invite link while signed in to **their** account, and that Tailscale is switched on. |
| My favorites are missing on another device | That's expected. Favorites are saved per browser. Tap ☆ again on that device. |
| An alert never fires | Price alerts fire only when the price **crosses** your level, and only during market hours. Test with "moves more than 0.1% in a day". |
| `ssh: connect to host ... timed out` | Check the Pi is on and on the same Wi-Fi, and the IP is right (`hostname -I` on the Pi). Away from home, switch Tailscale on and use `ssh pi@raspberrypi`. |
| Pi running hot | Check with `vcgencmd measure_temp`. With a small fan on pins 4 (5V) and 6 (GND), a Pi 4 stays around 35–50°C. |

## How it works

- `server.py`: a small Python web server, using only the standard library plus `requests` and `websocket-client`. It downloads the stock list, crawls history in the background, streams live trades and serves a JSON API.
- `index.html`: the whole website in one file (HTML, CSS and JavaScript). It uses [Chart.js](https://www.chartjs.org/) for charts.

## License

[MIT](LICENSE). You can use, change and share this project, as long as you keep the copyright notice.
