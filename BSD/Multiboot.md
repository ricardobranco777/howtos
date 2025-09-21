
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
# Use .eli to encrypt with geli(8)
sed -i"" -e "/swap/s,$swap,$swap.eli," /etc/fstab
swapon -a
swapinfo
```

Inside FreeBSD, create space for NetBSD:

`gpart add -t netbsd-ffs -s 300G -l NetBSD ada0`

## Install NetBSD & OpenBSD

When you install NetBSD in its own partition, it may reboot into FreeBSD, in which case
you may need to copy NetBSD's bootx64.efi (from the USB) into /boot/efi/efi/netbsd/ and
/boot/efi/efi/boot/ (do not worry about FreeBSD which created its own freebsd directory.

Inside NetBSD you may want to setup the same swap used by FreeBSD:

```
netbsd# dk=$(dmesg | grep swap | tee /dev/tty | grep -o 'dk[0-9]')
[     3.304628] dk4 at wd0: "swap0", 16777216 blocks at 534528, type: swap
netbsd# echo /dev/$dk none swap sw,priority=777 0 0 >> /etc/fstab
netbsd# swapon -a
swapon: adding /dev/dk4 as swap device at priority 777
```

Inside NetBSD you must create a partition for OpenBSD:

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

## Configure Grub on any bootable system (optional)

```
cat > /etc/grub.d/40_custom <<EOF
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry "FreeBSD" {
        insmod part_gpt
        insmod chain
        chainloader (hd0,gpt1)/EFI/freebsd/loader.efi
}

menuentry "NetBSD" {
        insmod part_gpt
        insmod chain
        chainloader (hd0,gpt1)/efi/netbsd/bootx64.efi
}

menuentry "OpenBSD" {
        insmod part_gpt
        insmod chain
        chainloader (hd0,gpt1)/efi/openbsd/bootx64.efi
}
EOF

grub2-mkconfig -o /boot/grub2/grub.cfg
```
