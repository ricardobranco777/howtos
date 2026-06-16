Procedure to update the EFI loader.

FreeBSD:

See loader.efi(8)

Sometimes it could be /boot/efi/efi/boot/bootx64.efi or /boot/efi/efi/freebsd/loader.efi
depending on the output of `efibootmgr -v`

```
# cp /boot/loader.efi /boot/efi/efi/freebsd/loader.efi
```

NetBSD:

See https://wiki.netbsd.org/Installation_on_UEFI_systems/

```
# mount -t msdos /dev/dk2 /mnt
# cp /usr/mdec/bootx64.efi /mnt/efi/netbsd/bootx64.efi
```

OpenBSD:

Find where the EFI is stored in the disklabel

```
# disklabel sd0 | grep MSDOS
  i:           532480               40   MSDOS                    
# mount -t msdos /dev/sd0i /mnt
# cp /usr/mdec/BOOTX64.EFI /mnt/efi/openbsd/BOOTX64.EFI
```
