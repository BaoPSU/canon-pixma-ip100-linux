# RESTORE.md — Full System Restore Guide for Claude

This file is written for Claude. If the Canon PIXMA iP100 stops working or needs to be set up on a fresh system, read this file top to bottom and follow the steps exactly. Everything here reflects the verified working state of this printer.

---

## Overview

There are two driver options. Use **Gutenprint** (`iP100-2`) — it is the reliable daily driver. The Canon official driver (`iP100-Canon`) was attempted but has a deep compatibility issue on Ubuntu 24.04 where it only produces 88 bytes of raster data per job, causing the printer to hang silently. Do not try to use it unless Canon releases an updated driver.

| Queue | Driver | Status | Use |
|-------|--------|--------|-----|
| `iP100-2` | Gutenprint (CUPS+Gutenprint v5.3.4) | **Working — use this** | Daily driver |
| `iP100-Canon` | cnijfilter 3.70 (Canon official) | Broken on Ubuntu 24.04 | Do not use |

---

## What Needs to Be in Place

1. `usblp` kernel module is blacklisted
2. CUPS printer queue `iP100-2` exists with the Gutenprint PPD
3. Default settings are applied
4. `iP100-2` is set as the system default printer

Check the current state before doing anything:

```bash
# Is usblp blacklisted?
cat /etc/modprobe.d/blacklist-usblp.conf

# Is usblp unloaded?
lsmod | grep usblp

# Is the printer queue there?
lpstat -p iP100-2

# Is it the system default?
lpstat -d

# Does the printer respond over USB?
sudo /usr/lib/cups/backend/usb 2>&1 | grep -i canon
```

---

## 1 — Blacklist usblp

The `usblp` kernel module grabs the USB device and locks CUPS out entirely. Jobs will appear to succeed but 0 bytes reach the printer and it makes no sound.

```bash
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf
```

Verify:

```bash
lsmod | grep usblp       # must print nothing
ls /dev/usb/lp* 2>/dev/null  # must print nothing
```

---

## 2 — Add the CUPS Printer Queue

The printer's USB serial number is `10E6AD`. The Gutenprint PPD for the iP100 is provided by the `printer-driver-gutenprint` package (install with `sudo apt install printer-driver-gutenprint` if missing).

Find the correct PPD:

```bash
lpinfo -m | grep -i "ip100\|ip110"
```

Add the printer:

```bash
sudo lpadmin -p iP100-2 -E \
    -v "usb://Canon/iP100%20series?serial=10E6AD" \
    -m "gutenprint.5.3://bjc-i100/expert" \
    -D "Canon iP100 series"
```

If the serial number differs (printer replaced), find the right URI:

```bash
lpinfo -v | grep -i ip100
```

---

## 3 — Apply the Correct Default Settings

These are the verified maximum-quality settings. Apply all at once:

```bash
lpoptions -p iP100-2 \
    -o Resolution=612x600dpi \
    -o StpColorPrecision=Best \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpImageType=Photo \
    -o StpColorCorrection=Accurate \
    -o ColorModel=RGB \
    -o print-color-mode=color

# Set as system default so Firefox and other apps auto-select it
lpoptions -d iP100-2
```

**Why each setting:**

| Setting | Value | Why |
|---------|-------|-----|
| `Resolution` | `612x600dpi` | Max available in Gutenprint for iP100 |
| `StpColorPrecision` | `Best` | Highest internal color processing |
| `StpDitherAlgorithm` | `HybridEvenTone` | Smoothest halftoning |
| `StpImageType` | `Photo` | Best rendering pipeline |
| `StpColorCorrection` | `Accurate` | True color matching |
| `ColorModel` | `RGB` | Full color output |
| `print-color-mode` | `color` | CUPS-level color — must be set separately or CUPS overrides to monochrome |

---

## 4 — Verify Everything Works

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
...
[Job N] Job completed.
```

A broken job (usblp not blacklisted) looks like:

```
[Job N] Unable to send data to printer.
[Job N] Backend usb returned status 1 (failed)
```

A broken job (Canon driver issue) looks like:

```
[Job N] Read 88 bytes of print data...
[Job N] Got USB transaction timeout during read.
```

---

## 5 — About the Canon Official Driver

The Canon cnijfilter 3.70 driver is installed (`cnijfilter-common` and `cnijfilter-ip100series`) and a libpng12 compatibility shim is at `/usr/local/lib/libpng12.so.0`. Despite this, the driver produces only 88 bytes of raster output per job on Ubuntu 24.04, which is just a job header with no page data. The printer hangs waiting for the rest and eventually times out. This is a known issue with the old driver on modern Ubuntu — do not attempt to use `iP100-Canon` as the daily driver.

---

## Verified Working State (as of 2026-05-05)

| Item | Value |
|------|-------|
| OS | Ubuntu 24.04 (Noble) |
| Working printer queue | `iP100-2` |
| Driver | CUPS+Gutenprint v5.3.4 |
| Device URI | `usb://Canon/iP100%20series?serial=10E6AD` |
| usblp blacklist | `/etc/modprobe.d/blacklist-usblp.conf` |
| System default printer | `iP100-2` |
| Resolution | `612x600dpi` (Gutenprint max) |
| StpColorPrecision | `Best` |
| StpDitherAlgorithm | `HybridEvenTone` |
| StpImageType | `Photo` |
| StpColorCorrection | `Accurate` |
| ColorModel | `RGB` |
| print-color-mode | `color` |
| PageSize | `Letter` (system default) |
