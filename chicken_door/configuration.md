---
layout: page
title: Configuration
subtitle: of the operating system
menubar: door_menu
show_sidebar: false
toc: true
---

## Image selection

I use the Raspberry Pi OS Lite (64bit) from the [Pi Imager](https://www.raspberrypi.com/software/). It can be found under Choose OS -> Raspberry Pi OS (Other)

I recommend configuring your wireless network and SSH in the imager to avoid having to plug in screens or keyboards

### Upgrading the OS

This is probably resolved, however I had some hardware issues with the latest images so I used the legacy image and then [manually upgraded the OS](https://gist.github.com/jauderho/6b7d42030e264a135450ecc0ba521bd8)

On a Pi4 I also had to change the wpa_supplicant service

```shell
sudo vim /lib/systemd/system/wpa_supplicant.service
ExecStart=/sbin/wpa_supplicant -iwlan0 -c/etc/wpa_supplicant/wpa_supplicant.conf -u -s
sudo systemctl daemon-reload
sudo systemctl restart wpa_supplicant.service

Then needed to install: sudo apt install dhcpcd5
sudo systemctl enable dhcpcd
sudo systemctl start dhcpcd

sudo vim /etc/avahi/avahi-daemon.conf
allow-interfaces=eth0,wlan0
```

## Install dependencies

```shell
sudo su

apt update
apt upgrade && apt dist-upgrade

# a decent editor
apt install vim

# IO pin control
apt install gpiod libgpiod-dev

# network utils
apt install dnsutils

# pijuice_cli tool
apt install pijuice-base
```

To install docker-compose the [apt repository needs to be added](https://docs.docker.com/engine/install/debian/#install-using-the-repository)

```shell
apt update
sudo apt install docker-compose
```

Then follow the [post install instructions](https://docs.docker.com/engine/install/linux-postinstall/) to have docker available for your user.

## Configure environment

I like vim so I set it up to work without mouse support:

```shell
echo "set mouse-=a" >> ~/.vimrc
sudo su
echo "set mouse-=a" >> ~/.vimrc
```

then I ensure it's my default editor

1. run `select-editor`
2. select vim-basic

add any additional wireless networks

```shell
vim /etc/wpa_supplicant/wpa_supplicant.conf

# network={
#         ssid="deploy-network"
#         psk="password"
# }
```

## Reduce power consumption

### Disable CPU cores

Run on a single CPU core as this can half the power consumption of the device

* Edit `/boot/cmdline.txt` file and add `maxcpus=1` after `console=tty1`
* Run: `lscpu` to confirm

### Disable the power LED

Add to `/boot/config.txt` for the PI Zero W

```
# Disable the ACT LED on the Pi Zero.
dtparam=act_led_trigger=none
dtparam=act_led_activelow=on
```

For other PIs see: https://www.jeffgeerling.com/blogs/jeff-geerling/controlling-pwr-act-leds-raspberry-pi

### Disable HDMI

Use: `sudo raspi-config`, select Display and blank display

## Configure access to GPIO

This will allow your user and the door control service to interact with the GPIO chips on the Pi.

```shell
git clone https://github.com/stakach/ladies-first-chicken-door
cd ladies-first-chicken-door
./scripts/setup_gpio.sh
```

This script does the following:

1. creates a gpio-users users group
1. adds the current user to the group
1. adds a [udev rule](https://opensource.com/article/18/11/udev) allowing read and write access

This ensures secure access to the hardware

## Configure PiJuice

The [PiJuice Power Management HAT](https://github.com/PiSupply/PiJuice) requires a schedule to be in place for it to start the Pi if it's goes offline. This means we really need to ensure the schedule is set and the example scripts can be used to ensure this is done (see the scripts for more details).

Using the `pijuice_cli` configure the following:

1. configure the battery type that is installed
1. configure the full path to `pi_juice_poweroff.py` script as `USER FUNC1`
1. configure System Events so that Low Charge executes USER FUNC1
1. I also disable the physical buttons
