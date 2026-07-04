
Multi-booting FreeBSD 15.x, NetBSD 11.x & OpenBSD 7.x on the same disk

Notes:
- This guide uses UEFI.
- This works even with full disk encryption for FreeBSD & OpenBSD.  NetBSD lacks encryption in the installer.

## Install FreeBSD

Install FreeBSD and choose a swap partition with a size large enough to accomodate the rest of the systems,
then resize this swap partition from FreeBSD:

```
swap=$(grep swap /etc/fstab | awk '{ print $1 }')
swapinfo
swapoff $swap
gpart show
gpart resize -i 3 -s 8g ${swap%p*}
swapon -a
swapinfo
```

Inside FreeBSD, create space for NetBSD:

`gpart add -t netbsd-ffs -s 300G -l NetBSD ada0`

## Install NetBSD & OpenBSD

Note: When you install NetBSD, it may reboot into FreeBSD, in which case
you may need to copy NetBSD's bootx64.efi (from the USB) into /boot/efi/efi/netbsd/ and
/boot/efi/efi/boot/ (do not worry about FreeBSD which created its own freebsd directory.

Inside NetBSD you may want to setup the same swap used by FreeBSD.
Note: The installer should detect FreeBSD's swap0, otherwise:

```
netbsd# dk=$(dmesg | grep swap | tee /dev/tty | grep -o 'dk[0-9]')
[     3.304628] dk4 at wd0: "swap0", 16777216 blocks at 534528, type: swap
netbsd# echo /dev/$dk none swap sw,priority=777 0 0 >> /etc/fstab
netbsd# swapon -a
swapon: adding /dev/dk4 as swap device at priority 777
netbsd# swapctl -l
```

Optional: Convert root partition to support extended attributes (and access control lists)
`fsck_ffs -c ea /`

Inside NetBSD you must create a partition for OpenBSD in the one of the *Unused* parts:

`gpt show wd0 | grep Unused`

`netbsd# gpt add -b 646457344 -s 612368384 -t obsd -l OpenBSD wd0`

When installing to OpenBSD choose "OpenBSD area".  OpenBSD is smart enough to add itself
to /boot/efi/openbsd/ but it also add itself to /boot/efi/efi/boot/

From FreeBSD:

```
rm -f /boot/efi/efi/boot/bootia32.efi
cp /boot/efi/efi/netbsd/bootx64.efi /boot/efi/efi/boot/bootx64.efi
```

## Create UEFI entries

Both FreeBSD & OpenBSD create their own.  To create it for NetBSD from FreeBSD:

```
efibootmgr -a -c -l /boot/efi/EFI/netbsd/bootx64.efi -L NetBSD
```

NOTES:
- Do not use `log` on NetBSD if you want to use `discard` and mount the FFSv2 partition from FreeBSD
- OpenBSD lacks SSD trim so from FreeBSD you can help it with:
`fsck_ffs -nE /dev/$(gpart show -p | awk '$4 == "openbsd-data" { print $3 }')`
