# download
https://ubuntu.com/#download
# burn
https://rufus.ie/en/
# install with wired internet to reduce gray hair
```
sudo -i
apt install network-manager
```
```
nano /etc/netplan/*
```
```
  network:
    ethernets:
        enp7s0:
          dhcp4: true
        optional: true
    version: 2
    wifis:
      wlp6s0:
        renderer: NetworkManager
        dhcp4: true
        access-points:
          "Security Van":
            password: "GoodMorning"   
```
```
netplan generate
netpaln apply
```
# add amdgpu to boot options
```
apt install & sudo apt upgrade
```
```
nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.ppfeaturemask=0xfffd7fff"
```
```
update-grub && sudo update-grub2 && sudo update-initramfs -u -k all
```
# install video drive
download driver from website
```
scp amdgpu* massaker@10.0.0.246:/home/massaker/
```
```
tar -Jxvf amdgpu*
cd amdgpu*
exit # exit sudo -i
```
```
./amdgpu-pro-install -y --opencl=pal,legacy,rocm --headless
```
```
sudo -i
apt install amdgpu-dkms libdrm-amdgpu-amdgpu1 libdrm2-amdgpu
```
```
usermod -a -G video massaker
usermod -a -G render massaker
```
# install rocm (skip maybe)
```
apt update
apt dist-upgrade
apt install libnuma-dev
```
```
reboot
```
```
apt install rocm-dkms
apt install rocm-dev
```
```
echo 'export PATH=$PATH:/opt/rocm/bin:/opt/rocm/rocprofiler/bin:/opt/rocm/opencl/bin' | sudo tee -a /etc/profile.d/rocm.sh
```
