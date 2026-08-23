# Burning a Region-Free DVD on Fedora Linux

## Prerequisites

```bash
sudo dnf install dvdbackup genisoimage dvd+rw-tools wodim
```

## 1. Rip the disc

```bash
dvdbackup -M -i /dev/sr0 -o /Music/
```

This strips CSS but **not** the region code.

## 2. Remove the region code

The region mask is at byte offset 35 (0x23) of `VIDEO_TS.IFO`. Patch it to `0x00` (region-free) in both the IFO and its backup:

```bash
# Verify current value (0xfe = Region 1, 0xfd = Region 2, etc.)
od -An -t x1 -j 35 -N 1 /Music/<TITLE>/VIDEO_TS/VIDEO_TS.IFO

# Patch
printf '\x00' | dd of="/Music/<TITLE>/VIDEO_TS/VIDEO_TS.IFO" bs=1 seek=35 conv=notrunc
printf '\x00' | dd of="/Music/<TITLE>/VIDEO_TS/VIDEO_TS.BUP" bs=1 seek=35 conv=notrunc
```

## 3. Build the ISO

```bash
genisoimage -dvd-video -udf -V "LABEL" -o /Music/output.iso "/Music/<TITLE>/"
```

## 4. Check disc type

```bash
dvd+rw-mediainfo /dev/sr0
```

- `Mounted Media: 1Bh, DVD+R` → single layer (4.7 GB), use a DVD+R blank
- `Mounted Media: 2Bh, DVD+R Double Layer` → dual layer (8.5 GB), use a DVD+R DL blank

If the source disc is DVD-9 (>4.7 GB), you need a DVD+R DL blank (Verbatim MKM/003 recommended).

## 5. Disable USB autosuspend (USB drives only)

Without this, the drive resets mid-burn:

```bash
echo on > /sys/bus/usb/devices/<USB_PATH>/power/control
```

Find `<USB_PATH>` from `dmesg | grep -i 'new.*device'` after plugging in.

## 6. Burn

```bash
growisofs -dvd-compat -Z /dev/sr0=/Music/output.iso
```

`growisofs` auto-detects the layer break for DL discs. Do not pass `-use-the-force-luke=break:N` unless growisofs complains and you know the exact block count (`expr $(stat -c %s output.iso) / 2048 / 2`).

```bash
growisofs -dvd-compat -Z /dev/sr0=/Music/output.iso
```

Or use `wodim` as an alternative:

```bash
wodim -v dev=/dev/sr0 speed=4 driveropts=burnfree /Music/output.iso
```

## Notes

- Burn at low speed (4x) for best compatibility with standalone players
- DVD-R blanks have the widest player compatibility for single-layer; DVD+R DL (Verbatim) for dual-layer
- The Sony UBP-X700 (EU model) is DVD Region 2 by default and outputs via HDMI only — NTSC content plays fine once the region code is cleared
