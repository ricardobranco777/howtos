
Remount read-write:

`zfs set readonly=off $POOL`

Optional:

`zfs mount -a`

Mount:

```
zpool import -fR /mnt zroot
zfs mount -a
...
zfs unmount -a
umount /mnt
zpool export zroot
```

On Ubuntu 23.10, the default 2.2.6 ZFS version is not officially supported on the 6.11 kernel:

```
sudo add-apt-repository ppa:arter97/zfs
sudo apt install zfs-dkms
```
