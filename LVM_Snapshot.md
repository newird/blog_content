---
title: 'Resizing and creating snapshots using LVM in Arch Linux'
date: '2023-04-06'
tags: ['lvm', 'arch linux','snapshot']
draft: false
summary: 'resize the lvm volume and create a snapshot'
---

# Resizing and creating snapshots using LVM in Arch Linux

One of the best advantages of LVM is that it can easily change the size of a volume and create a snapshot to record the system. Here are some steps to help you resize and create a snapshot using LVM.

## Reducing the size 

From previous tutorials, you may have learned that reducing the size of a volume requires `umount` and `resize2fs`. However, in my recent experience, I found that these steps were not necessary. You can simply enter the command:

```shell
sudo lvreduce -L -100G /dev/mapper/vol0-lv_root
```

As you may know from the previous blog , I allocated 600GB for my `lv_root` volume, and now I want to reduce its size.

## Extending the size

Similarly, we have a command to extend the size of a volume. Using `lv_root` as an example, you can use the following command:

```shell
sudo lvextend -L 20G -n /dev/mapper/vol0-lv_root
```

This command will enlarge the `lv_root` by 20GB.

## Creating a snapshot

Before creating a snapshot, you need to check if you have enough space. You can check the available space by running `lvs`.

### create

To create a snapshot for `lv_root`, you can use the following command:

```shell
sudo lvcreate -L 5G -s -n root_snapshot /dev/mapper/vol0-lv_root
```

This command will create a snapshot for `lv_root` named `root_snapshot` and allocate it 5GB of space. Don't worry about the space, as 5GB is enough to record the root. Linux uses an inode file system, which records links to unchanged files and only records changes to files.

### recover

To recover from a snapshot in case of a crash or other issues, you can use the following command:

```shell
sudo lvconvert --merge /dev/vol0/root_snapshot
```

This command will recover from a snapshot named `root_snapshot`, which is the name we assigned earlier. You can give your snapshot a more meaningful name to make it easier to identify.

### delete

If you no longer need a snapshot that was created a long time ago, you can simply delete it using the following command:

```shell
sudo lvremove /dev/vol0/root_snapshot
```

## Conclusion

Overall, LVM is a powerful tool for managing storage volumes, and snapshots can be incredibly useful for backing up and restoring data. Be sure to check the available space before creating a snapshot, and always backup your data before making any changes to your volumes.