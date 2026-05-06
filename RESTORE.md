# RESTORE.md — Full System Restore Guide for Claude

This file is written for Claude. If the Canon PIXMA iP100 stops working or needs to be set up on a fresh system, read this file top to bottom and follow the steps exactly. Everything here reflects the verified working state of this printer.

---

## What Needs to Be in Place

There are four things that must all be true for this printer to work:

1. Canon's official `cnijfilter` driver packages are installed
2. A `libpng12.so.0` compatibility shim is installed at `/usr/local/lib/`
3. The `usblp` kernel module is blacklisted
4. A CUPS printer queue called `iP100-Canon` exists with the right PPD and defaults

Check the current state before doing anything:

```bash
# Are the driver packages installed?
dpkg -l cnijfilter-common cnijfilter-ip100series

# Is the shim there?
ls -la /usr/local/lib/libpng12.so.0

# Is usblp blacklisted?
cat /etc/modprobe.d/blacklist-usblp.conf

# Is the printer queue there?
lpstat -p iP100-Canon

# Does cifip100 resolve all its libraries?
ldd /usr/bin/cifip100 | grep "not found"
```

If any of those fail, follow the relevant section below.

---

## 1 — Install the Canon Driver Packages

The `.deb` files were originally downloaded as `cnijfilter-ip100series-3.70-1-deb` from Canon's support site. On this machine they are at:

```
/home/bao/Documents/cnijfilter-ip100series-3.70-1-deb/packages/
```

Install them:

```bash
sudo dpkg -i /home/bao/Documents/cnijfilter-ip100series-3.70-1-deb/packages/cnijfilter-common_3.70-1_amd64.deb
sudo dpkg --force-depends -i /home/bao/Documents/cnijfilter-ip100series-3.70-1-deb/packages/cnijfilter-ip100series_3.70-1_amd64.deb
```

`--force-depends` is required because Ubuntu 24.04 has renamed `libpng12-0`, `libtiff4`, and `libpango1.0-0`. The driver still works despite this — the shim in Step 2 handles libpng12.

---

## 2 — Build and Install the libpng12 Shim

The Canon filter `cifip100` needs `libpng12.so.0` with versioned symbols tagged `PNG12_0`. Ubuntu 24.04 only has libpng16. Build a thin wrapper:

```bash
cat > /tmp/libpng12-shim.c << 'EOF'
#include <png.h>
#include <stdio.h>

static void _s_set_gray_to_rgb(png_structp p) { png_set_gray_to_rgb(p); }
__asm__(".symver _s_set_gray_to_rgb,png_set_gray_to_rgb@@PNG12_0");

static void _s_set_sig_bytes(png_structp p, int n) { png_set_sig_bytes(p, n); }
__asm__(".symver _s_set_sig_bytes,png_set_sig_bytes@@PNG12_0");

static void _s_destroy_read_struct(png_structpp ps, png_infopp pi, png_infopp pe) {
    png_destroy_read_struct(ps, pi, pe);
}
__asm__(".symver _s_destroy_read_struct,png_destroy_read_struct@@PNG12_0");

static png_uint_32 _s_get_IHDR(png_structp ps, png_infop pi,
    png_uint_32 *w, png_uint_32 *h, int *bd, int *ct, int *im, int *cf, int *ff) {
    return png_get_IHDR(ps, pi, w, h, bd, ct, im, cf, ff);
}
__asm__(".symver _s_get_IHDR,png_get_IHDR@@PNG12_0");

static void _s_set_expand(png_structp p) { png_set_expand(p); }
__asm__(".symver _s_set_expand,png_set_expand@@PNG12_0");

static png_structp _s_create_read_struct(png_const_charp v, png_voidp ep,
    png_error_ptr ef, png_error_ptr wf) {
    return png_create_read_struct(v, ep, ef, wf);
}
__asm__(".symver _s_create_read_struct,png_create_read_struct@@PNG12_0");

static png_uint_32 _s_get_valid(png_structp ps, png_infop pi, png_uint_32 f) {
    return png_get_valid(ps, pi, f);
}
__asm__(".symver _s_get_valid,png_get_valid@@PNG12_0");

static void _s_read_info(png_structp ps, png_infop pi) { png_read_info(ps, pi); }
__asm__(".symver _s_read_info,png_read_info@@PNG12_0");

static png_infop _s_create_info_struct(png_structp p) {
    return png_create_info_struct(p);
}
__asm__(".symver _s_create_info_struct,png_create_info_struct@@PNG12_0");

static void _s_set_packing(png_structp p) { png_set_packing(p); }
__asm__(".symver _s_set_packing,png_set_packing@@PNG12_0");

static void _s_read_rows(png_structp ps, png_bytepp rp, png_bytepp dp, png_uint_32 n) {
    png_read_rows(ps, rp, dp, n);
}
__asm__(".symver _s_read_rows,png_read_rows@@PNG12_0");

static void _s_init_io(png_structp ps, png_FILE_p fp) { png_init_io(ps, fp); }
__asm__(".symver _s_init_io,png_init_io@@PNG12_0");

static void _s_set_strip_16(png_structp p) { png_set_strip_16(p); }
__asm__(".symver _s_set_strip_16,png_set_strip_16@@PNG12_0");

static void _s_read_update_info(png_structp ps, png_infop pi) {
    png_read_update_info(ps, pi);
}
__asm__(".symver _s_read_update_info,png_read_update_info@@PNG12_0");

static void _s_set_strip_alpha(png_structp p) { png_set_strip_alpha(p); }
__asm__(".symver _s_set_strip_alpha,png_set_strip_alpha@@PNG12_0");
EOF

gcc -shared -fPIC -o /tmp/libpng12.so.0 /tmp/libpng12-shim.c \
    -lpng16 -lz -Wl,-soname,libpng12.so.0

sudo cp /tmp/libpng12.so.0 /usr/local/lib/libpng12.so.0
sudo ldconfig
```

