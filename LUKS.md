
To convert a LUKS encrypted partition to luks2:

`sudo cryptsetup convert --type luks2 /dev/$device`

To convert the key derivation algorithm to Argon2id (needs to be `luks2`, not yet supported by GRUB for booting):

`sudo cryptsetup luksConvertKey /dev/$device --pbkdf argon2id`

To persist `discard` flag:

`sudo cryptsetup --allow-discards --persistent refresh $cryptsetup`
