---
layout: page
title: Scripts
subtitle: startup and cron jobs
menubar: door_menu
toc: true
show_sidebar: false
---

## Startup

When the computer boots we need to do a few things:

1. ensure the Pi's clock is correct as there is no real time clock
1. based on the current time, open or close the door (if state is based on a schedule)

You can modify the example [./boot_script.sh](https://github.com/stakach/ladies-first-chicken-door/blob/master/scripts/boot_script.sh) to meet your needs and install a service to execute the boot script

```
[Unit]
Description=Configure door state for current time
Wants=network-online.target systemd-timesyncd.target
After=network-online.target systemd-timesyncd.target

[Service]
Type=oneshot
ExecStart=/home/steve/boot_script.sh

[Install]
WantedBy=multi-user.target
```

To install the service:

```shell
# in case it's not enabled
sudo systemctl enable --now systemd-timesyncd.service

sudo cp ./ladies-first-chicken-door/scripts/door_boot.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable door_boot.service
sudo systemctl start door_boot.service
```

## CRON jobs

Configure CRON jobs to run throughout the day: `crontab -e`

```
# open at 10am
0 10 * * * /home/steve/ladies-first-chicken-door/scripts/door_sensor.sh

# close at 9pm (door will have sensor closed already)
0 21 * * * /home/steve/ladies-first-chicken-door/scripts/door_close.sh

# power off the computer at 9:30pm
30 21 * * * /usr/bin/python3 /home/steve/ladies-first-chicken-door/scripts/pi_juice_poweroff.py

# ensure the power schedule is configured (every hour)
0 * * * * /usr/bin/python3 /home/steve/ladies-first-chicken-door/scripts/pi_juice_poweron_config.py
```
