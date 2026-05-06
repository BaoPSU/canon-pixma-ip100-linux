# CLAUDE.md — Canon iP100 Print Settings Guide

This file tells Claude how to help adjust print quality and settings for the Canon PIXMA iP100 set up with the `cnijfilter` driver on Ubuntu 24.04.

## Printer Queue Name

The printer is configured as `iP100-Canon` in CUPS.

## How to Change Settings

All settings are changed with `lpoptions`. Changes take effect immediately for all future jobs.

```bash
lpoptions -p iP100-Canon -o SETTING=VALUE
```

To check current defaults:

```bash
lpoptions -p iP100-Canon | tr ' ' '\n'
```

## Key Settings and Their Values

### Print Quality — `CNQuality`
- `1` = Maximum quality (9600×2400 dpi, very slow) — use for photo paper only
- `2` = High quality (recommended default) — good for everyday use
- `3` = Standard
- `4` = Low
- `5` = Draft (fastest)

**Default should be `2`.**

### Halftoning — `CNHalftoning`
- `ed` = Error diffusion — smoother, better for photos
- `pattern` = Dither pattern — faster, fine for documents

**Default should be `ed` for photo, `pattern` for documents.**

### Color Mode — `ColorModel`
- `rgb` = Full color (default, recommended)
- `Gray` = Grayscale
- `Black` = Black ink only

**Default should be `rgb`.**

### Media Type — `MediaType`
- `plain` = Plain paper (everyday default)
- `prophoto` = Photo Paper Pro (use with CNQuality=1)
- `glossygold` = Photo Paper Plus Glossy II (use with CNQuality=1 or 2)
- `superphoto` = Photo Paper Plus Glossy
- `glossypaper` = Glossy Photo Paper
- `semigloss` = Photo Paper Plus Semi-gloss
- `matte` = Matte Photo Paper
- `highres` = High Resolution Paper
- `otherphoto` = Other Photo Paper
- `envelope` = Envelope
- `postcard` / `ijpostcard` = Hagaki / Ink Jet Hagaki
- `tshirt` = T-Shirt Transfers

**Default should be `plain` for everyday, change per job for photo paper.**

### Render Intent — `CNRenderIntent`
- `photo` = Accurate colors (default)
- `vivid` = Boosted, vibrant colors

### Brightness — `CNGamma`
- `1.4` = Lighter output
- `1.8` = Normal (default)
- `2.2` = Darker output

### Color Balance (per channel, range -50 to 50, default 0)
- `CNBalanceC` = Cyan
- `CNBalanceM` = Magenta
- `CNBalanceY` = Yellow
- `CNBalanceK` = Black

### Density / Intensity — `CNDensity`
Range: `-50` to `50`, default `0`. Higher = more ink.

### Contrast — `CNContrast`
Range: `-50` to `50`, default `0`.

### Grayscale — `CNGrayscale`
Flag option (no value needed): `-o CNGrayscale`

### Borderless — `PageSize` with `.bl` suffix
Use page sizes like `4X6.bl`, `a4.bl`, etc. Combine with `CNExtension=0-3` (bleed amount).

### Copies — `CNCopies`
Range: `1` to `999`.

## Recommended Presets

### Everyday documents
```bash
lpoptions -p iP100-Canon -o CNQuality=2 -o MediaType=plain -o CNHalftoning=pattern -o ColorModel=rgb
```

### High-quality photo print
```bash
lpoptions -p iP100-Canon -o CNQuality=1 -o MediaType=prophoto -o CNHalftoning=ed -o ColorModel=rgb
```

### Draft / fast
```bash
lpoptions -p iP100-Canon -o CNQuality=3 -o MediaType=plain -o CNHalftoning=pattern -o ColorModel=rgb
```

## GUI Access

- CUPS web UI: `http://localhost:631` → Printers → iP100-Canon → Set Default Options
- App print dialogs: Properties → Advanced or Device Settings

## Filter Chain

CUPS sends jobs through: `pdftops` → `pstocanonij` → `cifip100` → USB backend

`cifip100` requires `libpng12.so.0` — a compatibility shim is installed at `/usr/local/lib/libpng12.so.0`.

## Known Issues

- `usblp` kernel module must be blacklisted (`/etc/modprobe.d/blacklist-usblp.conf`) or it blocks the USB backend
- `libpng12` is not in Ubuntu 24.04 repos — the shim at `/usr/local/lib/libpng12.so.0` provides it
- `CNQuality=1` is extremely slow on this portable printer — only use for final photo prints
