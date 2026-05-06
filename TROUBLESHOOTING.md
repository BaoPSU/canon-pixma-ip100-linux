# Troubleshooting

## Printer accepts jobs but nothing prints / no sound

The `usblp` kernel module is probably loaded and blocking CUPS.

```bash
lsmod | grep usblp
```

If it shows `usblp`, unload it:

```bash
sudo modprobe -r usblp
echo "blacklist usblp" | sudo tee /etc/modprobe.d/blacklist-usblp.conf
```

## "error while loading shared libraries: libpng12.so.0"

The Canon driver needs libpng 1.2 which was removed from Ubuntu 24.04. Rebuild the shim:

```bash
# See Step 3 in INSTALL.md
```

Verify after installing:

```bash
ldd /usr/bin/cifip100 | grep "not found"
# Should print nothing
```

## Print is grayscale / no color

Check your ColorModel setting:

```bash
lpoptions -p iP100-Canon | tr ' ' '\n' | grep -i color
```

Force color:

```bash
lpoptions -p iP100-Canon -o ColorModel=rgb
```

## Printing is very slow

You are likely on `CNQuality=1` (maximum). Switch to `CNQuality=2` for everyday use:

```bash
lpoptions -p iP100-Canon -o CNQuality=2
```

Also consider switching halftoning from `ed` to `pattern` for documents:

```bash
lpoptions -p iP100-Canon -o CNHalftoning=pattern
```

## Check CUPS logs

Enable debug logging for a detailed trace:

```bash
sudo sed -i 's/^LogLevel warn/LogLevel debug/' /etc/cups/cupsd.conf
sudo systemctl restart cups
# print something, then:
sudo strings /var/log/cups/error_log | grep "Job" | tail -30
# restore after debugging:
sudo sed -i 's/^LogLevel debug/LogLevel warn/' /etc/cups/cupsd.conf
sudo systemctl restart cups
```

## See all current defaults

```bash
lpoptions -p iP100-Canon -l
```

## Reset all defaults

```bash
sudo lpadmin -x iP100-Canon
# Then re-add from Step 5 in INSTALL.md
```

## CUPS web UI not loading

```bash
sudo systemctl restart cups
# Then open: http://localhost:631
```
