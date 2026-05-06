# Installation Guide

Full setup for the Canon PIXMA iP100 on Ubuntu 24.04 using Canon's official `cnijfilter` driver.

## Prerequisites

- Ubuntu 24.04 (Noble)
- Canon iP100 connected via USB
- `build-essential` installed (`sudo apt install build-essential`)
- `libpng-dev` installed (`sudo apt install libpng-dev`)

## Step 1 — Get the Canon Driver

Download `cnijfilter-ip100series-3.70-1-deb` from Canon's support site. You need these two `.deb` files from the `packages/` folder:

```
cnijfilter-common_3.70-1_amd64.deb
cnijfilter-ip100series_3.70-1_amd64.deb
```

## Step 2 — Install the Driver Packages

Ubuntu 24.04 has renamed some old library dependencies, so use `--force-depends`:

```bash
sudo dpkg -i cnijfilter-common_3.70-1_amd64.deb
sudo dpkg --force-depends -i cnijfilter-ip100series_3.70-1_amd64.deb
```

## Step 3 — Fix the libpng12 Dependency

The Canon driver needs `libpng12` which was removed from Ubuntu 24.04. Build a compatibility shim that wraps `libpng16`:

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

Verify it resolved:

```bash
ldd /usr/bin/cifip100 | grep "not found"
# should print nothing
```

## Step 4 — Fix the usblp Conflict

The `usblp` kernel module grabs the USB printer device and blocks the CUPS backend. Unload it and blacklist it permanently:

```bash
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf
```

## Step 5 — Add the Printer in CUPS

```bash
sudo lpadmin -p iP100-Canon -E \
    -v "usb://Canon/iP100%20series?serial=YOUR_SERIAL" \
    -P /usr/share/ppd/canonip100.ppd \
    -D "Canon iP100 (Canon Driver - High Quality)"
```

Replace `YOUR_SERIAL` with your printer's serial (find it via `lpinfo -v | grep iP100`).

## Step 6 — Set Default Quality Options

```bash
# Recommended everyday defaults
lpoptions -p iP100-Canon \
    -o CNQuality=2 \
    -o CNHalftoning=ed \
    -o MediaType=plain
```

## Step 7 — Verify with a Test Print

```bash
lpr -P iP100-Canon /usr/share/cups/data/testprint
```

Check CUPS logs if nothing prints:

```bash
sudo journalctl -u cups -f
```

---

## Uninstalling

```bash
sudo lpadmin -x iP100-Canon
sudo dpkg -r cnijfilter-ip100series cnijfilter-common
sudo rm /usr/local/lib/libpng12.so.0
sudo rm /etc/modprobe.d/blacklist-usblp.conf
sudo ldconfig
```
