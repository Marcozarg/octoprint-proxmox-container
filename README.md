# Proxmox octoprint container install (3D printer and webcam)
<p>

I couldn't find working instructions for 3D printer/Webcam passthrough to octoprint-container.<br>
This one worked for me and I hope this can inspire you.

## My setup
<p>
Old laptop 1CPU/4core/8GB/SSD<br>
Prusa MK3s<br>
Logitech webcam<br>
Proxmox 7.3-3
Octoprint 1.8.6

## Octoprint container
<p>
Create container for Octoprint (I used ubuntu 22.10 standard)<br>
Won't go to details here, plenty of instructions on internet.

## Proxmox host console
```
v4l2-ctl --list-devices
chmod o+rw /dev/video2
chmod o+rw /dev/ttyACM0
usb devices: lsusb

nano /etc/pve/lxc/<container ID>.conf
  lxc.cgroup2.devices.allow: c *:* rwm
  lxc.mount.entry: /dev/ttyACM0 dev/ttyACM0 none bind,optional,create=file
  lxc.mount.entry: /dev/video2 dev/video2 none bind,optional,create=file

nano /etc/udev/rules.d/50-myusb.rules
  SUBSYSTEMS=="usb", ATTRS{0x2c99}=="067b", ATTRS{0x0002}=="2303", GROUP="users", MODE="0666"
  SUBSYSTEMS=="usb", ATTRS{0x046d}=="067b", ATTRS{0x0825}=="2303", GROUP="users", MODE="0666"

rules reload: udevadm control --reload-rules && service udev restart && udevadm trigger
reboot container
```

## Octoprint installation (https://github.com/paukstelis/octoprint_install)

```
adduser octo
usermod -aG sudo octo
su octo
cd /home/octo
sudo apt install git
git clone https://github.com/paukstelis/octoprint_install.git
sudo octoprint_install/octoprint_install.sh
```

(usb camera failed to install at this point. /home/octo/ustreamer installed thought)<br>
I'm running ustreamer from octo home-dir<br>

## Service install
```
cd /home/octo/ustreamer
sudo nano /etc/systemd/system/ustreamer.service

    [Unit]
    Description=uStreamer service
    After=network.target
    [Service]
    User=octo
    ExecStart=/home/octo/ustreamer/ustreamer --process-name-prefix ustreamer --log-level 0 --device /dev/video2 --device-timeout=8  --quality 100 --resolution 800x600 --desired-fps=29 --host=0.0.0.0 --port=8080 --static /var/www/html/ustreamer/
    [Install]
    WantedBy=multi-user.target

sudo systemctl daemon-reload
sudo systemctl enable ustreamer.service
sudo systemctl start ustreamer.service
sudo systemctl status ustreamer.service
```
Link to stream is http://x.x.x.x:8080/stream
