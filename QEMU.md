
Resize qcow2 file:

`qemu-img resize image.qcow2 +77G`

Mount qcow2 images:

```
sudo modprobe nbd
sudo qemu-nbd --connect=/dev/nbd0 /var/lib/libvirt/images/bsd.qcow2
sudo modprobe -v ufs
sudo fdisk -l /dev/nbd0
sudo mount -r -t ufs -o ufstype=ufs2 /dev/nbd0p1 /mnt
sudo umount /mnt
sudo modprobe -rv ufs
sudo qemu-nbd --disconnect /dev/nbd0
sudo modprobe -rv nbd
```
