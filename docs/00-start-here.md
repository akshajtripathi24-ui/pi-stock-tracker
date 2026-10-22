# 0. Start here: the whole project, step by step

This is the complete project in order, from an empty SD card to a stock tracker with live prices and phone alerts. Do the steps **in order**. Each step has:

- ✅ a box to tick when you're done
- the **exact commands** to copy
- a **"You're done when…"** check, so you know it worked
- a link to the **detailed guide**, if you get stuck or want to know more

**Total time:** about 1–2 hours of doing things, plus 2–3 hours of the Pi working by itself in the background.

> **How to read commands:** grey boxes are commands. Copy the whole box, paste it into the SSH window (**right-click** on Windows), and press **Enter**. Lines starting with `#` are notes; you don't need to type them.

---

## Part A: What you need

### ☐ Step 1: Gather your things

- [ ] A **Raspberry Pi** (this project used a Pi 4 with 4GB)
- [ ] Its **power supply**
- [ ] A **microSD card**, 16GB or bigger
- [ ] A **computer** (Windows or Mac) on your home Wi-Fi
- [ ] Your **Wi-Fi name and password**
- [ ] A **phone**, for alerts (optional)
- [ ] A small **case fan** (optional)
- [ ] For the first boot only, if you don't set up Wi-Fi and SSH in step 2: a keyboard, mouse, screen and HDMI cable

No sensors, wires or breadboards needed.

---

## Part B: Set up the Raspberry Pi

**Detailed guide: [1. Get your Raspberry Pi ready](01-prepare-the-pi.md)**

### ☐ Step 2: Put Raspberry Pi OS on the SD card

Skip this if your Pi already starts up into Raspberry Pi OS.

1. On your computer, install **Raspberry Pi Imager** from <https://www.raspberrypi.com/software/>.
2. Put the SD card in your computer.
3. In Imager choose: your Pi model → **Raspberry Pi OS (64-bit)** → your SD card.
4. Click **Next** → **Edit settings**:
   - set a **username** (e.g. `pi`) and **password**, and **write them down**
   - tick **Configure wireless LAN** and enter your Wi-Fi details
   - **Services** tab → tick **Enable SSH** → **Use password authentication**
5. **Save** → **Yes**, and wait until it finishes.

**You're done when:** Imager says the write was successful.

### ☐ Step 3: Start the Pi

1. Put the SD card in the Pi and plug in the power.
2. Wait **3–5 minutes** for the first start.

**You're done when:** the Pi's green light has mostly stopped blinking.

### ☐ Step 4: Find the Pi's IP address

Either:
- **On the Pi itself** (with a screen and keyboard): open **Terminal** (the **>_** icon) and run `hostname -I`. Use the first number.
- **Or** look in your router's app for a device called `raspberrypi`.

**You're done when:** you've written down an address like `192.168.1.143`. Below, wherever you see `192.168.1.143`, use **your** address.

### ☐ Step 5: Connect from your computer (SSH)

1. **Windows:** press the **Windows key**, type `cmd`, press **Enter**. **Mac:** open **Terminal**.
2. Type (with your username and IP):
   ```
   ssh pi@192.168.1.143
   ```
3. If it asks `Are you sure you want to continue connecting?`, type `yes`.
4. Type your password. **Nothing shows while you type**; that's normal. Press **Enter**.

**You're done when:** you see a green `pi@raspberrypi:~ $`.

If it says `Connection refused`, SSH is off. On the Pi itself, run `sudo systemctl enable --now ssh`, then try again.

### ☐ Step 6: Update the Pi

```bash
sudo apt update && sudo apt full-upgrade -y
```

This can take 5–15 minutes.

**You're done when:** the green prompt comes back with no red error at the end.

> If your Pi still uses the old default password `raspberry`, change it now with `passwd`.

---

## Part C: Install the stock tracker

**Detailed guide: [2. Install the tracker](02-install.md)**

