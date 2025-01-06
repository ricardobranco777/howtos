
Download [image](https://cdn.netbsd.org/pub/NetBSD/misc/jun/raspberry-pi/2024-12-28-aarch64/2024-12-28-netbsd-raspi-aarch64.img.gz)

- Configure `/etc/wpa_supplicant.conf` adding these lines:

```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=wheel
update_config=1
eapol_version=2
ap_scan=1
fast_reauth=1

network={
	ssid="FRITZ!Box 77770 Cable HK"
	scan_ssid=0
	psk="1234567890abcdef"
	priority=5
}
network={
	priority=0
	key_mgmt=NONE
}
```

- Add these lines to /etc/rc.conf

```
dhcpcd_flags="-qM bwfm0"
wpa_supplicant=YES
wpa_supplicant_flags="-B -s -i bwfm0 -D bsd -c /etc/wpa_supplicant.conf"
```

- Change Japanese keymap to us:

```
wsconsctl -k -w encoding=us`
sed -i.bak 's/encoding jp/encoding us/' /etc/wscons.conf
```

- Add ssh key:

```
mkdir -m 700 $HOME/.ssh
nc -l 8888 > $HOME/.ssh/authorized_keys
```

- Add `log` to `/` partition in /etc/fstab

- Comment first uncommented line in `/usr/pkg/etc/pkgin/repositories.conf` and then:

`pkgin update && pkgin upgrade`


