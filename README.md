# Canon PIXMA iP100 on Linux (Ubuntu 24.04)

Setup guide for the Canon PIXMA iP100 on Ubuntu 24.04 using the **Gutenprint** driver — the reliable daily driver for this printer on modern Ubuntu.

> **Note on the Canon official driver:** `cnijfilter 3.70` was tested but has a compatibility issue on Ubuntu 24.04 that causes it to send only a job header (88 bytes) with no page data, making the printer hang silently. Gutenprint is the working solution.

---

## Quick Start

```bash
# Blacklist usblp (blocks CUPS from reaching the printer)
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf

# Apply max quality settings
lpoptions -p iP100-2 \
    -o Resolution=612x600dpi \
    -o StpColorPrecision=Best \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpImageType=Photo \
    -o StpColorCorrection=Accurate \
    -o ColorModel=RGB \
    -o print-color-mode=color

# Set as system default
lpoptions -d iP100-2

# Test print
lpr -P iP100-2 /usr/share/cups/data/testprint
```

---

## Current Default Settings

These are the verified maximum-quality settings for the Gutenprint driver on the iP100:

| Setting | Value | What it does |
|---------|-------|--------------|
| `Resolution` | `612x600dpi` | Max resolution available in Gutenprint |
| `StpColorPrecision` | `Best` | Highest internal color processing |
| `StpDitherAlgorithm` | `HybridEvenTone` | Smoothest halftoning — best for photos and color |
| `StpImageType` | `Photo` | Best rendering pipeline |
| `StpColorCorrection` | `Accurate` | True color matching |
| `ColorModel` | `RGB` | Full color |
| `print-color-mode` | `color` | CUPS-level color mode |
| `PageSize` | `Letter` | US standard paper |
| `StpInkType` | `CMYK` | Uses all ink channels (Gutenprint default) |
| `StpInkSet` | `Both` | Uses both black and color cartridges (Gutenprint default) |

---

## Changing Settings

### Paper / Media Type

```bash
lpoptions -p iP100-2 -o MediaType=Plain          # Plain paper (default)
lpoptions -p iP100-2 -o MediaType=PhotoProPlat   # Photo Paper Pro
lpoptions -p iP100-2 -o MediaType=PhotoPlusGloss2  # Photo Paper Plus Glossy II
lpoptions -p iP100-2 -o MediaType=PhotopaperMatte  # Matte Photo Paper
lpoptions -p iP100-2 -o MediaType=GlossyPaperStandard  # Glossy Photo Paper
```

### Page Size

```bash
lpoptions -p iP100-2 -o PageSize=Letter   # US standard (default)
lpoptions -p iP100-2 -o PageSize=A4       # International
lpoptions -p iP100-2 -o PageSize=4X6      # Photo 4x6"
lpoptions -p iP100-2 -o PageSize=5X7      # Photo 5x7"
```

### Color vs Grayscale

```bash
# Full color (default)
lpoptions -p iP100-2 -o ColorModel=RGB -o print-color-mode=color

# Grayscale
lpoptions -p iP100-2 -o ColorModel=Gray -o print-color-mode=monochrome
```

### Image Type (rendering pipeline)

```bash
lpoptions -p iP100-2 -o StpImageType=Photo         # Best for photos (default)
lpoptions -p iP100-2 -o StpImageType=TextGraphics  # Better for documents/text
lpoptions -p iP100-2 -o StpImageType=LineArt       # Best for diagrams, charts
```

### Dithering Algorithm

```bash
lpoptions -p iP100-2 -o StpDitherAlgorithm=HybridEvenTone  # Best quality (default)
lpoptions -p iP100-2 -o StpDitherAlgorithm=Adaptive        # Good balance
lpoptions -p iP100-2 -o StpDitherAlgorithm=Fast            # Faster, lower quality
```

---

## What Actually Affects Quality (in order of impact)

1. **Paper** — the biggest factor. Switching from plain to photo paper makes more difference than any setting.
2. **MediaType** — must match your actual paper or ink placement will be wrong.
3. **StpImageType** — use `Photo` for photos, `TextGraphics` for documents.
4. **StpDitherAlgorithm** — `HybridEvenTone` is the ceiling.
5. **Resolution** — already at max (`612x600dpi`).

---

## GUI Access

**CUPS Web Interface** — works in any browser:
```
http://localhost:631
```
Go to **Printers → iP100-2 → Set Default Options**.

**From a print dialog** (Firefox, LibreOffice, etc.): click **Properties** → look for quality/media options.

---

## Printer Specs

| Spec | Value |
|------|-------|
| Max Color Resolution (hardware) | 9600 × 2400 dpi |
| Max Resolution via Gutenprint | ~600 dpi |
| Connection | USB |
| Working driver | CUPS+Gutenprint v5.3.4 |
| Printer queue name | `iP100-2` |

---

## Full Restore / Fresh Install

See [RESTORE.md](RESTORE.md) — written for Claude to reproduce the exact working setup from scratch.

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md).
