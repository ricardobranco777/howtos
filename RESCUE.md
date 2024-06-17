
Rescue in GRUB shell:

```
insmod luks2
cryptomount -a
set root=(crypto0)
linuxefi (crypto0)/boot/vmlinuz
initrdefi (crypto0)/boot/initrd
boot
```

In UEFI systems:

`sudo shim-install`