### ☐ Step 7: Download the project

```bash
sudo apt install -y git
git clone https://github.com/akshajtripathi24-ui/pi-stock-tracker.git ~/stocktracker
cd ~/stocktracker
```

**You're done when:** `ls` shows `server.py`, `index.html` and `setup-autostart.sh`.

### ☐ Step 8: Install the Python packages

```bash
python3 -m venv venv
venv/bin/pip install requests websocket-client
```

**You're done when:** it says `Successfully installed …`. A notice about upgrading pip is fine to ignore.

### ☐ Step 9: Make it run by itself

```bash
bash setup-autostart.sh
```

Type your password if asked.

**You're done when:** it prints **`Done!`** and the address to open.

From now on the tracker runs in the background and starts whenever the Pi turns on. **Don't** also run `python server.py`.

### ☐ Step 10: Open the website

On your computer or phone (on the same Wi-Fi), open a browser and go to:

```
http://192.168.1.143:8000
```

**You're done when:** you see the Market page with the S&P 500, Nasdaq 100, Dow and Russell 2000 cards and lists of gainers and losers.

**Now leave the Pi on.** For the next 2–3 hours it downloads a year of history for every stock. The grey line at the top shows `loading price history …/…`. Until it says **price history ready**, some 1W, 1M and 1Y numbers show `–`. You can carry on with the next steps meanwhile.

### ☐ Step 11: Check it's healthy

```bash
journalctl -u stocktracker -f
```

**You're done when:** you see lines like `Prices refreshed: 6999 stocks from Nasdaq`. Press **Ctrl + C** to stop watching (the tracker keeps running).

🎉 **The basic tracker is finished.** Everything below is optional, but recommended.

---

## Part D: Live prices (optional)

**Detailed guide: [3. Live prices](03-live-prices.md)**

### ☐ Step 12: Get a free Finnhub key

1. Go to <https://finnhub.io> → **Get free API key** → sign up.
2. On your **Dashboard**, copy the **API key**. Don't use the "Secret".

Keep it private, like a password.

### ☐ Step 13: Save the key on the Pi

```bash
cd ~/stocktracker
nano finnhub_key.txt
```

Paste **only** the key (right-click), then **Ctrl + O**, **Enter**, **Ctrl + X**. Then:

```bash
chmod 600 finnhub_key.txt
sudo systemctl restart stocktracker
```

**You're done when:** `journalctl -u stocktracker -n 20` shows `Live prices connected (Finnhub)`, and the website badge says **● LIVE** during market hours (9:30am–4pm Eastern, Mon–Fri) or **● Live (market quiet)** outside them.

---

## Part E: Price alerts on your phone (optional)

**Detailed guide: [4. Price alerts](04-price-alerts.md)**

### ☐ Step 14: Install ntfy

