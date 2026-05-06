# Canon PIXMA iP100 on Linux (Ubuntu 24.04)

Full setup guide for the Canon PIXMA iP100 using Canon's official driver (`cnijfilter`) on Ubuntu 24.04. Gets you near the hardware's full **9600 × 2400 dpi** — far above what the generic Gutenprint driver can do (~600 dpi).

---

## Most Common Settings

These are the settings you'll actually change day-to-day. Run any `lpoptions` command in a terminal to apply it as the new default.

### Print Quality (`CNQuality`)

```bash
# Everyday use — high quality, reasonable speed (RECOMMENDED DEFAULT)
lpoptions -p iP100-Canon -o CNQuality=2

# Maximum quality — for photos on photo paper (slow)
lpoptions -p iP100-Canon -o CNQuality=1

# Draft — fast, lower quality, good for internal docs
lpoptions -p iP100-Canon -o CNQuality=3
```

| Value | Quality | Speed | Best For |
|-------|---------|-------|----------|
| 1 | Maximum (9600×2400 dpi) | Very slow | Photo paper, final prints |
| **2** | **High (recommended)** | **Fast** | **Documents, casual photos, everyday** |
| 3 | Standard | Faster | Drafts |
| 4 | Low | Fast | Internal drafts |
| 5 | Draft | Fastest | Quick proofs |

### Media / Paper Type (`MediaType`)

```bash
# Plain paper — everyday printing
lpoptions -p iP100-Canon -o MediaType=plain

# Glossy Photo Paper
lpoptions -p iP100-Canon -o MediaType=glossypaper

# Photo Paper Plus Glossy II — highest gloss
lpoptions -p iP100-Canon -o MediaType=glossygold

# Photo Paper Pro — professional photos
lpoptions -p iP100-Canon -o MediaType=prophoto

# Matte Photo Paper
lpoptions -p iP100-Canon -o MediaType=matte
```

| Value | Paper |
|-------|-------|
| `plain` | Plain Paper (default for documents) |
| `glossypaper` | Glossy Photo Paper |
| `glossygold` | Photo Paper Plus Glossy II |
| `prophoto` | Photo Paper Pro |
| `semigloss` | Photo Paper Plus Semi-gloss |
| `matte` | Matte Photo Paper |
| `highres` | High Resolution Paper |
| `superphoto` | Photo Paper Plus Glossy |
| `otherphoto` | Other Photo Paper |
| `envelope` | Envelope |
| `postcard` | Hagaki |
| `ijpostcard` | Ink Jet Hagaki |
| `tshirt` | T-Shirt Transfers |

### Set Multiple Options at Once

```bash
# Everyday documents
lpoptions -p iP100-Canon -o CNQuality=2 -o MediaType=plain -o CNHalftoning=pattern

# High-quality photo print
lpoptions -p iP100-Canon -o CNQuality=1 -o MediaType=prophoto -o CNHalftoning=ed

# Quick draft
lpoptions -p iP100-Canon -o CNQuality=3 -o MediaType=plain -o CNHalftoning=pattern
```

---

## All Settings Reference

### Halftoning (`CNHalftoning`)

Controls how colors are rendered.

```bash
lpoptions -p iP100-Canon -o CNHalftoning=ed       # Error diffusion (smoother, slower)
lpoptions -p iP100-Canon -o CNHalftoning=pattern  # Dither pattern (faster)
```

| Value | Description |
|-------|-------------|
| `ed` | Error diffusion — smoother gradients, best for photos |
| `pattern` | Dither pattern — faster, fine for text and documents |

### Color Balance

Adjust individual ink channel intensity. Range: `-50` to `50`, default `0`.

```bash
lpoptions -p iP100-Canon -o CNBalanceC=0   # Cyan
lpoptions -p iP100-Canon -o CNBalanceM=0   # Magenta
lpoptions -p iP100-Canon -o CNBalanceY=0   # Yellow
lpoptions -p iP100-Canon -o CNBalanceK=0   # Black
```

### Intensity / Density (`CNDensity`)

Overall ink density. Range: `-50` to `50`, default `0`. Higher = more ink.

```bash
lpoptions -p iP100-Canon -o CNDensity=0
```

### Contrast (`CNContrast`)

Range: `-50` to `50`, default `0`.

```bash
lpoptions -p iP100-Canon -o CNContrast=0
```

### Brightness / Gamma (`CNGamma`)

```bash
lpoptions -p iP100-Canon -o CNGamma=1.4   # Lighter
lpoptions -p iP100-Canon -o CNGamma=1.8   # Normal (default)
lpoptions -p iP100-Canon -o CNGamma=2.2   # Darker
```

### Render Intent (`CNRenderIntent`)

```bash
lpoptions -p iP100-Canon -o CNRenderIntent=photo   # Default — accurate colors
lpoptions -p iP100-Canon -o CNRenderIntent=vivid   # Boosted, vibrant colors
```

### Grayscale Printing

```bash
lpoptions -p iP100-Canon -o CNGrayscale   # Enable grayscale
```

### Borderless Printing

Use a borderless page size and set extension amount (how much image bleeds past edge):

```bash
lpoptions -p iP100-Canon -o PageSize=4X6.bl -o CNExtension=2
```

`CNExtension` range: `0` (minimum bleed) to `3` (maximum bleed), default `2`.

Borderless page sizes: `letter.bl`, `a4.bl`, `4X6.bl`, `4X8.bl`, `5X7.bl`, `8X10.bl`, `l.bl`, `2l.bl`, `postcard.bl`, `businesscard.bl`, `creditcard.bl`, `wide.bl`

### Copies

```bash
lpoptions -p iP100-Canon -o CNCopies=2
```

---

## Changing Settings via GUI

**CUPS Web Interface** — works in any browser:
```
http://localhost:631
```
Go to **Printers → iP100-Canon → Set Default Options**.

**From a print dialog** (LibreOffice, GNOME Files, etc.):
Click **Properties** → **Advanced** or **Device Settings** to find Canon-specific options.

---

## Printer Specs

| Spec | Value |
|------|-------|
| Max Color Resolution | 9600 × 2400 dpi |
| Max Black Resolution | 600 × 600 dpi |
| Paper Feed | Rear tray |
| Connection | USB |
| Driver | cnijfilter 3.70 (Canon official) |
| CUPS Filter | `pstocanonij` → `cifip100` |

---

## Setup & Installation

See [INSTALL.md](INSTALL.md) for the full step-by-step installation guide.

---

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

---

## License

Documentation is MIT licensed. Canon driver binaries belong to Canon Inc.
