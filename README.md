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
- while on the ubuntu server pc. get the ip address. its required for ssh login
```
ip a | grep -i net
```

# connect to ubuntu server pc
- open terminal fist then: ssh username@ip_address
```
ssh rig1@10.0.0.237
```

# might have to fuck with internet settings if using wifi
- if the internet is working after fresh install i'd just leave it alone and skip this to save your sanity.

- troubleshooting: finding device names
```
ip a
```

- troubleshooting: if not finding device name? might be network driver issue
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
- edit grub file and save
- update grub
- restart pc

- amdgpu.ppfeaturemask=0xfffd7fff creates /sys/class/drm/card0/device/pp_od_clk_voltage that you can use for overclocking.
- amdgpu.ppfeaturemask=0xffffffff could also work but that higher setting causes artifacts on many cards (mainly RX 470/570)

```
sudo nano /etc/default/grub
```

```
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.ppfeaturemask=0xfffd7fff"
```

```
sudo update-grub && sudo update-grub2 && sudo update-initramfs -u -k all && reboot
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

- install drivers
- i was using one of thoese 8 slot motherboards with a soldered in cpu and this quite a while for the command to finish
```
amdgpu-install -y --accept-eula --usecase=workstation,rocm,opencl --vulkan=pro --opencl=rocr,legacy --no-32 
amdgpu-install -y --accept-eula --usecase=workstation,rocm,opencl --opencl=rocr,legacy
```


- trobleshooting: uninstalling
- something go wrong? you can try starting over using this
```
amdgpu-uninstall
sudo apt-get purge amdgpu-install
```


- add user to video group  (rig1 is your username)
```
sudo usermod -aG video rig1
```




# installing mining software
- find and copy the download link of the miner (usualy found on github)
- use the wget command to download shit in linux terminal
- then use the tar command to extract the shit that was downloaded
- here is an example for nbminer

```
wget https://github.com/NebuTech/NBMiner/releases/download/v39.7/NBMiner_39.7_Linux.tgz
tar -xf NBMiner*.tgz
ls # if you want to see if it was extracted properly
cd NB # then press tab to auto complete becasue you are way to lazy to type out the whole name every time
nano start_ergo.sh # to edit your info
./start_ergo.sh # to start mining
```


# overclocking

- run as root instead of sudo
```
sudo su
echo "manual" > /sys/class/drm/card1/device/power_dpm_force_performance_level
echo 1 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1_enable
echo 200 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1 # Fan speed
```

# overclock info
```
/sys/class/drm/card0/device/hwmon/hwmon0/pwm1_enable
```
- controls fans PWM state (pulse width modulation)
- 0 = off
- 1 = manual
- 2 = auto
- default setting is 2 / auto  (aka when gpu gets hot fan goes faster)
- change to 1 / manual (aka constant speed)


```
/sys/class/drm/card0/device/power_dpm_force_performance_level
```
- prevents getting this error:  "write error: Invalid argument"



echo "manual" > /sys/class/drm/card0/device/power_dpm_force_performance_level 

Then to apply the changes:

sudo echo 0 > /sys/class/drm/card0/device/pp_sclk_od
sudo echo 1 > /sys/class/drm/card0/device/pp_sclk_od

These are all the settings I use on my crypto mining card which is an AMD Radeon RX570 in case it's useful to anyone.

echo 1 > /sys/class/drm/card0/device/hwmon/hwmon0/pwm1_enable
echo manual > /sys/class/drm/card0/device/power_dpm_force_performance_level
echo 200 > /sys/class/drm/card0/device/hwmon/hwmon0/pwm1 # Fan speed
echo 4 > /sys/class/drm/card0/device/pp_power_profile_mode # Compute Mode

echo "s 3 1100 950" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "s 4 1100 950" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "s 5 1100 950" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "s 6 1100 950" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "s 7 1100 950" > /sys/class/drm/card0/device/pp_od_clk_voltage

echo "m 2 1985 950" > /sys/class/drm/card0/device/pp_od_clk_voltage

echo 0 > /sys/class/drm/card0/device/pp_sclk_od
echo 1 > /sys/class/drm/card0/device/pp_sclk_od

echo 0 > /sys/class/drm/card0/device/pp_mclk_od
echo 1 > /sys/class/drm/card0/device/pp_mclk_od

