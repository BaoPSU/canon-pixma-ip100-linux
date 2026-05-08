# Canon PIXMA iP100 on Linux (Ubuntu 24.04)

Setup guide for the Canon PIXMA iP100 using the **iP110 CUPS+Gutenprint driver** — the reliable working solution on modern Ubuntu.

---

## Recommended: Use an AI Agent

The easiest way to set this up is with an AI that can directly run commands on your machine. It can read this repo and follow it automatically without you typing anything.

- **[Claude Code](https://claude.ai/code)** (recommended) — Anthropic's CLI tool. Install it, clone this repo, and tell it: _"read RESTORE.md and set up my Canon iP100 printer."_ It will handle everything.
- Any other AI coding assistant that has terminal access works the same way.

If you prefer to do it yourself, follow [SETUP.md](SETUP.md).

---

## Quick Start

```bash
# Install Gutenprint if not already present
sudo apt install printer-driver-gutenprint

# Blacklist usblp — it grabs the USB device and blocks CUPS entirely
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf

# Add the printer queue using the iP110 Gutenprint PPD
sudo lpadmin -p iP100 -E \
    -v "usb://Canon/iP100%20series?serial=10E6AD" \
    -m "gutenprint.5.3://bjc-iP110-series/expert" \
    -D "Canon iP100 series"

# Apply max quality settings
lpoptions -p iP100 \
    -o Resolution=612x600dpi \
    -o StpColorPrecision=Best \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpImageType=TextGraphics \
    -o StpColorCorrection=Accurate \
    -o StpDensity=800 \
    -o ColorModel=RGB \
    -o print-color-mode=color

# Set as system default
lpoptions -d iP100

# Test print
lpr -P iP100 /usr/share/cups/data/testprint
```

---

## Current Default Settings

| Setting | Value | What it does |
|---------|-------|--------------|
| `Resolution` | `612x600dpi` | Max resolution available via Gutenprint |
| `StpColorPrecision` | `Best` | Highest internal color processing |
| `StpDitherAlgorithm` | `HybridEvenTone` | Smoothest halftoning |
| `StpImageType` | `TextGraphics` | Tuned for documents on plain paper |
| `StpColorCorrection` | `Accurate` | True color matching |
| `StpDensity` | `800` | 80% ink density — prevents bleed on plain paper |
| `ColorModel` | `RGB` | Full color |
| `print-color-mode` | `color` | CUPS-level color mode |
| `PageSize` | `Letter` | US standard paper |
| `StpInkType` | `CMYK` | Uses all ink channels |
| `StpInkSet` | `Both` | Uses both black and color cartridges |

---

## Changing Settings

### Paper / Media Type

```bash
lpoptions -p iP100 -o MediaType=Plain             # Plain paper (default)
lpoptions -p iP100 -o MediaType=PhotoProPlat      # Photo Paper Pro
lpoptions -p iP100 -o MediaType=PhotoPlusGloss2   # Photo Paper Plus Glossy II
lpoptions -p iP100 -o MediaType=PhotopaperMatte   # Matte Photo Paper
lpoptions -p iP100 -o MediaType=GlossyPaperStandard  # Glossy Photo Paper
```

### Page Size

```bash
lpoptions -p iP100 -o PageSize=Letter   # US standard (default)
lpoptions -p iP100 -o PageSize=A4       # International
lpoptions -p iP100 -o PageSize=4X6      # Photo 4x6"
lpoptions -p iP100 -o PageSize=5X7      # Photo 5x7"
```

### Printing Photos vs Documents

```bash
# For photos — more ink, smoother gradients
lpoptions -p iP100 -o StpImageType=Photo -o StpDensity=None

# For documents — default, less ink, sharper text
lpoptions -p iP100 -o StpImageType=TextGraphics -o StpDensity=800
```

### Color vs Grayscale

```bash
lpoptions -p iP100 -o ColorModel=RGB -o print-color-mode=color       # Color
lpoptions -p iP100 -o ColorModel=Gray -o print-color-mode=monochrome # Grayscale
```

---

## GUI Access

**CUPS Web Interface:**
```
http://localhost:631
```
Go to **Printers → iP100 → Set Default Options**.

---

## Printer Specs

| Spec | Value |
|------|-------|
| Max hardware resolution | 9600 × 2400 dpi |
| Max resolution via Gutenprint | ~600 dpi |
| Connection | USB |
| Working driver | iP110 CUPS+Gutenprint v5.3.4 |
| Printer queue name | `iP100` |

---

## Full Restore / Fresh Install

See [RESTORE.md](RESTORE.md) — written for Claude to reproduce the exact working setup from scratch.

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md).
