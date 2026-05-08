# Setup Guide — Canon PIXMA iP100 on Ubuntu 24.04

Step-by-step instructions for a human. No prior Linux printing knowledge needed.

---

## Before You Start

- Ubuntu 24.04 installed
- iP100 connected via USB and turned on
- Internet connection (to install one package)

---

## Step 1 — Install the Gutenprint Printer Driver

Open a terminal and run:

```bash
sudo apt install printer-driver-gutenprint
```

Enter your password when prompted. This installs the driver that makes the printer work.

---

## Step 2 — Disable the usblp Module

This is the most important step. There is a Linux module called `usblp` that grabs the printer before CUPS can use it, causing print jobs to silently do nothing. You need to disable it.

Run these two commands:

```bash
sudo modprobe -r usblp
```

```bash
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf
```

The first command disables it right now. The second makes sure it stays disabled after a reboot.

---

## Step 3 — Find Your Printer's Serial Number

Run:

```bash
lpinfo -v | grep -i ip100
```

You should see something like:

```
direct usb://Canon/iP100%20series?serial=10E6AD
```

The part after `serial=` is your serial number. **Write it down** — you need it in the next step.

---

## Step 4 — Add the Printer

Copy this command and replace `YOUR_SERIAL` with the serial number you found above:

```bash
sudo lpadmin -p iP100-2 -E \
    -v "usb://Canon/iP100%20series?serial=YOUR_SERIAL" \
    -m "gutenprint.5.3://bjc-iP110-series/expert" \
    -D "Canon iP100 series"
```

For example if your serial is `10E6AD` it would be:

```bash
sudo lpadmin -p iP100-2 -E \
    -v "usb://Canon/iP100%20series?serial=10E6AD" \
    -m "gutenprint.5.3://bjc-iP110-series/expert" \
    -D "Canon iP100 series"
```

---

## Step 5 — Apply Quality Settings

Run this whole block at once — copy and paste the entire thing:

```bash
lpoptions -p iP100-2 \
    -o Resolution=612x600dpi \
    -o StpColorPrecision=Best \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpImageType=TextGraphics \
    -o StpColorCorrection=Accurate \
    -o StpDensity=800 \
    -o ColorModel=RGB \
    -o print-color-mode=color
```

Then set it as your default printer so apps like Firefox pick it up automatically:

```bash
lpoptions -d iP100-2
```

---

## Step 6 — Test Print

```bash
lpr -P iP100-2 /usr/share/cups/data/testprint
```

The printer should make a sound within a few seconds and print a color test page. If it does, you're done.

---

## Something Went Wrong?

**Printer makes no sound and nothing prints:**

The `usblp` module is probably still loaded. Run:

```bash
lsmod | grep usblp
```

If you see any output, run Step 2 again, then try printing again.

**Error says "printer not found" or similar:**

The printer might not be detected. Make sure it is turned on and the USB cable is plugged in, then run:

```bash
lpinfo -v | grep -i canon
```

If nothing shows up, try unplugging and replugging the USB cable.

**Print comes out grey / no color:**

Run:

```bash
lpoptions -p iP100-2 -o ColorModel=RGB -o print-color-mode=color
```

**Want to change quality settings later:**

Open a browser and go to `http://localhost:631` — this is the CUPS settings page where you can change printer options without using the terminal.

---

## Printing Photos

The default settings are tuned for plain paper documents. When printing photos, override the settings per job:

```bash
lpr -P iP100-2 -o StpImageType=Photo -o StpDensity=None -o MediaType=PhotoProPlat yourphoto.pdf
```

Or change the defaults temporarily:

```bash
lpoptions -p iP100-2 -o StpImageType=Photo -o StpDensity=None
# ... print your photos ...
# then switch back:
lpoptions -p iP100-2 -o StpImageType=TextGraphics -o StpDensity=800
```
