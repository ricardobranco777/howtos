Run `dd` to copy `install75.img` to SD card.

Install [firmware](https://github.com/pftf/RPi4/releases/download/v1.37/RPi4_UEFI_Firmware_v1.37.zip) on BOOT partition.

Boot and press ESC (Setup) to go to
`Device Manager` / `Rapberry Pi Configuration` / `Advanced Configuration` and:

- Set `Limit RAM to 3 GB` to `Disabled`
- Set `System Table Selection` to `ACPI + Devicetree`

To boot, plug HDMI cable to the port near the power supply.
Then, in the OpenBSD (not UEFI) `boot>` prompt type
`set tty fb0` to get console output in HDMI port.

## TIPS

- Add `noatime` to root entry in /etc/fstab
- Disable library relinking with `rcctl disable library_aslr`

Issues:

- `bwfm0: failed loadfirmware of file brcmfmac43455-sdio.raspberrypi,4-model-b.bin`

More information at:
- https://ftp.openbsd.org/pub/OpenBSD/7.5/arm64/INSTALL.arm64
- https://github.com/pftf/RPi4
