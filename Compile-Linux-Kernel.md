---
title: 'Compile Linux Kernel'
date: '2023-04-29'
tags: ['arch Linux', 'kernel','i226V']
draft: false
summary: 'update linux kernel to 6.3 to fix the net card driver problem'
---

# Compile Linux Kernel

## motivation

As you may know, I had previously installed a router on my Arch Linux, but I noticed that it hangs almost every day. I checked the `journalctl` and `dmesg` logs, but couldn't find any relevant information. So, I decided to connect a monitor and patiently observe for a day. Finally, I discovered the error message "adapter reset."

## Finding solution

Initially, I came across some articles where other individuals experienced the same error message, such as [Solved: Having issues with I225-V Ethernet Adapter](https://community.intel.com/t5/Ethernet-Products/Having-issues-with-I225-V-Ethernet-Adapter/td-p/1304281) and [i226v crash](https://community.intel.com/t5/Ethernet-Products/I226-v會一直瞬間斷線/m-p/1479547). This confirmed that the problem lies with the network card driver. Unfortunately, there is no driver available for my network card (i226v), so it's advisable not to install Linux on the latest hardware. However, I was fortunate to discover that this bug has been fixed in [Linux Kernel 6.3](https://github.com/torvalds/linux/commit/1d1b4c63ba739c6ca695cb2ea13fefa9dfbff60d).

## Compiling the kernel

Compiling the Linux kernel is not as challenging as putting an elephant on a ridge. It involves just three steps: 1. `make xxxconfig`, 2. `make`, and 3. `make install`. Since I cannot put an elephant on a ridge, I deem it easier to compile a Linux kernel.

### downloading the kernel

Begin by downloading the latest kernel from [The Linux Kernel Archives](https://www.kernel.org/) and then execute the following commands:

```
tar xvf  <your kernel version>
cd <your kernel version>
```



### make config

To obtain your old config, you can use the following command:

```shell
 zcat /proc/config.gz > .config
```

Alternatively, you can use `make menuconfig` to choose the necessary options using a graphical interface. There are many articles available on this topic.

<font color='red'>If you are using Arch Linux like me, you need to change the default config as mentioned in [Kernel build fail with 'btf_encoder__encode: btf__dedup failed!'](https://lore.kernel.org/bpf/20230418214925.ay3jpf2zhw75kgmd@treble/T/)</font>

```code
CONFIG_X86_KERNEL_IBT=n
```

### make

This step is straightforward. Simply type `make` and wait patiently. The time required depends on your CPU.

### make install

You need root privileges to execute the following command:

``` 
sudo make install
```

The kernel will be installed in the `/boot` directory. Then, use the following command:

```
sudo make module_install
```

to install the module to `/lib/module`

Afterward, you should generate an initramfs for your Linux kernel. Skipping this step will result in a root filesystem without a kernel. You can find your Linux kernel version by using `ls /lib/modules/`. In my case, the version is `6.3.0`.

```shell
sudo mkinitcpio -k 6.3.0 -g /boot/initramfs-linux-6.3.img
```

### Changing the Boot Configuration

Lastly, you need to modify the boot configuration to boot into the compiled kernel.

```code
# sudo nvim /boot/loader/loader.conf
default arch63
timeout 3
editor 1
```

Then, add a new entry for the new kernel:

```code
# sudo nvim  /boot/loader/entries/arch63.conf
title Arch Linux 6.3
linux /vmlinuz-linux-6.3
initrd /initramfs-linux-6.3.img
initrd /intel-ucode.img
options root=/dev/mapper/vol0-lv_root rw
```

Finally, reboot your system. You will now be running the new kernel. You can verify the kernel change by using the command `uname -a`.

Remember, compiling and installing a custom kernel can have risks and may cause instability or compatibility issues with your system. It's always recommended to backup your data and ensure you have a fallback option in case any issues arise.

Happy kernel compiling!