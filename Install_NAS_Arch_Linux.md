---
title: 'Install NAS on Arch Linux'
date: '2023-03-25'
tags: ['arch Linux', 'NAS']
draft: false
summary: ' install a NAS on my Arch Linux .'
---

# Install NAS on Arch Linux

I decided to install a NAS on my Arch Linux by myself, just like I did with my router.

## Partitioning

Assuming the disk is `/dev/sda`, let's partition it and change it to Btrfs. I prefer to use LVM mode, even though I only have one disk. However, if I want to add more disks in the future, it can be done easily.

```shell
fidsk /dev/sda
n # create a new volume
p # partition
1 [or enter] # change the volume number
enter #keep default
enter #allocate the whole disk
t # change the type
8e # lvm mode
w # write the change
```

## Mounting

Before changing the file system, you need to install Btrfs.

```shell
mkfs.btrfs -f /dev/sda1
mount /dev/sda1 /mnt/data/
btrfs subvolume create /mnt/data/nas
mount -o compress=zstd -o subvol=nas /dev/sda1 /home/<username>/nas	
```

Now, we need to add the mount information to `fstab`.

```shell
pacman -S arch-install-scripts
genfstab -U /
```

Copy the last line about NAS and add it to `/etc/fstab`.

## SMB

Use `pacman -S samba` to install it and configure the fil

```code
[global]
workgroup = WORKGROUP
security = user
usershare owner only = false
public = yes
browseable = yes
[ceNAS]
comment = My NAS # your descirbe
path = /home/<username>/nas # your own user name ,the same as where you mount the disk
read only = no
writeable = yes
browseable = yes
guest ok = yes
public = yes
create mask = 0775
directory mask = 0777
vaild users = <username> # who can visit it
```

then start the service

```shell
sudo systemctl enable smb.service
sudo systemctl start smb.service
```

Now, you can visit your NAS from the Windows network -> map the networking driver or use an app that supports the SMB service on your phone. Where the source location is`ip:445` But be careful of the notorious [EternalBlue - Wikipedia](https://en.wikipedia.org/wiki/EternalBlue). Never expose your port to the public.

## syncthing

[Syncthing](https://syncthing.net/) is an open-source tool that helps you sync files from different devices such as Linux, Android, Windows, and iPhone.

You can download it to your device. As for Linux, you can install it using `sudo pacman -S syncthing` and modify the config file. You need to change the listen address to `0.0.0.0` on your server so that you can visit the web GUI on your browser.

```code
   # vim /etc/syncthing/config.xml
   <gui enabled="true" tls="false" debugging="false">
        <address>0.0.0.0:8384</address>
        ...
    </gui>
```

Next, you need to add a service file for Syncthing:

```code
# sudo vim /etc/systemd/system/syncthing.service
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization
After=network.target

[Service]
User=<username> your current user
ExecStart=/usr/bin/syncthing -no-browser -no-restart -logflags=0
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog

[Install]
WantedBy=multi-user.target
```

Then start the service:

```shell
sudo systemctl enable syncthing
sudo systemctl start syncthing
```

Now you can configure Syncthing by visiting `ip:8384` in your browser. Syncthing has two main parts to manage: files and devices. For files, you can add the files you want to share. For devices, you need to add the device ID which you can get form ths syncthing on your device, and then add an IP address. You need to add the IP address for your NAS, but leave it as default dynamic on your phone.

In my school network, I can sync the file everywhere as long as I connect to the school wifi, for the NAS is in the internet.

## Conclusion

When I read the manual, I found it difficult to understand without a picture to illustrate. However, when I wrote this blog, I think you can get it :anger: Otherwise, read the documentation.

