
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
