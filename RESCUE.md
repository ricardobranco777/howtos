
Rescue in GRUB shell:

```
insmod luks2
cryptomount -a
set root=(crypto0)
linuxefi (crypto0)/boot/vmlinuz
initrdefi (crypto0)/boot/initrd
boot
```

SUSE:

```
sudo shim-install
sudo update-bootloader
```

Disable so-called "Secure Boot":

`sudo mokutil --disable-validation`

Fix UEFI entries:

`sudo efibootmgr ...`
