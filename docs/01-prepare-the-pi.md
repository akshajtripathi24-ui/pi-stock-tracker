# 1. Get your Raspberry Pi ready

> New to the project? Follow [0. Start here](00-start-here.md) for every step in order.

This guide gets a Raspberry Pi to the point where you can type commands on it from your computer. If your Pi already runs Raspberry Pi OS and you can SSH into it, skip to [2. Install the tracker](02-install.md).

**You need:**
- A Raspberry Pi. This project was built and tested on a **Raspberry Pi 4 (4GB)**. Other Pi models with Wi-Fi or Ethernet should work too.
- A microSD card (16GB or bigger) with Raspberry Pi OS on it
- The Pi's power supply
- Your home Wi-Fi name and password, or an Ethernet cable to your router
- A computer (Windows or Mac) on the same Wi-Fi

You don't need any sensors, wires or add-on boards. A fan is optional; see [8. Cooling fan](08-cooling-fan.md).

---

## Step 1: Put Raspberry Pi OS on the SD card

Skip this step if your Pi already starts up into Raspberry Pi OS.

1. On your computer, download **Raspberry Pi Imager** from <https://www.raspberrypi.com/software/> and install it.
2. Put the microSD card into your computer. Use a card reader if your computer has no slot.
3. Open Raspberry Pi Imager and choose:
   - **Device:** your Pi model, for example *Raspberry Pi 4*
   - **Operating system:** *Raspberry Pi OS (64-bit)*
   - **Storage:** your microSD card. Double-check it's the SD card, because everything on it gets erased.
4. Click **Next**. When it asks about **OS customisation**, click **Edit settings** and fill in:
   - **General** tab: a **username** and **password** (for example username `pi`). Write them down; you'll type the password every time you connect. Tick **Configure wireless LAN** and enter your Wi-Fi name and password. Set your country and time zone.
   - **Services** tab: tick **Enable SSH** and choose **Use password authentication**.
5. Click **Save**, then **Yes** to write the card. Wait until it says it's finished.
6. Put the card in the Pi and plug in the power. The first start takes a few minutes.

> This project was tested on Raspberry Pi OS **Bullseye** (Python 3.9). Newer versions such as Bookworm have a newer Python and should work the same way.

## Step 2: Turn on SSH (if it isn't already)

SSH lets you control the Pi from your computer, so you can copy and paste commands instead of typing them.

If you ticked **Enable SSH** in Imager, it's already on; skip to step 3.

Otherwise, on the Pi itself (with a keyboard and screen plugged in), open **Terminal** (the black **>_** icon at the top left) and run:

```bash
sudo systemctl enable --now ssh
```

## Step 3: Find your Pi's IP address

The IP address is your Pi's address on your home network, for example `192.168.1.143`.

- **On the Pi:** open Terminal and run:
  ```bash
  hostname -I
  ```
  Use the **first** number it prints.
- **Or on your router:** your router's app or admin page lists connected devices. Look for `raspberrypi`.

> **Tip:** your router may give the Pi a different address after a restart. If the address stops working, run `hostname -I` again. Many routers have a "reserve IP" or "DHCP reservation" setting that keeps the same address for the Pi.

## Step 4: Connect from your computer (SSH)

### On Windows

1. Press the **Windows key**, type `cmd`, and press **Enter**. A black Command Prompt window opens.
2. Type this, using your username and IP address, and press **Enter**:
   ```
   ssh pi@192.168.1.143
   ```
3. The first time, it asks `Are you sure you want to continue connecting?`. Type `yes` and press **Enter**.
4. Type your Pi's password and press **Enter**. **Nothing appears while you type the password.** That's normal; just type it and press Enter.

### On a Mac

1. Open **Terminal** (press **Cmd + Space**, type `Terminal`, press **Enter**).
2. Follow steps 2–4 from the Windows instructions.

### You're connected

You'll see a green prompt like:

```
pi@raspberrypi:~ $
```

Everything you type now runs on the Pi.

| To do this | Do this |
|---|---|
| Paste a command (Windows) | **Right-click** in the window |
| Paste a command (Mac) | **Cmd + V** |
| Stop a running program | **Ctrl + C** |
| Leave SSH | Type `exit` and press **Enter** |

> **Security tip:** if your Pi still uses the old default password `raspberry`, change it now by running `passwd`. Anyone on your Wi-Fi could log in otherwise.

## Step 5: Update the Pi (recommended)

```bash
sudo apt update && sudo apt full-upgrade -y
```

This can take a few minutes. When it finishes, you're ready for [2. Install the tracker](02-install.md).

---

## Problems?

| Problem | Fix |
|---|---|
| `ssh: connect to host ... timed out` | Check the Pi is on, connected to the **same Wi-Fi** as your computer, and that the IP address is right (`hostname -I` on the Pi). |
| `Connection refused` | SSH is off. Turn it on with `sudo systemctl enable --now ssh` on the Pi itself. |
| `'ssh' is not recognized` (Windows) | Your Windows is very old. Update Windows, or install the free PuTTY app. |
| `Permission denied` | The password or username is wrong. The username is the one you set in Imager, often `pi`. |

Next: [2. Install the tracker →](02-install.md)
