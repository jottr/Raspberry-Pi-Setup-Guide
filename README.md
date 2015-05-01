# Raspberry Pi Setup Guide

A really opionionated guide how to setup a RPi with Arch Linux including WiringPi, ntp, wi-fi, ssh, ruby, zsh etc.

Take a look into the wiki for more interesing stuff like finding out your Raspberry Pi version.


## Some words regarding the hardware
I recommend you to get a speed class 10 SD Card with more then 4 GB capacity for optimal performance.

Additionally you should buy a small heatsink. [Something like that](http://www.amazon.com/s/ref=nb_sb_noss_1?url=search-alias%3Daps&field-keywords=raspberry%20pi%20heatsink&sprefix=raspberry+pi+he%2Caps&rh=i%3Aaps%2Ck%3Araspberry%20pi%20heatsink).


## 1. Prepare your SD

* For Raspberry Pi 1 follow [this](http://archlinuxarm.org/platforms/armv6/raspberry-pi) guide
* For Raspberry Pi 2 follow [this](http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2) guide 


## 2. Basic system setup
### 2.1. German keyboard layout and timezone
Of course just if you want to have a german keyboard layout. You may skip this step or use another layout.

```bash
loadkeys de
echo LANG=de_DE.UTF-8 > /etc/locale.conf
echo KEYMAP=de-latin1-nodeadkeys > /etc/vconsole.conf
rm /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime
sed -i "s/en_US.UTF-8/#en_US.UTF-8/" /etc/locale.conf
```

```bash
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
locale-gen en_US.UTF-8
```


### 2.2. Setup swapfile
```bash
fallocate -l 1024M /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo 'vm.swappiness=1' > /etc/sysctl.d/99-sysctl.conf
```

* Then add the following line to `/etc/fstab`:

```bash
/swapfile none swap defaults 0 0
```


### 2.3. Set hardware clock to UTC and set timezone
```bash
timedatectl set-local-rtc 0

nano /etc/timezone
```

* Set to "Europe/Berlin"
 

## 3. Update system and enable NTP
### 3.1. Tweak pacman
```bash
sed -i 's/#Color/Color/' /etc/pacman # Add color to pacman
```


### 3.2. System update
```bash
pacman -Sy pacman
pacman-key --init
pacman -S archlinux-keyring
pacman-key --populate archlinux
pacman -Syu --ignore filesystem
pacman -S filesystem --force
reboot
```


### 3.3. NTP
```bash
pacman -S ntp
systemctl enable ntpd.service
systemctl start ntpd.service
```


## 4. Advanced setup
### 4.1. Set a secure root passwd
```bash
passwd
```


### 4.2. Set hostname
```bash
hostnamectl set-hostname your-hostname
```


### 4.3. sudo & user
```bash
pacman -S sudo vim
visudo
```

* Search for following line and uncomment it:

```bash
%wheel ALL=(ALL) ALL
```

* Then install adduser

```bash
pacman -S adduser
```

* Add a new user. The additional groups are `rvm` and `wheel`

```bash
adduser
```

### 4.4. Additional software
* Log out and log in with our newly created user

```bash
sudo pacman -S --needed nfs-utils htop openssh autofs alsa-utils alsa-firmware alsa-lib alsa-plugins git zsh zsh-grml-config base-devel diffutils libnewt
```


* Install yaourt:

```bash
wget https://aur.archlinux.org/packages/pa/package-query/package-query.tar.gz
tar -xvzf package-query.tar.gz
cd package-query
makepkg -si
cd ..
wget https://aur.archlinux.org/packages/ya/yaourt/yaourt.tar.gz
tar -xvzf yaourt.tar.gz
cd yaourt
makepkg -si
```


### 4.5 vcgencmd and other vc tools
```bash
vim /etc/profile
```

Change the line saying `PATH=`:

```bash
# Set our default path
PATH="/usr/local/sbin:/usr/local/bin:/usr/bin:/opt/vc/sbin:/opt/vc/bin"
export PATH
```


### 4.6 WiringPi
```bash
sudo git clone git://git.drogon.net/wiringPi /opt/wiringpi
cd /opt/wiringpi
sudo ./build

gpio -v
gpio readall
```

* The last both command should give an `ok` or something similar. If not, something may be broken.




## 5. Setup SSH server
```bash
systemctl enable sshd
systemctl start sshd
```

* Try to connect from another machine, if it works, you can disconnect the screen and keyboard and work via ssh



## 6. Sound
Set the output device

```bash
amixer cset numid=3 1
```



## 7. Raspberry Pi overclocking
You may want to overclock the Pi. And you won't even lose the guarantee for your pi, if you use the "offical" overclocking presets. The simplest way to overclock the pi is `rasp-config` tool which ships with the offical allowed overclocking presets.

```
wget https://raw2.github.com/chattama/raspi-config-archlinux/archlinux/raspi-config
```

Get to the overclocking menu and choose the overclocking preset you want. I recommend the "high" preset. After changing the overclocking preset, reboot your raspberry pi.
 


## 8. WLAN
```bash
wifi-menu -o
netctl start yourWlanSSID
netctl enable yourWlanSSID
```


## 9. Ruby
```bash
\curl -sSL https://get.rvm.io | bash -s stable
useradd -G rvm benny
yaourt -S ruby
rvm reload
rvm install ruby
rvm list
rvm alias create default ruby-2.1.0 # Or something else depending on what rvm list says
gem install bundler
```


## 10. ZSH and dotfiles
If you want to use ZSH
```bash
sudo usermod -s /usr/bin/zsh
```

Additionally you may want to clone and setup your personal dotfiles.

* Logout and login back again or just reboot the pi
    

## 11. Tweaks
### 11.1 TRIM and noatime
Change in your fstab:

```
/dev/root  /  ext4  noatime,discard  0  0
```


### 11.2 Disable tmpfs for /tmp
Since our RAM is limited.

```bash
sudo systemctl stop tmp.mount
sudo systemctl disable tmp.mount
```




## Sources
* My experiences
* [Arch Linux Wiki: Raspberry Pi](https://wiki.archlinux.org/index.php/Raspberry_Pi)
* [elinux.org ArchLinux Install Guide](http://elinux.org/ArchLinux_Install_Guide)
* [Arch Linux Wiki: SSH Server](https://wiki.archlinux.de/title/SSH)
* [Michael Paquier: Raspberry PI with basic setup](http://michael.otacoo.com/manuals/raspberry-pi/)
