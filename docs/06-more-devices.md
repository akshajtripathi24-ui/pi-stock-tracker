# 6. Use it on more devices (family and friends)

You can open the tracker on as many phones, tablets and computers as you like, all at the same time, and they all get live prices.

---

## How it works: there's no app to install

The tracker is a **website** that your Pi serves. The Pi does all the work, and every device just opens the website in a normal browser:

```
Finnhub ──live trades──▶  Your Pi  ──website──▶  your laptop
(your key stays            (server.py)  ──website──▶  your phone
 on the Pi)                             ──website──▶  family phones…
```

- The Pi is the **only** thing that talks to Finnhub, Nasdaq and Yahoo, using the key in `finnhub_key.txt`.
- Every open page asks the Pi for new prices **every second**, so every device sees prices tick and flash, with the **● LIVE** badge.
- The other devices **never** need your Finnhub key, your password, or any account to see live prices.

## What each device needs

| What you want on the device | What it needs |
|---|---|
| The website with live prices, **at home** | Nothing. Just a browser on the same Wi-Fi. |
| The website, **away from home** | The Tailscale app (see the options below) |
| Alert notifications | The ntfy app, subscribed to your topic ([iPhone](https://apps.apple.com/app/ntfy/id1625396347) · [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy)). See [4. Price alerts](04-price-alerts.md). |

---

## At home: just open it

1. Make sure the device is on the **same Wi-Fi** as the Pi (not mobile data, and not a "guest" network).
2. Open Safari, Chrome or any browser.
3. Go to `http://192.168.1.143:8000`, using your Pi's IP address (`hostname -I` on the Pi).

### Put it on the home screen like an app

- **iPhone (Safari):** tap the **Share** button (square with an arrow) → **Add to Home Screen** → **Add**.
- **Android (Chrome):** tap **⋮** → **Add to Home screen** → **Add**.

The tracker now has its own icon.

---

## Away from home: pick a safe option

Away from home, a device needs **Tailscale** to reach your Pi. First set up Tailscale on the Pi: [5. Use it away from home](05-tailscale.md). Then choose how to add each device:

| Option | How | Safe? | Use it for |
|---|---|---|---|
| **A. Your own account** | Install Tailscale on the device and sign in **yourself** with your account | ✅ Safe on **your own** devices | Your own second phone, tablet or laptop |
| **B. Share just the Pi** | They make their **own** free Tailscale account, and you send them a share link for the Pi only | ✅ **Safest for other people** | Family and friends |
| **C. Give someone your login** | Tell them the email and password you use for Tailscale | ❌ **Unsafe** | **Never.** It hands over your whole email account (Google, GitHub…), not just the Pi |
| **D. Make the Pi public** | Open ports on your router, or use a public tunnel | ❌ **Unsafe** | **Never.** Anyone on the internet could reach a computer in your home, and the free data is for personal use only |

### Option A: add your own device

1. Install Tailscale on the device: [iPhone](https://apps.apple.com/app/tailscale/id1470499037) · [Android](https://play.google.com/store/apps/details?id=com.tailscale.ipn) · [Windows, Mac and more](https://tailscale.com/download)
2. Sign in with the **same** account the Pi uses.
3. Tap **Allow** when it asks to add a VPN configuration, and switch Tailscale **on**.
4. Open `http://raspberrypi:8000`, or `http://100.x.x.x:8000` with the address from `tailscale ip -4` on the Pi.

⚠️ A device on your account can reach **all** your devices on Tailscale, so only do this on devices **you** own.

### Option B: share just the Pi with family and friends

This gives someone access to your Pi only. They use their **own** account and never need your email or password.

**What the other person does first:**
1. Installs Tailscale: [iPhone](https://apps.apple.com/app/tailscale/id1470499037) · [Android](https://play.google.com/store/apps/details?id=com.tailscale.ipn)
2. Opens it and creates their **own** free account, signing in with their own Google, Apple, Microsoft or GitHub account. Kids may need a parent's help.

**What you do (on your computer):**

1. Go to the [Tailscale admin console](https://login.tailscale.com/admin/machines) and sign in with **your** account.
2. You'll see the **Machines** list. Find the row for **`raspberrypi`**.
3. Click the **⋯** (three dots) at the far right of that row.
4. Choose **Share…**.
5. In the window that opens, either:
   - click **Generate & copy invite link** to make a link you can send yourself, **or**
   - type the person's email address to have Tailscale email them the invite.
6. Send the link to that person, by text or message. **Each invite is for one person**, so make a new one for each family member, and don't post it anywhere public.

> Tailscale sometimes renames buttons on its website. If you can't find these exact words, look for **Share** in the **⋯** menu next to your Pi.

**What the other person does next:**
1. Opens the invite link on their phone and signs in to **their own** Tailscale account.
2. Taps **Accept**.
3. **`raspberrypi`** now appears in their Tailscale app.
4. Switches Tailscale **on** and opens `http://raspberrypi:8000`, or `http://100.x.x.x:8000` using your Pi's Tailscale address.

**What a shared person can and can't do:**

| | |
|---|---|
| ✅ | Open your stock tracker, with live prices and alerts |
| ✅ | Add or delete alerts (alerts are shared, see below) |
| ❌ | See or reach any of your other devices (laptop, phone…) |
| ❌ | Share your Pi with anyone else |
| ❌ | Change your Tailscale settings |

At home on your Wi-Fi, they don't need Tailscale at all.

### Removing someone's access

1. Go to the [Tailscale admin console](https://login.tailscale.com/admin/machines).
2. On the **`raspberrypi`** row, click **⋯** → **Share…**.
3. Find the person and remove them, or cancel the invite if they haven't accepted yet.

They lose access straight away. Do this if someone loses their phone, or shouldn't have access any more.

---

## Good to know with more devices

| Thing | Shared between devices? | What that means |
|---|---|---|
| **Alerts** | ✅ Yes | Alerts are stored on the Pi. Every device sees the same alerts and can add or delete them. |
| **ntfy topic** | ✅ Yes | Every phone subscribed to your topic gets every alert. Only give it to people you trust, because anyone with it can read your alerts. |
| **★ Favorites** | ❌ No | Favorites are saved in each browser. A new device starts with an empty list. Tap ☆ on that device to add them. |
| **Light/dark mode** | ❌ No | Saved separately in each browser |
| **Live slots** | ✅ Yes | The free Finnhub key streams up to 45 stocks across **all** devices together. If several people look at different stocks at once, some update every minute instead of live. |

**More tips:**
- Your Pi easily handles a family's devices. It's built for a household, not hundreds of visitors.
- **Never share** your Finnhub key, `finnhub_key.txt`, or any account passwords.
- If someone loses a phone that had access, remove its access in the Tailscale admin console and, if needed, make a new ntfy topic (see [4. Price alerts](04-price-alerts.md)).

---

## Problems?

| Problem | Fix |
|---|---|
| A family phone can't open it at home | It must be on the **same Wi-Fi** as the Pi, not mobile data or a guest network. Use `http://`, not `https://`, and include `:8000`. |
| A shared person doesn't see `raspberrypi` in their app | Check they accepted the invite while signed in to **their** account, and that Tailscale is switched on. If needed, make a new invite. |
| `http://raspberrypi:8000` doesn't load for them | Try the `100.x.x.x` address instead. Check Tailscale is on. |
| My favorites are missing on another device | That's expected; favorites are per browser. |
| Some prices aren't live when several people use it | The 45 live slots are shared. The rest update every minute. |

Next: [7. Using the website →](07-using-the-website.md)
