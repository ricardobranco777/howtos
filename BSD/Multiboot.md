
Multi-booting FreeBSD 14.x, NetBSD 10.x & OpenBSD 7.x on the same disk

Notes:
- This guide uses UEFI.
- This works even with full disk encryption for FreeBSD & OpenBSD.  NetBSD lacks encryption in the installer.

## Install FreeBSD

Install FreeBSD and choose a swap partition with a size large enough to accomodate the rest of the systems,
then resize this swap partition from FreeBSD:

```
swap_device=$(grep swap /etc/fstab | awk '{ print $1 }')
swapinfo
swapoff $swap_device
gpart show
gpart resize -i 3 -s 8g ${swap_device%p*}
gpart modify -i 3 -l swap0 ${swap_device%p*}
swapon $swap_device
swapinfo
```

## Install NetBSD & OpenBSD

Create GPT partitions for NetBSD & OpenBSD and let the installers use this space.

## Configure Grub on any bootable system

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

## Create UEFI entries

FreeBSD creates its own.  I did this on Linux for NetBSD & OpenBSD:

```
netbsd_partition=$(sudo fdisk -l /dev/sda | grep NetBSD | awk '{ print $1 }')
uuid=$(sudo blkid -s PARTUUID -o value $netbsd_partition)
sudo efibootmgr -v -c -p "$uuid" -l "\efi\netbsd\bootx64.efi" -L NetBSD
uuid=$(sudo blkid -s PARTUUID -o value /dev/sda1)
openbsd_partition=$(sudo fdisk -l /dev/sda | grep OpenBSD | awk '{ print $1 }')
sudo efibootmgr -v -c -p "$uuid" -l "\efi\openbsd\bootx64.efi" -L OpenBSD
```
