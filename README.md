# download
https://ubuntu.com/#download
# burn
https://rufus.ie/en/
# might have to fuck with internet setting if not using ethernet
```
sudo apt install network-manager
```
```
sudo nano /etc/netplan/*
```
```
# example
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
# updates
```
apt install & sudo apt upgrade
```
# add amdgpu to boot options ( this allows overclocking or compute mode i think )
```
sudo nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.ppfeaturemask=0xfffd7fff"
```
```
sudo update-grub && sudo update-grub2 && sudo update-initramfs -u -k all
```

# install video drive
google amdgpu-pro-install
or if link still works - https://www.amd.com/en/support/kb/release-notes/rn-amdgpu-unified-linux-20-20
```
scp amdgpu* rig1@10.0.0.246:/home/rig1/
```
```
tar -Jxvf amdgpu*
cd amdgpu*
exit # exit sudo -i
```
pal = vega
legacy = polaris
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
