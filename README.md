# info
- had a rough time installing AMD linux drives so I wrote this document for my future reference and to possibly help others.
- used for: ubuntu server crypto mining rig
- worked on: november 17 2021
- iso file i used: ubuntu-20.04.3-live-server-amd64.iso
- after install I did everything via ssh

# download
- https://ubuntu.com/#download
- find ubtuntu server (LTS version: long term support)

# burn linux iso from windows to usb
- download https://rufus.ie/en/
- use rufus to burn .iso

# installing ubuntu server
- i assume you know how to do this (its not hard)
- put usb in server pc, boot up and selet install and follow the instuctions (thats about it)
- while on the ubuntu server pc. get the ip address and network name of the ubuntu server pc. these are required for ssh login
```
ip a | grep -i net
uname -n
```

# connect to ubuntu server pc
- open terminal fist then: ssh network_name@ip_address
```
ssh rig1@10.0.0.237
```

# might have to fuck with internet settings if using wifi
- if the internet is working after fresh install i'd just leave it alone and skip this to save your sanity.
```
sudo apt install network-manager
```

- troubleshooting: finding device names
```
ip a
```

- if not finding device name? might be network driver issue
- lspci + google + good luck
```
lspci
```

- install network-manager

```
sudo apt install network-manager
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

# increase swap
- skip this if you have lots of ram

```
sudo swapoff /swap.img
sudo fallocate -l 8G /swap.img
sudo mkswap /swap.img
sudo swapon /swap.img
```

# updates
- update system
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
- if amdgpu.ppfeaturemask=0xfffd7fff
- amdgpu.ppfeaturemask=0xffffffff

```
sudo update-grub && sudo update-grub2 && sudo update-initramfs -u -k all
sudo reboot
```


# AMD video driver install
- went to https://repo.radeon.com/amdgpu-install/latest/ubuntu/focal/
- right-clicked the link (brings up menu)
- clicked copy link 
- then wget and paste that link

```
wget https://repo.radeon.com/amdgpu-install/latest/ubuntu/focal/amdgpu-install-21.40.40500-1_all.deb
```

- alternative: if for some reason wget doesn't work can try to download the file on your main pc then scp file transfer over

```
cd c:\what_ever_dir_has_the_file_in_it_probably_download_folder_thought
scp amd*.dep rig1@10.0.0.237:/home/rig1
```

```
sudo apt-get install ./amdgpu*.deb
sudo apt-get update
```

- i used this link for help https://amdgpu-install.readthedocs.io/en/latest/

- https://amdgpu-install.readthedocs.io/en/latest/install-overview.html said that I needed to use "Workstation" usecase becasue "All-Open" usecase doesn't support opencl on the rx570 (rx570 older than vega 10)
- https://en.wikipedia.org/wiki/List_of_AMD_graphics_processing_units?oldformat=true

- ROCr: Provides support for Vega 10 and newer hardware.
- Legacy: Provides support for hardware older than Vega 10.
- OpenCL: lets you run programs on gpu.



```
sudo dpkg --add-architecture i386
```

- prevents this errror
```
The following packages have unmet dependencies:
 amdgpu-pro-lib32 : Depends: amdgpu-lib32 (= 21.40.40500-1331380) but it is not going to be installed
                    Depends: libgl1-amdgpu-pro-glx:i386 (= 21.40-1331380) but it is not installable
                    Depends: libegl1-amdgpu-pro:i386 (= 21.40-1331380) but it is not installable
                    Depends: libgles2-amdgpu-pro:i386 (= 21.40-1331380) but it is not installable
                    Depends: libglapi1-amdgpu-pro:i386 (= 21.40-1331380) but it is not installable
                    Depends: libgl1-amdgpu-pro-dri:i386 (= 21.40-1331380) but it is not installable

```

- install
```
amdgpu-install -y --accept-eula --usecase=workstation,rocm,opencl --opencl=rocr,legacy
```

- add user to video group  (rig1 is your username)
```
sudo usermod -aG video rig1
```




# installing miners
- find and copy the download link of the miner (usualy found on github)
- use the wget command to download shit on linux
- then use the tar command to extract the shit that was downloaded
- example for nbminer

```
wget https://github.com/NebuTech/NBMiner/releases/download/v39.7/NBMiner_39.7_Linux.tgz
tar -xf NBMiner*.tgz
ls # if you want to see if it was extracted properly
cd NB # then hit tab to auto complete becasue you are way to lazy to type out the whole name every time
nano start_ergo.sh # to edit your info
./start_ergo.sh # to start mining
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
