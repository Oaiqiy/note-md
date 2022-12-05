---
title: "Linux Swap"
date: 2021-01-21T06:44:36Z
draft: false
tags: ["linux","swap"]
categories: ["linux"]
---

## Ubuntu

1. `sudo mkdir /usr/swap`
2. `sudo dd if=/dev/zero of=/usr/swap/swapfile bs={size} count=1000000` size: MB
3. `sudo mkswap /usr/swap/swapfile`
4. `sudo swapon /usr/swap/swapfile`
5. turn on forever `echo '/root/swap/swapfile swap swap defaults 0 0' >> /etc/fstab`
6. turn off `sudo swapoff /usr/swap/swapfile`
