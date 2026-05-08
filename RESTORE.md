# RESTORE.md — Full System Restore Guide for Claude

This file is written for Claude. If the Canon PIXMA iP100 stops working or needs to be set up on a fresh system, read this file top to bottom and follow the steps exactly. Everything here reflects the verified working state of this printer.

---

## What Needs to Be in Place

1. `usblp` kernel module is blacklisted
2. CUPS printer queue `iP100-2` exists using the iP110 Gutenprint PPD
3. Default settings are applied
4. `iP100-2` is set as the system default printer

Check the current state before doing anything:

```bash
# Is usblp blacklisted and unloaded?
cat /etc/modprobe.d/blacklist-usblp.conf
lsmod | grep usblp

# Is the printer queue there?
lpstat -p iP100-2

# Is it the system default?
lpstat -d

# Does the printer respond over USB?
sudo /usr/lib/cups/backend/usb 2>&1 | grep -i canon
```

---

## 1 — Install Gutenprint

```bash
sudo apt install printer-driver-gutenprint
```

---

## 2 — Blacklist usblp

The `usblp` kernel module grabs the USB device and locks CUPS out entirely. Jobs will appear to succeed but 0 bytes reach the printer and it makes no sound.

```bash
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf
```

Verify:

```bash
lsmod | grep usblp          # must print nothing
ls /dev/usb/lp* 2>/dev/null # must print nothing
```

---

## 3 — Add the CUPS Printer Queue

The printer's USB serial number is `10E6AD`. The driver used is the **iP110 Gutenprint PPD** — not the iP100 one. The iP100 and iP110 are close enough hardware that the iP110 driver works and is more reliable.

```bash
sudo lpadmin -p iP100-2 -E \
    -v "usb://Canon/iP100%20series?serial=10E6AD" \
    -m "gutenprint.5.3://bjc-iP110-series/expert" \
    -D "Canon iP100 series"
```

If the serial number differs (printer replaced), find the right URI first:

```bash
lpinfo -v | grep -i ip100
```

---

## 4 — Apply Default Settings

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

lpoptions -d iP100-2
```

**Why each setting:**

| Setting | Value | Why |
|---------|-------|-----|
| `Resolution` | `612x600dpi` | Max available in Gutenprint |
| `StpColorPrecision` | `Best` | Highest internal color processing |
| `StpDitherAlgorithm` | `HybridEvenTone` | Smoothest halftoning |
| `StpImageType` | `TextGraphics` | Tuned for plain paper documents |
| `StpColorCorrection` | `Accurate` | True color matching |
| `StpDensity` | `800` | 80% ink — prevents bleed-through on plain paper |
| `ColorModel` | `RGB` | Full color |
| `print-color-mode` | `color` | CUPS-level color — must be set or CUPS defaults to monochrome |
| System default | `iP100-2` | So Firefox and other apps auto-select it |

---

## 5 — Verify Everything Works

```bash
lpr -P iP100-2 /usr/share/cups/data/testprint
```

The printer should make a sound within a few seconds and print a color test page. Watch progress:

```bash
sudo strings /var/log/cups/error_log | grep "Job" | tail -20
```

A working job looks like:

```
[Job N] Printing page 1, 13%
[Job N] Printing page 1, 14%
[Job N] Job completed.
```

A broken job (usblp not blacklisted) looks like:

```
[Job N] Unable to send data to printer.
[Job N] Backend usb returned status 1 (failed)
```

---

## Verified Working State (as of 2026-05-07)

| Item | Value |
|------|-------|
| OS | Ubuntu 24.04 (Noble) |
| Printer queue | `iP100-2` |
| Driver | iP110 CUPS+Gutenprint v5.3.4 (`gutenprint.5.3://bjc-iP110-series/expert`) |
| Device URI | `usb://Canon/iP100%20series?serial=10E6AD` |
| usblp blacklist | `/etc/modprobe.d/blacklist-usblp.conf` |
| System default | `iP100-2` |
| Resolution | `612x600dpi` |
| StpColorPrecision | `Best` |
| StpDitherAlgorithm | `HybridEvenTone` |
| StpImageType | `TextGraphics` |
| StpColorCorrection | `Accurate` |
| StpDensity | `800` |
| ColorModel | `RGB` |
| print-color-mode | `color` |
| PageSize | `Letter` |
