Setup a MacOS X VM on Linux with QEMU/KVM

```
git clone --depth 1 --recursive https://github.com/kholia/OSX-KVM.git

cd OSX-KVM
wget https://github.com/kholia/OSX-KVM/pull/270.patch
wget https://github.com/kholia/OSX-KVM/pull/274.patch
git apply -3 270.patch
git apply -3 274.patch

# Choose Sonoma
./fetch-macOS-v2.py

dir=/var/lib/libvirt/images
sudo qemu-img convert BaseSystem.dmg -O raw $dir/BaseSystem.img
sudo chown qemu:qemu $dir/BaseSystem.img
sudo qemu-img create -f qcow2 $dir/mac_hdd_ng.img 100G
sudo chown qemu:qemu $dir/mac_hdd_ng.img

sudo cp OVMF_CODE_4M.fd OVMF_VARS.fd OpenCore.qcow2 $dir/
sudo chown qemu:qemu $dir/OVMF_CODE_4M.fd $dir/OVMF_VARS.fd $dir/OpenCore.qcow2
sed "s%/home/CHANGEME/OSX-KVM%$dir%" macOS-libvirt.xml  > macOS.xml
sed -i s/virbr0/br0/ macOS.xml
virsh define macOS.xml
```

Don't forget to configure the desired memory & CPU.

Go to Disk Utility and Erase & Format the disk.
