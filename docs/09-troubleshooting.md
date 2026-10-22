# 9. Troubleshooting

Find your problem below. Most fixes are one command, run on the Pi over SSH ([how to connect](01-prepare-the-pi.md#step-4-connect-from-your-computer-ssh)).

**First, three checks that fix most problems:**

```bash
systemctl is-active stocktracker          # should say: active
journalctl -u stocktracker -n 30          # shows the last 30 lines of the log
sudo systemctl restart stocktracker       # restarts the tracker
```

---

## Connecting to the Pi

| Problem | Fix |
|---|---|
| `ssh: connect to host ... timed out` | The Pi is off, on a different Wi-Fi, or the IP changed. Check `hostname -I` on the Pi. Away from home, switch Tailscale on and use `ssh pi@raspberrypi`. |
| `Connection refused` | SSH is off. On the Pi itself, run `sudo systemctl enable --now ssh`. |
| `Permission denied` | Wrong username or password. The username is often `pi`. |
| Nothing appears when I type the password | Normal. Type it anyway and press Enter. |
| I can't paste into the SSH window | On Windows, **right-click** pastes. On a Mac, **Cmd + V**. |

## Installing and running

| Problem | Fix |
|---|---|
| `Unit stocktracker.service not found` | Auto-start isn't set up. Run `cd ~/stocktracker && bash setup-autostart.sh`. |
| `bash: python: command not found` | Use the background service (above). To run it by hand: `cd ~/stocktracker && source venv/bin/activate`, then `python server.py`. The line must start with `(venv)`. |
| `fatal: destination path ... already exists` | You already have a `stocktracker` folder. See ["Installed it before by copying files by hand?"](02-install.md#installed-it-before-by-copying-files-by-hand). |
| `git pull` shows orange `hint:` messages | Harmless. Run `git config pull.ff only` once to hide them. |
| `pip install` errors on an old Pi OS | This project only needs `requests` and `websocket-client`. Run `~/stocktracker/venv/bin/pip install requests websocket-client`. |
| `GLIBC_2.33 not found` | This came from the old `yfinance` package, which this project no longer uses. Install the latest version from GitHub. |
| `Address already in use` | Another copy is already running. `sudo systemctl stop stocktracker; pkill -f server.py; sudo systemctl start stocktracker`. |

## The website

| Problem | Fix |
|---|---|
| The page doesn't load at all | Check the tracker is `active` (above), your device is on the **same Wi-Fi**, and the address is `http://<pi-ip>:8000` (http, not https, with `:8000`). |
| "Can't reach the Pi — is server.py running?" | The tracker stopped or the Pi is off. Restart it (above). |
| "Downloading all US stocks…" doesn't go away | The Pi can't reach Nasdaq. Check its internet connection and look at the log. |
| 1W, 1M, 6M, YTD or 1Y show `–` | The Pi is still downloading history, which takes 2–3 hours the first time. Watch the grey status line. |
| A stock's "About" says "not loaded yet" | The Pi hasn't downloaded that company's profile yet. Try later. |
| "company profiles unavailable" in the status line | Yahoo is blocking profile downloads. Everything else still works. A restart tries again. |
| The chart says "Chart data unavailable" | Yahoo didn't answer. Try another range or wait a minute. |
| The chart says "Chart library couldn't load" | Your device has no internet; the chart tool loads from the web. |
| My changes or updates don't show | Press **Ctrl + F5** (or pull down to refresh on a phone). |
| A stock's % looks very different from another site | Data sources sometimes disagree, especially for small stocks. Compare with Google Finance. |

## Live prices

| Problem | Fix |
|---|---|
| Badge says **● Live off** | `finnhub_key.txt` is missing or empty. See [3. Live prices](03-live-prices.md). |
| Badge says **● Connecting…** | Hover to see why. "Finnhub rejected the API key" means the key is wrong: use the **API key**, not the Secret. |
| Badge says **● Live (market quiet)** | Normal outside market hours: 9:30am–4pm Eastern, Mon–Fri. |
| `Finnhub error: Subscribing to too many symbols` | Two copies are running with the same key. `sudo systemctl stop stocktracker; pkill -f server.py; sudo systemctl start stocktracker`. |
| Some prices aren't live | Only 45 stocks stream at once, shared by every device. The rest update every minute. |

## Price alerts

| Problem | Fix |
|---|---|
| The test notification doesn't arrive | Check the ntfy topic matches the Alerts page **exactly**, and notifications for ntfy are allowed in your phone's Settings. |
| "Couldn't reach ntfy.sh" | The Pi isn't online. |
| An alert never fires | Price alerts fire only when the price **crosses** your level, during market hours. Look for "(waiting for it to cross again)". Test with "moves more than 0.1% in a day". |
| Too many notifications | Delete alerts you don't need, or use bigger numbers. |
| Someone has my topic who shouldn't | Make a new topic. See [4. Price alerts](04-price-alerts.md#problems). |

## Tailscale and other devices

| Problem | Fix |
|---|---|
| `http://raspberrypi:8000` doesn't load away from home | Switch Tailscale **on** in the app. Try the `100.x.x.x` address (`tailscale ip -4` on the Pi). |
| My phone says I'm on a VPN | That's Tailscale. It only connects your own devices. Turn it off in the app when you don't need it. See [5. Tailscale](05-tailscale.md#it-says-im-on-a-vpn). |
| It worked, then stopped after months | Tailscale signed the Pi out. Run `sudo tailscale up` and turn off key expiry. See [5. Tailscale](05-tailscale.md). |
| A family phone can't open it at home | It must be on the same Wi-Fi as the Pi, not mobile data or a guest network. |
| A shared person doesn't see the Pi | They must accept the invite while signed in to **their own** Tailscale account. Make a new invite if needed. |
| Favorites are missing on another device | Expected. Favorites are saved per browser. |

## Hardware

| Problem | Fix |
|---|---|
| The Pi feels hot | Check with `vcgencmd measure_temp`. Under 60°C is great. See [8. Cooling fan](08-cooling-fan.md). |
| The fan doesn't spin | Check it's on the **top row**, **2nd and 3rd** pins from the end away from the USB ports. |

---

Still stuck? Look at the full log, which usually says what's wrong:

```bash
journalctl -u stocktracker -n 100 --no-pager
```

Next: [10. How it works →](10-how-it-works.md)
