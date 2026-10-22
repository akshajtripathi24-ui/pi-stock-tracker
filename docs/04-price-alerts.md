# 4. Price alerts on your phone

Get a notification on your phone when a stock:
- **rises above** a price you choose
- **falls below** a price you choose
- **moves more than a set %** in one day (up or down)

Your Pi checks your alerts **every 10 seconds**, all day, even when the website is closed. Notifications go through **ntfy**, a free app with **no account and no sign-up**.

---

## Step 1: Install the ntfy app

- **iPhone:** [ntfy on the App Store](https://apps.apple.com/app/ntfy/id1625396347)
- **Android:** [ntfy on Google Play](https://play.google.com/store/apps/details?id=io.heckel.ntfy)

Open it once, and allow notifications when it asks.

## Step 2: Find your topic

1. Open the tracker website and click **🔔 Alerts** in the top bar.
2. Under **📱 Get alerts on your phone**, you'll see your **topic**, something like:
   ```
   pi-stocks-74613dd91fd2bc60
   ```
3. Click **Copy**.

The topic is like a private channel name. Your Pi made up a random one the first time it started, so nobody can guess it. **Anyone who knows your topic can read your alerts**, so only share it with family you trust.

## Step 3: Subscribe in the app

1. In the ntfy app, tap **+**.
2. Paste the topic **exactly**, letter for letter. Leave the server as the default (`ntfy.sh`).
3. Tap **Subscribe**.

## Step 4: Send a test

On the 🔔 Alerts page, click **Send test notification**.

Within a few seconds your phone should show:

> **Pi Stock Tracker**: Test notification - your price alerts are working!

The page says **"Sent! It should pop up on your phone in a few seconds."** If it says it couldn't reach ntfy.sh, the Pi isn't connected to the internet.

## Step 5: Add an alert

1. Search for a stock in the box at the top (for example `apple`) and open it.
2. Scroll to the **🔔 Price alerts** box.
3. Next to "Notify me when …", choose a type:
   - **rises above $**
   - **falls below $**
   - **moves more than % in a day**
4. Type a number. The box suggests one for you: 5% above or below the current price, or 5%.
5. Click **Add alert**. You'll see **Alert added ✓** and the alert appears in the list below.

**Good alerts to start with:**

| Alert | What it tells you |
|---|---|
| moves more than **3**% in a day | The stock is having a big day, up or down |
| rises above a price a bit **higher** than now | It's going up |
| falls below a price a bit **lower** than now | It's dropping |

---

## How alerts fire (and why one might not)

| Type | When it fires | How often |
|---|---|---|
| **rises above $X** | When the price goes from **below** X to **above** X | Once each time it crosses. To fire again, it has to drop about 0.5% below X first, so small wiggles don't spam you. |
| **falls below $X** | When the price goes from **above** X to **below** X | Once each time it crosses (same 0.5% rule) |
| **moves more than X%** | When today's change, up **or** down, is X% or more | At most **once a day** per alert |

**Important:**
- **It has to cross.** If you add "rises above $1" on a $500 stock, it's already above $1. It shows **(waiting for it to cross again)** and only fires if the price drops below $1 and comes back up. When you add an alert like that, the page tells you.
- **Prices only move during market hours:** 9:30am–4pm Eastern, Monday–Friday. Outside those hours, alerts don't fire because prices don't change.
- **Quick test:** add "moves more than **0.1**% in a day" on a big stock like MSFT during market hours. It should fire within about 10 seconds. Delete it afterwards with ✕.

---

## Managing your alerts

The **🔔 Alerts** page shows:
- **Your alerts:** every alert, with the stock's current price. Click a stock name to open it, or **✕** to delete an alert.
- **Recent notifications:** the last 30 notifications sent, with the time. "not sent" means the Pi couldn't reach ntfy at that moment.

Each stock page also lists that stock's alerts, with ✕ to delete. You can have up to 100 alerts.

Alerts are saved on the Pi in `alerts.json`, so they survive restarts and updates.

---

## Alerts on more than one phone

Install ntfy on the other phone and subscribe to the **same topic**. Every phone subscribed gets every alert.

Alerts live on the Pi, so every device that opens the website sees the same alerts and can add or delete them. See [6. Use it on more devices](06-more-devices.md).

You **don't** need Tailscale for alerts. They reach your phone anywhere: at school, on mobile data, anywhere with internet.

---

## Problems?

| Problem | Fix |
|---|---|
| The test notification doesn't arrive | Check the topic in the app matches the Alerts page **exactly**. Check notifications for ntfy are allowed in your phone's **Settings**. |
| The page says "Couldn't reach ntfy.sh" | The Pi isn't online. Check its internet connection. |
| An alert never fires | Price alerts need the price to **cross** your level, during market hours. Look for "(waiting for it to cross again)" on the Alerts page. |
| I get too many notifications | Delete alerts you don't need, or use bigger numbers (e.g. 5% instead of 1%). |
| I want a new private topic (e.g. someone you didn't want has it) | Run the commands below. Your alerts are kept, and a new random topic is created. Then subscribe to the new topic in the ntfy app on each phone, and delete the old one. |

**Making a new private topic:**

```bash
sudo systemctl stop stocktracker
cd ~/stocktracker
python3 -c "import json; d = json.load(open('alerts.json')); d['topic'] = ''; json.dump(d, open('alerts.json', 'w'))"
sudo systemctl start stocktracker
```

Then refresh the 🔔 Alerts page to see the new topic.

Next: [5. Use it away from home →](05-tailscale.md)
