
## Kernel

Cross-compiling the Linux kernel for Raspberry Pi 4b / 5:

On Ubuntu 24.04 VM:

```
sudo apt install bc bison flex libssl-dev make libc6-dev libncurses5-dev crossbuild-essential-arm64
git clone git@github.com:raspberrypi/linux.git
cd linux
```

For Raspbery Pi 5:

```
KERNEL=kernel_2712
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2712_defconfig
```

For Raspberry Pi 4b:

```
cd linux
$ KERNEL=kernel8
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
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
