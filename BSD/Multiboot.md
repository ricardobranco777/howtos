
Installing NetBSD 10.x & FreeBSD 14.x in multiboot.

## Install NetBSD

Adapted from [NetBSD wiki](https://wiki.netbsd.org/Installation_on_UEFI_systems/):

```
# Use Korn shell
ksh

dkctl wd0 listwedges
gpt destroy wd0
gpt create wd0
# Use this size to avoid the formatting done as FAT-16
gpt add -l gptboot -t efi -s 512m wd0
gpt add -l NetBSD -t ffs -s 300g wd0
gpt add -l swap -t swap wd0

dkctl wd0 listwedges

newfs_msdos /dev/rdk5
mount -t msdos /dev/dk5 /mnt
mkdir -p /mnt/EFI/boot
cp /usr/mdec/*.efi /mnt/EFI/boot
umount /mnt

newfs -O 2 dk6
```

## Install FreeBSD

Adapted from [FreeBSD wiki](https://wiki.freebsd.org/RootOnZFS/GPTZFSBoot):

```
gpart add -a 1m -t freebsd-zfs -l FreeBSD ada0

# Optional: Enable encryption
geli init -g -s 4k gpt/FreeBSD
geli attach gpt/FreeBSD

mount -t tmpfs tmpfs /mnt

# Drop .eli extension if you don't want encrypt
zpool create -o altroot=/mnt zroot gpt/FreeBSD.eli

# Create ZFS pool with transparent compression enabled
zfs set compress=on zroot

zfs create -o mountpoint=none                                  zroot/ROOT
zfs create -o mountpoint=none                                  zroot/ROOT/default
mount -t zfs zroot/ROOT/default /mnt

zfs create -o mountpoint=/tmp -o exec=on -o setuid=off zroot/tmp
zfs create -o canmount=off -o mountpoint=/usr zroot/usr
zfs create zroot/home
zfs create -o setuid=off zroot/usr/src
zfs create zroot/usr/obj
zfs create -o mountpoint=/usr/ports -o setuid=off zroot/usr/ports
zfs create -o exec=off -o setuid=off zroot/usr/ports/distfiles
zfs create -o exec=off -o setuid=off zroot/usr/ports/packages
zfs create -o canmount=off -o mountpoint=/var zroot/var
zfs create -o exec=off -o setuid=off zroot/var/audit
zfs create -o exec=off -o setuid=off zroot/var/crash
zfs create -o exec=off -o setuid=off zroot/var/log
zfs create -o atime=on -o exec=off -o setuid=off zroot/var/mail
zfs create -o exec=on -o setuid=off zroot/var/tmp

chmod 1777 /mnt/var/tmp /mnt/tmp

zpool set bootfs=zroot/ROOT/default zroot

echo /dev/gpt/swap0 none swap sw 0 0 >> /tmp/bsdinstall_etc/fstab

# Backup GELI header
mkdir -m 0750 /mnt/var/backups/
cp -p /var/backups/* /mnt/var/backups/
```

After installation before reboot:

```
sysrc zfs_enable="YES"
cat >> /boot/loader.conf << EOF
aesni_load="YES"
geom_eli_load="YES"
kern.geom.label.disk_ident.enable="0"
kern.geom.label.gptid.enable="0"
EOF
```

Configure Grub on any bootable system:

```
cat > /etc/grub.d/40_custom <<EOF
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
menuentry "NetBSD" {
        insmod part_gpt
        insmod chain
        chainloader (hd0,gpt1)/EFI/boot/bootx64.efi
}

menuentry "FreeBSD" {
        insmod part_gpt
        insmod chain
        chainloader (hd0,gpt1)/EFI/freebsd/loader.efi
}
EOF

grub2-mkconfig -o /boot/grub2/grub.cfg
```
