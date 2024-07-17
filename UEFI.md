Create EFI entry on device:

`efibootmgr -c -p 1 -d /dev/nvme0n1p4 -L OpenBSD -l /EFI/Boot/bootx64.efi`
