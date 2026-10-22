# 8. Cooling fan (optional)

The tracker runs 24 hours a day, so keeping the Pi cool helps it stay fast and last longer. Many Pi cases come with a small fan with two wires (red and black). This guide shows how to connect it. It needs no tools and no soldering.

**Without a fan**, a Pi 4 often sits around 50–60°C. **With one**, it stays around **35–50°C**. A Pi 4 only slows itself down at about 80°C, so a fan is nice to have, not required.

---

## Step 1: Turn the Pi off properly

Never plug things into the pins while the Pi is on.

```bash
sudo shutdown now
```

Wait about 10 seconds until the green light stops blinking, then **unplug the power cable**.

## Step 2: Clear the pins

The fan plugs into the Pi's **GPIO pins**: the double row of 40 metal pins along one edge.

If a **grey ribbon cable** is plugged into them (often used for breadboard kits), remove it: hold the black plastic connector and pull it straight up gently.

> ⚠️ **If you use a ribbon cable later:** the wire with the **red stripe** must go on the end of the pins **farthest from the USB ports**. Plugged in backwards, it can damage things connected to it.

## Step 3: Find the right pins

Hold the Pi with the **pins along the top edge** and the **USB ports on the right**.

- **Top row:** the pins closest to the edge of the board
- **Left end:** the end farthest from the USB ports

Count from the **left end of the top row**:

```
          left end (far from the USB ports)
Top row:    [5V]  [5V]  [GND]  [ ]  [ ] ...   ← plug the fan into this row
Lower row:  [3V3] [ ]   [ ]    [ ]  [ ] ...
             1st   2nd   3rd
```

| Fan wire | Goes on | What the pin is |
|---|---|---|
| **Red** | **2nd pin** in the top row | 5V power (pin 4) |
| **Black** | **3rd pin** in the top row | Ground (pin 6) |

The two wires sit side by side. Push each one **all the way down** onto its pin.

## Step 4: Put the lid on

Mount the fan in the case lid with the **label side facing down**, toward the Pi, so it blows air onto the black heatsinks. Tuck the wires so they don't touch the fan blades.

## Step 5: Power on and check

Plug the power back in. **The fan should start spinning straight away.**

After the Pi starts, connect over SSH and check the temperature:

```bash
vcgencmd measure_temp
```

For example:

```
temp=33.6'C
```

| Temperature | Meaning |
|---|---|
| Under 60°C | 👍 Great |
| 60–80°C | OK, but a fan or better airflow helps |
| Over 80°C | Too hot. The Pi slows itself down. Check the fan. |

---

## Problems?

| Problem | Fix |
|---|---|
| The fan doesn't spin | Turn the Pi off, then check both wires are pushed fully down, in the **top row** (edge side), on the **2nd and 3rd** pins from the end **away from the USB ports**. If they were one row in, move them over. That mistake doesn't harm anything. |
| The fan is too loud | Turn the Pi off and move the **red** wire to the **1st pin of the lower row** (3.3V, pin 1). The fan runs slower and quieter. Some fans won't start on 3.3V; if yours doesn't, move it back. |
| The fan always runs at full speed | That's normal for a 2-wire fan. It has no speed control. |

Next: [9. Troubleshooting →](09-troubleshooting.md)
