To boot, plug HDMI cable to the port near the power supply.
Then, in the OpenBSD (not UEFI) `boot>` prompt type
`set tty fb0` to get console output in HDMI port.

Configure `bwfm0` interface.

Issues:

- `bwfm0: failed loadfirmware of file brcmfmac43455-sdio.raspberrypi,4-model-b.bin`

More information at:
- https://ftp.openbsd.org/pub/OpenBSD/7.5/arm64/INSTALL.arm64
- https://github.com/pftf/RPi4
