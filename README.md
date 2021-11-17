# download
- https://ubuntu.com/#download
- find ubtuntu server (LTS version: long term service)
# burn linux iso from windows
- download https://rufus.ie/en/
- use rufus to burn .iso

# log in to server pc
- while still using the server
- finding ip address
- finding network name
```
ip a | grep -i net
uname -n
```
- once your main pc ssh to server
```
ssh rig1@10.0.0.237
```

# might have to fuck with internet settings if using wifi
- if internet is working after fresh install i'd just leave it alone and skip this to save your sanity.
```
sudo apt install network-manager
```

- finding device names
```
ip a
```

- if not finding device name? might be network driver issue
- lspci + google + good luck
```
lspci
```
- edit your network file
- net work file is .yaml
- extra info = https://www.tutorialspoint.com/yaml/yaml_basics.htm
```
sudo nano /etc/netplan/*
```
- i remember reading if there are more then 1 config files in dir
- that it will choose in filename order
- 001xyz.yaml will be picked over 002xyz.yaml
- example below
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
          "your_wifi_name+keep_the_quotes":
            password: "your_wifi_password+keep_the_quotes"   
```

```
netplan generate
netpaln apply
```

# updates

```
sudo apt update && sudo apt upgrade
```

# unlock gpu settings
- edit the bootloader to unlock gpu settings
```
sudo nano /etc/default/grub
```

- change GRUB_CMDLINE line
- save
- update grub
- restart pc
```
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.ppfeaturemask=0xfffd7fff"
```

```
sudo update-grub && sudo update-grub2 && sudo update-initramfs -u -k all
sudo reboot
```


# AMD video driver install
- worked on: November 17 2021 for rx570 (id imagine all cards would be the same process)
- went to https://www.amd.com/en/support/graphics/radeon-500-series/radeon-rx-500-series/radeon-rx-570
- clicked Ubuntu x86 64-Bit (to open drop down menu)
- right-clicked download button (brings up menu)
- clicked copy link 
- then wget and paste that link

```
wget https://repo.radeon.com/amdgpu-install/21.40.1/ubuntu/focal/amdgpu-install_21.40.1.40501-1_all.deb
```

- worst case download on another gui pc
- scp file transfer over

```
scp amd*.dep rig1@10.0.0.237:/home/rig1
```

```
sudo apt-get install ./amdgpu*.deb
sudo apt-get update
```

- i used this link for help https://amdgpu-install.readthedocs.io/en/latest/

- https://amdgpu-install.readthedocs.io/en/latest/install-overview.html
- said that i needed to use "Workstation" usecase becasue "All-Open" usecase doesn't support opencl on the rx570 (rx570 older than vega 10)
- https://en.wikipedia.org/wiki/List_of_AMD_graphics_processing_units?oldformat=true

- ROCr: Provides support for Vega 10 and newer hardware.
- Legacy: Provides support for hardware older than Vega 10.
- OpenCL: lets you run programs on gpu.

without dpkg command. i recived errors executing amdgpu-install
```
sudo dpkg --add-architecture i386
amdgpu-install -y --accept-eula --usecase=workstation,rocm,opencl --opencl=rocr,legacy
```













# install video drive
google amdgpu-pro-install
or if link still works - https://www.amd.com/en/support/kb/release-notes/rn-amdgpu-unified-linux-20-20

might be easier to download on another pc and scp the files over to ubuntu server pc
```
scp amdgpu* rig1@10.0.0.246:/home/rig1/
```
```
tar -Jxvf amdgpu*
cd amdgpu*
```
- pal = vega
- legacy = polaris
- rocm = overclocking/compute mode
- headless = server only system (aka pc doesn't use a gui)
```
./amdgpu-pro-install -y --opencl=pal,legacy,rocm --headless
```
```
sudo -i
apt install amdgpu-dkms libdrm-amdgpu-amdgpu1 libdrm2-amdgpu
```
```
usermod -a -G video rig1
usermod -a -G render rig1
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
