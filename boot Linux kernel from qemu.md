---
title: 'boot Linux kernel from qemu'
date: '2023-12-21'
tags: ['arch Linux', 'linux kernel']
draft: false
summary: 'boot Linux kernel from qemu'
---

# boot Linux kernel from qemu

I have compiled 

## compile Linux kernel

just three steps as usual 

```bash
make defconfig # or any config you like , just  keep it easy
make # then wait a long time .......
make modules_install # to generate iniramfs.img
```

## prepare the initramfs.img

in case ,execute `fakeroot` ,

```
mkdir roots
```

create a file 

```code
# mkinitcpio.conf
# Required
MODULES=(ext4)
HOOKS=(base systemd modconf sd-vconsole filesystems block keyboard)
```

then generate `initramfs.img`

```
mkinitcpio -k /path/to/linux/arch/x86/boot/bzImage -c mkinitcpio.conf -g initramfs.img
```

## create hard disk

```
dd if=/dev/zero of/hd0 bs=1M count=2048
mkfs.ext4 hd0
```

## install basic OS 

I use Arch bty, so 

```
sudo pacman -S arch-install-scripts
sduo mount hd0 /mnt
sudo pacstrap /mnt base
sudo passwd --root /mnt root
sudo umount hd0
```



## just boot up

```
qemu-system-x86_64 -kernel /path/to/linux/arch/x86/boot/bzImage -initrd initramfs.img -m 1G -hda hd0 -append "root=/dev/sda"
```

## change passwd

if you  still can't login , you can change the password of root 

```
mkpasswd -m sha-512 # input the password as "1234" as example
mount hd0 /mnt 
```

sudo vim /mnt/etc/shadow,find root, then password will become "1234"

```
root:$6$EiCKr/sm5E2C8QGE$0XJxLSfzgddy/syMhyEjo83OY5d3247NPnNKPSqz.7KJzvIp6d/cl2JP6XDh9iWlFE4RwJrfr4ZKst8ZXU3dj/:19631::::::
```

