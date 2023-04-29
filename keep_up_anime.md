---
title: 'anime wo ou'
date: '2023-04-12'
tags: ['nas', 'anime','qBittorrent']
draft: no
summary: 'keep up with anime series using RSS'
---

# anime wo ou

## Introduction

As the 2D culture has been dead for a long time in China, there are still ways to keep up with anime series. In this blog post, I will share some tips on how to do just that.

## Downloading qbittorrent-nox

To download qBittorrent-nox, enter the following command:

```shell
sudo pacman -S qbittorrent-nox
```

Once installed, enter `qbittorrent-nox` to start the application. You will need to agree to the terms by entering 'y'. After that, you can open it at `ip:8080`. You can change the port later.

## Setting Up Auto Download

To enable auto-downloading, click on the Tools menu, then Options, then RSS. Check the box next to "Enable auto-downloading".

![auto download](/static/config_auto_download_RSS.png)

## Finding the RSS Source

I use [acg.rip](https://acg.rip/)  (good name) as my RSS source. To find the RSS source for the anime you want to keep up with, type the name of the anime into the search bar and copy the URL to RSS downloader. Don't forget to change the default download location. 

When I was undergraduate ,I have a class named XML application in which the teacher told us RSS has been  left behind by the times ,we pick it up.

Also, use [XIU2/TrackersListCollection](https://github.com/XIU2/TrackersListCollection) as trackers (although I don't know its effect :smile: ). You can configure it in Options > BitTorrent > Automatically add these trackers to new downloads.

## Mounting on Alist

I use alist (maybe I need another blog later )on my NAS to watch the anime I download. To install alist, read the official manual. Then, go to the storage and add a local disk, choosing the location where you download the anime. I use potplayer to watch the video on alist.  Here's the final result:

![finall effective](/static/final_effect_of_qb_nas.png)

