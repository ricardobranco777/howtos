Instructions for LUKS encrypted SSD connected to M.2 Hat on Raspberry Pi 5 and ZFS with automatic decryption using FIDO2 key on Ubuntu 26.04

https://ubuntu.com/download/raspberry-pi/thank-you?version=26.04.1&architecture=server-arm64+raspi

Burn image to an SD card:

```
sudo blkdiscard -vf /dev/mmcblk0
sudo dd if=ubuntu-26.04.1-preinstalled-server-arm64+raspi.img of=/dev/mmcblk0 status=progress
```

Boot it.  Log in as user `ubuntu` (password: `ubuntu`).  Then upgrade:

```
sudo apt upgrade ; sudo apt upgrade
sudo reboot
```

Optional: Change hostname:

```
sudo hostnamectl hostname rpi5.fritz.box
```

Optional: Enable SSH:

```
sudo systemctl enable --now ssh
curl -so .ssh/authorized_keys https://github.com/$YOUR_GITHUB_USER.keys
```

Optional: Enable Ubuntu Pro

```
sudo pro attach $UBUNTU_TOKEN
# Dismiss warning about kernel not being covered by livepatch.
sudo pro disable livepatch
```

Optional: Remove unneeded software since we'll rsync this SD card to the NVME.

```
# Remove Snap
sudo snap remove canonical-livepatch core22
sudo snap remove snapd
sudo apt purge snapd

# Remove Rust-based sudo.  Ubuntu ships both setuid sudos by default which is security theater.
sudo apt purge sudo-rs

# Remove unattended upgrades
sudo apt purge unattended-upgrades
sudo rm -rf /var/log/unattended-upgrades/

# Remove Rust coreutils and use good old GNU coreutils
sudo apt install --allow-remove-essential coreutils-from-gnu coreutils-from-uutils-
sudo apt remove rust-coreutils
sudo apt-mark hold rust-coreutils

# These ship setuid binaries I don't need
sudo apt remove fuse3 ntfs-3g
sudo apt autoremove
```

Configure Dracut:

```
cat <<EOF | sudo tee /etc/dracut.conf.d/90-luks-fido2.conf
hostonly="no"
add_dracutmodules+=" crypt systemd-cryptsetup fido2 "
EOF
```

Optional: Use ppa:arter97/zfs because Ubuntu always ships a newer kernel version that the officially supported by OpenZFS:

```
# We need the kernel headers to compile the module
sudo apt install linux-raspi
sudo add-apt-repository ppa:arter97/zfs
# Accept license
sudo debconf-set-selections <<< "zfs-dkms zfs-dkms/note-incompatible-licenses boolean true"
# XXX: Have to force the version here because 2.4.4-2 failed to build for arm64
version=2.4.4-1arter97~ubuntu26.04.1
# This will take some time...
sudo apt install zfs-dkms=$version zfsutils-linux zfs-zed zfs-dracut=$version
# Reboot so we have ZFS version from PPA
sudo zfs version
sudo reboot
sudo zfs version
```

Create partitions on NVME drive:

- p1: 1 GiB for /boot/firmware. The A/B layout keeps up to three copies of the boot assets.
- p2: 8 GiB swap.
- p3: LUKS container for the ZFS pool.

```
sudo blkdiscard -vf /dev/nvme0n1
echo -e '2048,2097152,c,*\n2099200,16777216,82\n18876416,,83' | sudo sfdisk /dev/nvme0n1
# Note: Do NOT label it "BOOT" because of https://github.com/raspberrypi/firmware/issues/1529
# Also don't label it "system-boot" because it would collide with the same partition on the SD card.
sudo mkfs.vfat -v /dev/nvme0n1p1
```

Create LUKS encrypted partition:

```
sudo cryptsetup luksFormat --type luks2 --cipher aes-xts-plain64 --key-size 256 --hash sha256 --pbkdf argon2id /dev/nvme0n1p3
# Allow TRIM
sudo cryptsetup --allow-discards open /dev/nvme0n1p3 crypt
```

Register FIDO2 token:

```
# Make sure it's detected
sudo systemd-cryptenroll --fido2-device=list
# You may register more than one and use true if you want to use PIN or touch
sudo systemd-cryptenroll --fido2-device=auto --fido2-with-client-pin=false --fido2-with-user-presence=false /dev/nvme0n1p3
```

Populate firmware partition:

```
sudo mount -v /dev/nvme0n1p1 /mnt
sudo rsync --archive --verbose --one-file-system --acls --xattrs /boot/firmware/ /mnt/
```

Edit cmdline:

```
LUKS_UUID=$(sudo blkid -s UUID -o value /dev/nvme0n1p3)
sudo sed -i "s%root=\S*%root=ZFS=rpool/ROOT/RPI5 rd.luks.uuid=$LUKS_UUID rd.luks.name=$LUKS_UUID=crypt rd.luks.options=$LUKS_UUID=fido2-device=auto,discard%" /mnt/current/cmdline.txt
sudo sed -i 's/rootfstype=ext4 //' /mnt/current/cmdline.txt
# Don't panic
sudo sed -i 's/panic=10 //' /mnt/current/cmdline.txt
sudo umount -v /mnt
```

