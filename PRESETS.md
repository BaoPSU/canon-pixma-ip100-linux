# Quality Presets

Named combinations of settings for different print jobs. Each preset can be used **per-job** (with `lpr`) or **set as the persistent default** (with `lpoptions -p`).

The current persistent default is **Highest Quality** (set via Step 4 in [RESTORE.md](RESTORE.md)).

---

## 🔥 Highest Quality *(current default)*

**For:** flyers, posters, anything with dark areas where you want richer blacks and sharper grays on plain paper. Also fine for everyday docs if you don't mind extra ink use.

**Per-job:**
```bash
lpr -P iP100 \
    -o MediaType=PhotopaperMatte \
    -o StpDensity=None \
    -o StpImageType=Photo \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpColorCorrection=Accurate \
    -o Resolution=612x600dpi \
    YOUR_FILE
```

**Set as default:**
```bash
lpoptions -p iP100 \
    -o MediaType=PhotopaperMatte \
    -o StpDensity=None \
    -o StpImageType=Photo \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpColorCorrection=Accurate \
    -o Resolution=612x600dpi
```

**Trade-off:** uses more ink per page; slight bleed-through risk on thin paper.

---

## 📄 Standard Quality

**For:** everyday plain text documents, homework, drafts where ink economy matters.

**Per-job:**
```bash
lpr -P iP100 \
    -o MediaType=Plain \
    -o StpDensity=800 \
    -o StpImageType=TextGraphics \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpColorCorrection=Accurate \
    -o Resolution=612x600dpi \
    YOUR_FILE
```

**Set as default:**
```bash
lpoptions -p iP100 \
    -o MediaType=Plain \
    -o StpDensity=800 \
    -o StpImageType=TextGraphics \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpColorCorrection=Accurate \
    -o Resolution=612x600dpi
```

---

## ⚡ Draft / Ink Saver

**For:** internal proofs, quick reads, anything you'll throw away after. Faster and uses the least ink.

**Per-job:**
```bash
lpr -P iP100 \
    -o MediaType=Plain \
    -o StpDensity=600 \
    -o StpImageType=TextGraphics \
    -o StpDitherAlgorithm=Fast \
    -o Resolution=300dpi \
    YOUR_FILE
```

---

## 📷 Real Photo Paper

**For:** when you've actually loaded glossy or matte photo paper. Don't use these settings on plain paper or it'll soak.

**Per-job (matte):**
```bash
lpr -P iP100 \
    -o MediaType=PhotopaperMatte \
    -o StpDensity=None \
    -o StpImageType=Photo \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpColorCorrection=Accurate \
    -o Resolution=612x600dpi \
    YOUR_FILE
```

**Per-job (glossy):**
```bash
lpr -P iP100 \
    -o MediaType=PhotoPlusGloss2 \
    -o StpDensity=None \
    -o StpImageType=Photo \
    -o StpDitherAlgorithm=HybridEvenTone \
    -o StpColorCorrection=Accurate \
    -o Resolution=612x600dpi \
    YOUR_FILE
```

---

## Quick Comparison

| Preset | Resolution | Density | Image Type | Dither | Ink Use | Speed |
|---|---|---|---|---|---|---|
| 🔥 Highest Quality | 612×600 | unlimited | Photo | HybridEvenTone | Heavy | Slow |
| 📄 Standard Quality | 612×600 | 80% | TextGraphics | HybridEvenTone | Medium | Medium |
| ⚡ Draft / Ink Saver | 300 | 60% | TextGraphics | Fast | Light | Fast |
| 📷 Photo (real photo paper) | 612×600 | unlimited | Photo | HybridEvenTone | Heavy | Slow |

---

## Resetting to Default

If you set one of these as your persistent default and want to switch back to **Highest Quality**, run the "Set as default" block under that preset above.
