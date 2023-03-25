---
title: 'Install  Arch Linux'
date: '2023-03-22'
tags: ['arch Linux', 'LVM']
draft: false
summary: ' install Arch Linux .'
---

# install arch linux

## Download the ISO file

Download the ISO file from the [official website](https://archlinux.org/download/), and then burn it to a USB disk. As USB devices become larger and larger, it is a waste to use just one disk to install an operating system. Therefore, I use [Ventoy](https://www.ventoy.net/en/download.html), which allows me to install multiple ISO files on the same USB drive.

## Partition the disk

I prefer to use `fdisk`. First, find the disk that needs to be partitioned by running the following command:

```shell
fdisk -l
```

Assuming that the disk you want to partition is `/dev/sda`, you can now partition it using the following commands:

now ,let;s part it

```shell
fdisk /dev/sda
g # create a gpt version
p # partition
n #create a new partition
1 #the partition number
[enter] # just use default
+5G #ceate a 5g size volume
t # change the file
1 [or enter] # choose the first volume
1 # the file type is EFI system 
n # create another partition
[enter] # just use default
[enter] # just use default
[enter] # just use default
t # change the file
8e # i would like to ues lvm
w #write the config
```

## Create the PV group and partition it

As my disk is 1 TB, I give 600 GB to `/` and 400 GB to `/home`. However, you can choose any size that suits your needs. If you are unsure, using one partition is also acceptable. To create the physical volume group and partition it, run the following commands:

```shell
pvcreate  /dev/sda2
vgcreate vol0 /dev/sda2
lvcreate -L 600GB vol0 -n lv_root
lvcreate -l 100%FREE vol0 -n lv_home
```

## Change the disk file type

Change the file type of `/dev/sda1` to EFI System Partition (ESP), and partition `/dev/sda2` as `/` and `/home` using the following commands:

```shell
mkfs.fat -F 32 /dev/sda1 
mkfs.ext4 /dev/vol0/lv_root
mkfs.ext4 /dev/vol0/lv_home
```

## Connect to the network

It is better to use a wired connection. However, if you do not have access to one, you can use your smartphone by connecting it via USB and choosing Ethernet.

## Mount the disk

Mount the `lv_root` as `/`, `lv_home` as `/home`, and `/dev/sda1` as `/boot` using the following commands:

```shell
mount /dev/vol0.lv_root /mnt
mkdir /mnt/home 
mount /dev/vol0/lv_home /mnt/home
mkdir /mnt/boot #maybe some like to /mnt/boot/EFI but it may casue problem when use systemd-boot
mount /dev/sda1 /mnt/boot
```

## Install the base system

download the base system to `/mnt`

```shell
pacstrap -i /mnt base
pacstrap -i /mnt base-devel
passtrap -i /mnt firmware
# write the mount condition to fstab ,if will auto mount the disk after reboot
genfstab -U -p /mnt >> /mnt/etc/fstab 
```



## chroot and do basic config

Once the base system has been installed, the next step is to chroot into the new system and perform some basic configuration. This involves installing the kernel and headers, setting up network connectivity, and configuring the LVM.

To chroot into the new system, run the following command:

```shell
arch-chroot /mnt
```

###  install the kernel and headers

To install the kernel and headers, run the following command:

```
pacman -S linux linux-headers
```

You can also choose to install the linux-lts and linux-lts-headers or linux-zen and linux-zen-headers packages instead of the regular linux and linux-headers packages, depending on your requirements.

### Install NetworkManager

To enable network connectivity in the new system, install NetworkManager using the following command:

```shell
pacman -S networkmanager
```

### Configure LVM

If you have configured LVM during the installation, you need to install the lvm2 package and add it to the mkinitcpio.conf file. You can do this using the following commands:

```shell
pacman -S lvm2
vim /etc/mkinitcpio.conf
```

In the mkinitcpio.conf file, find the line that starts with `HOOKS=` and add `lvm2` to the end of the line. Save and exit the file.



```shell
pacman -S lvm2
```

### Change language settings

To change the language settings, uncomment the appropriate line in the /etc/locale.gen file. For example, to use English as the default language, uncomment the following line:

```shell 
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

After editing the file, generate the locale using the following command:

```shell
locale-gen
```

### Set the time zone

To set the time zone, create a symbolic link to the appropriate time zone file in the /etc/localtime directory. For example, to set the time zone to Asia/Shanghai, run the following command:

```shell 
ln -s /usr/share/zoneinfo/Asia/shanghai /etc/localtime
```

## Set up systemd-boot

Next, set up the systemd-boot bootloader to boot the new system.

### Install the bootloader

To install the bootloader, run the following command:

```
bootctl install --path=/boot
```

### Configure the bootloader

```shell
# vim /boot/loader/loader.conf
default arch
timeout 3
editor 1
```

This sets the default boot entry to `arch` and sets a timeout of 3 seconds before booting. It also enables the editor option, which allows you to edit the boot options before booting.

Next, create a new file called `/boot/loader/entries/arch.conf`with the following contents:

```shell
# vim /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
initrd /intel-ucode.img
options root=/dev/vol0/lv_root rw
```

## Finish installation

Once you have completed the basic configuration and set up the bootloader, you can exit the chroot environment and reboot the system. To do this, run the following commands:

```shell
exit
umount -R /mnt
reboot
```

Congratulations, you have successfully installed Arch Linux on your system!