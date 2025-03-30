Setup a MacOS X VM on Linux with QEMU/KVM

```
git clone --depth 1 --recursive https://github.com/kholia/OSX-KVM.git

cd OSX-KVM

# Fetch Ventura
./fetch-macOS-v2.py

dir=/var/lib/libvirt/images
sudo qemu-img convert BaseSystem.dmg -O raw $dir/BaseSystem.img
sudo chown qemu:qemu $dir/BaseSystem.img
sudo qemu-img create -f qcow2 $dir/mac_hdd_ng.img 100G
sudo chown qemu:qemu $dir/mac_hdd_ng.img

sudo cp OVMF_CODE.fd OVMF_VARS.fd OpenCore/OpenCore.qcow2 $dir/
sudo chown qemu:qemu $dir/OVMF_CODE.fd $dir/OVMF_VARS.fd $dir/OpenCore.qcow2
sed -i "s%/home/CHANGEME/OSX-KVM%$dir%" macOS-libvirt-Catalina.xml  > macOS.xml
sed -i s/virbr0/br0/ macOS.xml
virsh define macOS.xml
```

Don't forget to configure the desired memory & CPU.