Create ZFS datasets:

```
sudo zpool create -o ashift=12 -o autotrim=on -O acltype=posixacl -O canmount=off -O compression=lz4 -O dnodesize=auto -O normalization=formD -O relatime=on -O xattr=sa -O mountpoint=/ -R /mnt rpool /dev/mapper/crypt
sudo zfs create -o canmount=off -o mountpoint=none rpool/ROOT
sudo zfs create -o canmount=noauto -o mountpoint=/ rpool/ROOT/rpi5
sudo zfs mount rpool/ROOT/rpi5
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
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/lib/containerd
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/lib/containers
sudo zfs create -o com.sun:auto-snapshot=false rpool/var/lib/docker
sudo zfs create -o com.sun:auto-snapshot=false rpool/tmp
sudo chmod 1777 /mnt/tmp
sudo mkdir -m 755 /mnt/run
sudo mkdir -m 755 /mnt/run/lock
sudo zfs create rpool/var/mail
sudo mkdir -m755 /mnt/boot
sudo mkdir -m755 /mnt/boot/firmware
```

Populate filesystem:

```
sudo rsync --archive --verbose --one-file-system --acls --xattrs / /mnt/
```

Setup /etc/crypttab

```
cat <<EOF | sudo tee /mnt/etc/crypttab
crypt UUID=$LUKS_UUID none luks,discard,fido2-device=auto,x-initrd.attach
swap /dev/nvme0n1p2 /dev/urandom swap,cipher=aes-xts-plain64,size=256,sector-size=4096,discard
EOF
```

Fix /etc/fstab

```
BOOT_UUID=$(sudo blkid -s UUID -o value /dev/nvme0n1p1)
cat <<EOF | sudo tee /mnt/etc/fstab
UUID=$BOOT_UUID /boot/firmware vfat defaults 0 2
/dev/mapper/swap none swap sw 0 0
EOF
```

Make the system mount /boot/firmware after ZFS mount:

```
sudo mkdir -p /mnt/etc/systemd/system/boot-firmware.mount.d
cat <<EOF | sudo tee /mnt/etc/systemd/system/boot-firmware.mount.d/override.conf
[Unit]
After=zfs-import.target zfs-mount.service
Requires=zfs-mount.service
EOF
```

Umount filesystems and close tent:

```
sudo zfs unmount -a
sudo umount -v /mnt
sudo zpool export rpool
sudo cryptsetup close crypt
```

Now change the boot order in the EEPROM. Read right to left: 6=NVMe, 1=SD card, 4=USB MSD, F=restart.

```
sudo rpi-eeprom-config --edit
# Set BOOT_ORDER=0xf416
```

Reboot and cross fingers:

```
sudo reboot
```

Backup LUKS header:

```
# You may want to choose a better place
sudo cryptsetup luksHeaderBackup /dev/nvme0n1p3 --header-backup-file /root/luks-header-$(date +%F).img
```

Optional hardening:

```
sudo sed -i 's/$/  lsm=lockdown,capability,landlock,yama,apparmor,tomoyo,bpf,ipe,ima,evm/' /boot/firmware/current/cmdline.txt
```

Remove `ubuntu` user from `lxd` group to prevent LPE described in:
https://starlabs.sg/blog/2026/06-old-wine-in-a-new-bottle-a-decade-old-lxd-group-root-re-armed/

```
sudo gpasswd --delete ubuntu lxd
```

If you don't plan to use rootless containers, drop the setuid & devices bits:

```
sudo zfs set setuid=off devices=off rpool/var/tmp rpool/tmp rpool/home
```

Also run the Ansible playbook from https://github.com/ricardobranco777/ansible-linux

Optional: Configure Docker & Podman for ZFS:

- Add `"storage-driver": "zfs"` in /etc/docker/daemon.json and restart the service.
- `echo -e '[storage]\ndriver = "zfs"' > /etc/containers/storage.conf`

Optional: Configure Zswap:

```
sudo sed -i 's/$/ zswap.enabled=1/' /boot/firmware/current/cmdline.txt
echo 1 | sudo tee /sys/module/zswap/parameters/enabled
sudo grep -r . /sys/module/zswap/parameters/ /sys/kernel/debug/zswap/
```

Optional: Enable PCI Gen3.0

```
echo dtparam=pciex1_gen=3 | sudo tee -a /boot/firmware/config.txt
```

Note: See disclaimer at https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#pcie-gen-3-0

More information:
- https://www.raspberrypi.com/documentation/computers/config_txt.html
- https://openzfs.github.io/openzfs-docs/Getting%20Started/Debian/Debian%20Trixie%20Root%20on%20ZFS.html
