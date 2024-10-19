Instructions for LUKS encrypted SSD connected to M.2 Hat on Raspberry Pi 5 and optional ZFS with automatic decryption using FIDO2 key on Raspberry Pi OS (Debian 12).

Make sure you have latest firmware

`sudo rpi-eeprom-update`

Optional: Enable PCI Gen3.0

`echo dtparam=pciex1_gen=3 | sudo tee -a /boot/firmware/config.txt`

Add contrib to backports:

```
echo "deb http://deb.debian.org/debian bookworm-backports main contrib" > /etc/apt/sources.list.d/backports.list
```

Install systemd from backports

```
cat << EOF | sudo tee /etc/apt/preferences.d/01_systemd
Package: src:systemd
Pin: release n=bookworm-backports
Pin-Priority: 990
EOF

sudo apt update ; sudo apt upgrade
sudo reboot
```

Install FIDO2 packages

`sudo apt install fido2-tools`

Install initramfs hook for FIDO2

```
cat <<- 'EOF' | sudo tee /etc/initramfs-tools/hooks/libfido2_hook
#!/bin/sh
#
PREREQ=""

prereqs()
{
echo "$PREREQ"
}

case "$1" in
prereqs)
prereqs
exit 0
;;
esac

. /usr/share/initramfs-tools/hook-functions
. /lib/cryptsetup/functions

# Copy libfido2 and its dependencies
copy_exec /usr/bin/fido2-token
copy_exec /usr/lib/systemd/systemd-cryptsetup

copy_exec /lib/aarch64-linux-gnu/cryptsetup/libcryptsetup-token-systemd-fido2.so

if [ -f /lib/udev/rules.d/60-fido-id.rules ]; then
copy_file rule "/lib/udev/rules.d/60-fido-id.rules"
fi
EOF
```

Install ZFS from backports

```
cat << EOF | sudo tee //etc/apt/preferences.d/90_zfs
Package: src:zfs-linux
Pin: release n=bookworm-backports
Pin-Priority: 990
EOF

# NOTE: We use --no-install-recommends to avoid headers for older kernels
sudo apt install --no-install-recommends zfs-dkms zfsutils-linux zfs-initramfs zfs-zed

# Rebuild initrd
echo REMAKE_INITRD=yes | sudo tee -a /etc/dkms/zfs.conf

# Include zfs modules in initrd
echo zfs | sudo tee -a /etc/initramfs-tools/modules
```

Rebuild initrd

`sudo update-initramfs -u -k $(uname -r)`

Cleanup disk & partions

```
sudo blkdiscard -vf /dev/nvme0n1
sudo sfdisk --delete /dev/nvme0n1
```

Create partitions

```
echo -e '2048,1048576,c,*\n1050624,,83' | sudo sfdisk /dev/nvme0n1
sudo mkfs.vfat -v /dev/nvme0n1p1
```

Create LUKS encrypted partition

NOTE: On Raspberry Pi 4b use xchacha12,aes-adiantum-plain64 instead.

```
sudo cryptsetup luksFormat --type luks2 --cipher aes-xts-plain64 --key-size 256 --hash sha256 --pbkdf argon2id /dev/nvme0n1p2
sudo cryptsetup --allow-discards open /dev/nvme0n1p2 crypt
```

Register FIDO2 token

```
# Make sure it's detected
sudo systemd-cryptenroll --fido2-device=list
# NOTE: You may register more than one and use true if you want to use PIN or touch
sudo systemd-cryptenroll --fido2-device=auto --fido2-with-client-pin=false --fido2-with-user-presence=false /dev/nvme0n1p2
```

Populate firmware partition

```
sudo mount -v /dev/nvme0n1p1 /mnt
sudo rsync --archive --verbose --one-file-system --acls --xattrs /boot/firmware/ /mnt/
```

- For ZFS:
`sudo sed -i 's%root=.*rootwait%root=ZFS=rpool/ROOT/debian cryptopts=target=crypt,source=/dev/nvme0n1p2,discard,key=/dev/null,fido2-device=auto rootwait%' /mnt/cmdline.txt`
- For other filesystems:
`sudo sed -i 's%root=.*rootwait%root=/dev/mapper/crypt cryptopts=target=crypt,source=/dev/nvme0n1p2,discard,key=/dev/null,fido2-device=auto rootwait%' /mnt/cmdline.txt`

`sudo umount -v /mnt`

Create ZFS stuff

```
sudo zpool create -o ashift=12 -o autotrim=on -O acltype=posixacl -O canmount=off -O compression=lz4 -O dnodesize=auto -O normalization=formD -O relatime=on -O xattr=sa -O mountpoint=/ -R /mnt rpool /dev/mapper/crypt
sudo zfs create -o canmount=off -o mountpoint=none rpool/ROOT
sudo zfs create -o canmount=noauto -o mountpoint=/ rpool/ROOT/debian
sudo zfs mount rpool/ROOT/debian
sudo zfs create rpool/home
sudo zfs create -o mountpoint=/root rpool/home/root
sudo chmod 700 /mnt/root
sudo zfs create -o canmount=off rpool/var
sudo zfs create -o canmount=off rpool/var/lib
sudo zfs create rpool/var/log
sudo zfs create rpool/var/spool
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/cache
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/lib/nfs
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/tmp
sudo chmod 1777 /mnt/var/tmp
sudo zfs create rpool/srv
sudo zfs create -o canmount=off rpool/usr
sudo zfs create rpool/usr/local
sudo zfs create rpool/var/games
sudo zfs create rpool/var/lib/AccountsService
sudo zfs create rpool/var/lib/NetworkManager
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/lib/docker
sudo zfs create -o com.sun:auto-snapshot=false rpool/tmp
sudo chmod 1777 /mnt/tmp
sudo mkdir -m 755 /mnt/run
sudo mkdir -m 1777 /mnt/run/lock
sudo zfs create rpool/var/mail
sudo zfs create rpool/var/lib/apt
sudo zfs create rpool/var/lib/dpkg
sudo zfs create rpool/boot
sudo mkdir -m755 /mnt/boot/firmware
```

Populate filesystem

`sudo rsync --archive --verbose --one-file-system --acls --xattrs / /mnt/`

Setup /etc/crypttab (optional for ZFS):

`echo crypt /dev/nvme0n1p2 /dev/null discard,fido2-device=auto | sudo tee /mnt/etc/crypttab`

Rebuild initrd:

`sudo update-initramfs -u -k $(uname -r)`

Fix /etc/fstab keeping /proc and /boot/firmware

```
cat <<EOF | sudo tee /mnt/etc/fstab
proc /proc proc defaults 0 0
/dev/nvme0n1p1 /boot/firmware vfat defaults 0 2
EOF
```

Not needed with ZFS:

`echo /dev/mapper/crypt / ext4 discard 0 1 | sudo tee -a /mnt/etc/fstab`

Umount filesystems

```
sudo zfs unmount -a
sudo umount -v /mnt
```

Close tent

`sudo cryptsetup close crypt`

Now run sudo raspi-config and change boot order in Advanced Options / Boot Order

if you have issues in initramfs shell, toggle the first partition to `83`
and reboot. SD card will boot. Fix stuff and toggle it back to `0c`

```
fdisk /dev/nvme0n1
echo b > /proc/sysrq-trigger
```

NOTES:
- Remember to backup your LUKS header.
- Swapfile don't work on ZFS: https://github.com/openzfs/zfs/issues/7734

```
sudo systemctl disable --now dphys-swapfile.service
sudo apt install systemd-zram-generator
```
