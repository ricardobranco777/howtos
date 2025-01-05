
## Kernel

Compiling the Linux kernel for Raspberry Pi 4b / 5:

```
sudo apt install bc bison flex libncurses5-dev libssl-dev make
git clone git@github.com:raspberrypi/linux.git
cd linux
# Optional
make menuconfig
```

For Raspberry Pi 5:

```
KERNEL=kernel_2712
make bcm2712_defconfig
```

For Raspberry Pi 4b:

```
$ KERNEL=kernel8
$ make bcm2711_defconfig
```

```
make -j6 Image.gz modules dtbs
```

To install:

```
sudo make -j6 modules_install
sudo cp /boot/firmware/$KERNEL.img /boot/firmware/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image.gz /boot/firmware/$KERNEL.img
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/overlays/
sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/overlays/
```

More information:
- [Official documentation](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compile-the-kernel)

## ZFS

```
sudo apt install build-essential autoconf automake libtool gawk alien fakeroot dkms libblkid-dev uuid-dev libudev-dev libssl-dev zlib1g-dev libaio-dev libattr1-dev libelf-dev linux-headers-generic python3 python3-dev python3-setuptools python3-cffi libffi-dev python3-packaging git libcurl4-openssl-dev debhelper-compat dh-python po-debconf python3-all-dev python3-sphinx parallel
git clone https://github.com/openzfs/zfs
cd zfs
git checkout master
sh autogen.sh
./configure
make -s -j$(nproc)
make native-deb
```

More information:
- [Official documentation](https://openzfs.github.io/openzfs-docs/Developer%20Resources/Building%20ZFS.html)