- iPhone: [ntfy on the App Store](https://apps.apple.com/app/ntfy/id1625396347)
- Android: [ntfy on Google Play](https://play.google.com/store/apps/details?id=io.heckel.ntfy)

No account needed. Allow notifications.

### ☐ Step 15: Subscribe to your topic

1. On the website, click **🔔 Alerts** and click **Copy** next to your topic (`pi-stocks-…`).
2. In ntfy, tap **+**, paste the topic **exactly**, and tap **Subscribe**.

### ☐ Step 16: Test it

Click **Send test notification** on the Alerts page.

**You're done when:** your phone shows "Test notification - your price alerts are working!"

### ☐ Step 17: Add your first alert

1. Search for a stock (e.g. `apple`) and open it.
2. In the **🔔 Price alerts** box, choose **moves more than % in a day**, type `3`, and click **Add alert**.

**You're done when:** it says **Alert added ✓**. You'll get a notification on a day Apple moves 3% or more.

> Price alerts fire only when the price **crosses** your level, during market hours.

---

## Part F: Keep the Pi cool (optional)

**Detailed guide: [8. Cooling fan](08-cooling-fan.md)**

### ☐ Step 18: Connect the fan

1. Shut down safely: `sudo shutdown now`. Wait 10 seconds, then **unplug** the power.
2. Hold the Pi with the pins along the top and the USB ports on the right. In the **top row**, counting from the **left**:
   - **red wire → 2nd pin** (5V)
   - **black wire → 3rd pin** (ground)
3. Put the lid on, fan label facing down. Plug the power back in.

**You're done when:** the fan spins, and after reconnecting with SSH, `vcgencmd measure_temp` shows under 60°C.

---

## Part G: Use it away from home (optional)

**Detailed guide: [5. Tailscale](05-tailscale.md)**

Ask a parent if you need help or permission to create an account.

### ☐ Step 19: Install Tailscale on the Pi

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Open the link it prints, sign in (e.g. with GitHub), and click **Connect**. Then:

```bash
tailscale ip -4
```

Write down the `100.x.x.x` address.

### ☐ Step 20: Install Tailscale on your phone

[iPhone](https://apps.apple.com/app/tailscale/id1470499037) · [Android](https://play.google.com/store/apps/details?id=com.tailscale.ipn). Sign in with the **same** account, tap **Allow** for the VPN configuration, and switch it on.

### ☐ Step 21: Test from outside

Turn your phone's Wi-Fi **off** and open `http://raspberrypi:8000`, or `http://100.x.x.x:8000`.

**You're done when:** the tracker loads on mobile data.

### ☐ Step 22: Keep the Pi signed in

In the [Tailscale admin console](https://login.tailscale.com/admin/machines): **⋯** next to `raspberrypi` → **Disable key expiry**.

> **"It says I'm on a VPN":** that's Tailscale. It only connects your own devices. Switch it off in the app when you don't need it. At home you don't need it at all.

---

## Part H: Family and other devices (optional)

**Detailed guide: [6. Use it on more devices](06-more-devices.md)**

### ☐ Step 23: Family at home

On their phone, on your Wi-Fi, open `http://192.168.1.143:8000`. Nothing to install. Tip: **Share → Add to Home Screen** gives it an app icon.

### ☐ Step 24: Family away from home (the safe way)

1. They install Tailscale and make **their own** account.
2. You: [Tailscale admin console](https://login.tailscale.com/admin/machines) → **⋯** next to `raspberrypi` → **Share…** → copy the invite link → send it to them.
3. They open the link and tap **Accept**, then open `http://raspberrypi:8000`.

❌ **Never** give anyone your email or Tailscale password, and never make the Pi public on the internet.

### ☐ Step 25: Alerts on their phone (optional)

They install ntfy and subscribe to the **same topic**. Only share the topic with people you trust.

---

## Part I: Looking after it

### ☐ Step 26: Learn the everyday commands

| To do this | Run on the Pi |
|---|---|
| Get the latest version | `cd ~/stocktracker && git pull && sudo systemctl restart stocktracker` |
| Check it's running | `systemctl is-active stocktracker` |
| See what it's doing | `journalctl -u stocktracker -f` (Ctrl + C to stop) |
| Restart it | `sudo systemctl restart stocktracker` |
| Check the temperature | `vcgencmd measure_temp` |
| Turn the Pi off safely | `sudo shutdown now` |

### ☐ Step 27: Explore

Read [7. Using the website](07-using-the-website.md) for a tour of every page: the screener, search by CEO, charts, favorites and more.

If anything goes wrong, check [9. Troubleshooting](09-troubleshooting.md).

---

## 🎉 Finished!

You now have:
- ✅ a tracker for every US stock, running on your own Raspberry Pi 24/7
- ✅ live prices
- ✅ price alerts on your phone
- ✅ a cool, quiet Pi
- ✅ private access from anywhere, for you and your family

Want to know how it works inside? Read [10. How it works](10-how-it-works.md).
