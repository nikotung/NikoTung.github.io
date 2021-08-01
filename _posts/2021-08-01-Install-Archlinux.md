--- 
layout: post
title: "Install Archlinux"
description: ""
category: "2021-08"
tags: [Archlinux,install]
---


## Device

* Lenovo ThinkPad E14

## Install Guide

Basically, all the guide from [archlinux wiki](https://wiki.archlinux.org/title/Installation_guide) work quiet well for me.

Notes:

* I didn't use a swap partion, swapfile instead,[wiki](https://wiki.archlinux.org/title/Swap#Swap_file)


## Post-installation

* Desktop Environment, using [plasma KDE](https://wiki.archlinux.org/title/KDE#Plasma)


## Issue run into during configuration

#### signature from "lilac (build machine) <lilac@build.archlinuxcn.org>" is unknown trust

You may run into this issue while installing from AUR, just update the archlinux key ring.

	sudo pacman -S archlinuxcn-keyring
 
### mount a second disk

mount a second disk and make it auto mount after restart.

First, find the identify of your disks `lsblk -f`, you will see the UUID of your disk.

	NAME        FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
	sda                                                                                
	└─sda1      ext4   1.0   data  3ca0dbc1-fa30-44dc-900c-2c5d66a51a33  843.1G     3% /home/niko/data
	nvme0n1                                                                            
	├─nvme0n1p1 vfat   FAT32       3F7C-4045                                           
	└─nvme0n1p2 ext4   1.0         5bb27951-03e8-42cc-9294-9fcb1ff1ef93    163G    25% /

Second, update the fstab content in `/etc/fstab`,add a new line to the file.

	UUID=3ca0dbc1-fa30-44dc-900c-2c5d66a51a33      /home/niko/data  ext4            rw,relatime     0 1

* UUID: the uuid of your disk from the first step
* dir: the directory to which your disk mount

Detail from [wiki](https://wiki.archlinux.org/title/Fstab#Identifying_filesystems)

### install fcitx5

Setup the environment in the `~/.pam_enviroment`. Detail from [wiki](https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

	GTK_IM_MODULE DEFAULT=fcitx
	QT_IM_MODULE  DEFAULT=fcitx
	XMODIFIERS    DEFAULT=@im=fcitx
	INPUT_METHOD  DEFAULT=fcitx
	SDL_IM_MODULE DEFAULT=fcitx

### install WeCom

Install WeCom from [AUR](https://aur.archlinux.org/packages/com.qq.weixin.work.deepin/?O=0&PP=10)

	git clone https://aur.archlinux.org/com.qq.weixin.work.deepin.git
	makepkg  # build ,make sure you install the build package
	pacman -U com.qq.weixin.work.deepin-3.1.6.3605deepin6-2-x86_64.pkg.tar.zst
	7z x /opt/apps/com.qq.weixin.work.deepin/files/helper_archive.7z # if you can not find any files in ~/.deepinwine/deepin-wine-helper

If your WeCom can not display the Chinese properly,install the following fonts.

	wqy-microhei - WenQuanYi Micro Hei font family (also known as Hei, Gothic or Dotum) is a sans-serif style derived from Droid Sans Fallback, it offers high quality CJK outline font and it is extremely compact (~5M).
	wqy-zenhei - Hei Ti Style (sans-serif) Chinese Outline font embedded with bitmapped Song Ti (also supporting Japanese (partial) and Korean characters).
	wqy-bitmapfont - Bitmapped Song Ti (serif) Chinese font.
	
From [wiki](https://wiki.archlinux.org/title/Localization/Chinese)

	pacman -S wqy-miccrohei
	pacman -S wqy-zenhei
	pacman -S wqy-bitmapfont

