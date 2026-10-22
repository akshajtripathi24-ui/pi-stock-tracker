# 5. Use it away from home (Tailscale)

At home, any device on your Wi-Fi can open the tracker. Away from home (at school, on mobile data, at a friend's house) it won't load, because your Pi is only reachable inside your home network. That's a good thing: it means strangers on the internet can't reach it either.

**Tailscale** is a free app that connects **your own devices** to each other over the internet, privately. With it you can:
- open the tracker on your phone from anywhere
- SSH into your Pi from anywhere

Nothing becomes public. Only devices signed in to your Tailscale account, and people you choose to share the Pi with, can reach it.

You **don't** need Tailscale for [price alerts](04-price-alerts.md). Those already reach your phone anywhere.

---

## What you need

- A Tailscale account. It's free. You sign in with an account you already have: **GitHub, Google, Microsoft or Apple**. Ask a parent if you need help or permission.
- The Tailscale app:
  - **iPhone:** [Tailscale on the App Store](https://apps.apple.com/app/tailscale/id1470499037)
  - **Android:** [Tailscale on Google Play](https://play.google.com/store/apps/details?id=com.tailscale.ipn)
  - **Windows, Mac and more:** [Tailscale download page](https://tailscale.com/download)

---

## Step 1: Install Tailscale on the Pi

Connect to the Pi with SSH, then run:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

This downloads and installs Tailscale. It takes a minute or two.

## Step 2: Sign the Pi in

```bash
sudo tailscale up
```

It prints a link like:

```
To authenticate, visit:

        https://login.tailscale.com/a/1a2b3c4d5e6f
```

1. Copy the link and open it in your computer's browser.
2. Sign in, for example with **GitHub**.
3. Click **Connect**.

The command on the Pi finishes by itself.

## Step 3: Find the Pi's Tailscale address

```bash
tailscale ip -4
```

It prints an address starting with `100.`, for example `100.84.12.7`. Write it down. It doesn't change.

Your Pi also gets a name, **`raspberrypi`**, which usually works instead of the number.

## Step 4: Install Tailscale on your phone

1. Install the app ([iPhone](https://apps.apple.com/app/tailscale/id1470499037) · [Android](https://play.google.com/store/apps/details?id=com.tailscale.ipn)).
2. Open it and sign in with the **same account** you used for the Pi.
3. When it asks to **add a VPN configuration**, tap **Allow**. (See "It says I'm on a VPN" below for what this means.)
4. Turn the switch **on**. You should see `raspberrypi` in the device list.

## Step 5: Test it away from home

1. On your phone, turn **Wi-Fi off**, so you're on mobile data like you would be outside your home.
2. Open Safari or Chrome and go to:
   ```
   http://raspberrypi:8000
   ```
3. If that doesn't load, use the address from step 3 instead:
   ```
   http://100.84.12.7:8000
   ```

Your stock tracker loads, with live prices. 🎉 Turn Wi-Fi back on afterwards.

## Step 6 (optional): Your computer too

Install Tailscale on your computer from the [download page](https://tailscale.com/download) and sign in with the same account. Then, from anywhere:

- open `http://raspberrypi:8000` in your browser
- SSH into the Pi: `ssh pi@raspberrypi` (or `ssh pi@100.84.12.7`)

## Step 7 (recommended): Stop the Pi from being signed out

By default, Tailscale signs devices out after about 6 months. For the Pi, which nobody signs in to by hand, turn that off:

1. On your computer, go to the [Tailscale admin console](https://login.tailscale.com/admin/machines).
2. Find `raspberrypi` in the list and click the **⋯** (three dots) on the right.
3. Choose **Disable key expiry**.

Tailscale starts by itself whenever the Pi boots. You don't need to run `tailscale up` again.

---

## "It says I'm on a VPN"

Phones and computers call any app like Tailscale a **VPN**, and show a VPN icon while it's on. But Tailscale **isn't** hiding your browsing or sending it somewhere else:
- it only carries traffic **between your own devices** (and anyone you share with)
- all your other internet traffic goes out normally, as if Tailscale weren't there

You don't have to leave it on.

### Turn it off when you don't need it

- **Phone:** open the Tailscale app and flip the switch **off**. The VPN icon disappears.
- **Windows:** click the Tailscale icon by the clock (you may need to click **^** first) and choose **Disconnect**.
- **Mac:** click the Tailscale icon in the menu bar and choose **Disconnect**.

Turn it back on only when you want the tracker away from home. **At home you don't need it**: just use `http://<your-pi-ip>:8000` and `ssh pi@<your-pi-ip>` like normal.

### Remove it completely

- **Phone:** delete the Tailscale app.
- **Windows:** **Settings → Apps → Installed apps → Tailscale → ⋯ → Uninstall**.
- **Mac:** quit Tailscale and drag it from Applications to the Bin.
- **Pi:**
  ```bash
  sudo tailscale down
  sudo apt remove -y tailscale
  ```

The tracker and your phone alerts keep working either way.

---

## Problems?

| Problem | Fix |
|---|---|
| `http://raspberrypi:8000` doesn't load away from home | Check Tailscale is switched **on** in the app. Try the `100.x.x.x` address instead of the name. |
| The Pi doesn't show up in the phone app | Check you signed in to the **same** account on both. On the Pi, run `tailscale status` to see if it's connected. |
| It worked, then stopped after months | Key expiry signed the Pi out. Run `sudo tailscale up` on the Pi, sign in again, then do step 7. |
| `curl: command not found` | Run `sudo apt install -y curl`, then step 1 again. |
| My school or work network blocks it | Some networks block VPN apps. Use mobile data, or just use it at home. |

Next: [6. Use it on more devices →](06-more-devices.md)