Verify:

```bash
ldd /usr/bin/cifip100 | grep "not found"
# Must print nothing
```

---

## 3 — Blacklist usblp

The `usblp` kernel module grabs the USB device and locks CUPS out entirely. Jobs will appear to succeed but 0 bytes reach the printer and it makes no sound.

```bash
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf
```

Verify it is gone:

```bash
lsmod | grep usblp
# Must print nothing
ls /dev/usb/lp* 2>/dev/null
# Must print nothing
```

---

## 4 — Add the CUPS Printer Queue

The printer's USB serial number is `10E6AD`. The PPD is installed at `/usr/share/ppd/canonip100.ppd`.

```bash
sudo lpadmin -p iP100-Canon -E \
    -v "usb://Canon/iP100%20series?serial=10E6AD" \
    -P /usr/share/ppd/canonip100.ppd \
    -D "Canon iP100 (Canon Driver - High Quality)"
```

If the serial number is different (e.g. the printer was replaced), find the right URI:

```bash
lpinfo -v | grep -i ip100
```

---

## 5 — Apply the Correct Default Settings

These are the verified working defaults. Apply all of them:

```bash
lpoptions -p iP100-Canon \
    -o CNQuality=2 \
    -o CNHalftoning=ed \
    -o ColorModel=rgb \
    -o print-color-mode=color \
    -o MediaType=plain
```

**Why each one matters:**
- `CNQuality=2` — high quality everyday printing; `CNQuality=1` is max but very slow
- `CNHalftoning=ed` — error diffusion, smoother color output
- `ColorModel=rgb` — Canon driver color mode; without this it prints in grayscale
- `print-color-mode=color` — CUPS-level color flag; must be set separately from `ColorModel` or CUPS overrides to monochrome
- `MediaType=plain` — matches plain paper; change per job for photo paper

---

## 6 — Verify Everything Works

```bash
lpr -P iP100-Canon /usr/share/cups/data/testprint
```

The printer should make a sound within a few seconds and print a color test page. If it is silent, check:

```bash
sudo strings /var/log/cups/error_log | grep "Job" | tail -20
```

The most common failure causes and fixes are in [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

---

## Verified Working State (as of 2026-05-05)

| Item | Value |
|------|-------|
| OS | Ubuntu 24.04 (Noble) |
| Printer queue | `iP100-Canon` |
| Device URI | `usb://Canon/iP100%20series?serial=10E6AD` |
| PPD | `/usr/share/ppd/canonip100.ppd` |
| Driver packages | `cnijfilter-common 3.70-1`, `cnijfilter-ip100series 3.70-1` |
| libpng12 shim | `/usr/local/lib/libpng12.so.0` (built from libpng16) |
| usblp blacklist | `/etc/modprobe.d/blacklist-usblp.conf` |
| CNQuality | `2` (everyday default) |
| CNHalftoning | `ed` |
| ColorModel | `rgb` |
| print-color-mode | `color` |
| MediaType | `plain` |
