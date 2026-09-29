
Debian & Ubuntu ship old versions of Podman, compiled with EOL Golang versions.

Use the version from Alvistack instead.  Unfortunately only available for amd64.

Debian:

```
source /etc/os-release
distro="${ID^}_${VERSION_ID}"
```

Ubuntu:

```
source /etc/os-release
distro="x${ID^}_${VERSION_ID}"
```

Add repositories:

```
echo "deb http://download.opensuse.org/repositories/home:/alvistack/$distro/ /" | sudo tee /etc/apt/sources.list.d/home:alvistack.list
curl -fsSL https://download.opensuse.org/repositories/home:alvistack/$distro/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/home_alvistack.gpg > /dev/null
```

```
cat <<EOF | sudo tee /etc/apt/preferences.d/alvistack
Package: *
Pin: origin "download.opensuse.org"
Pin-Priority: 1

Package: podman
Pin: origin "download.opensuse.org"
Pin-Priority: 500
EOF

sudo apt update
# On Ubuntu:
sudo apt modernize-sources
sudo apt install podman
```

If you don't use rootless containers:

```
sudo apt purge passt uidmap
```
