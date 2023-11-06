---
title: "Docker Dev Enviroment坑中的小快乐"
date: 2023-01-06T15:02:33+08:00
draft: true
---

```dockerfile
# Use the official Arch Linux base image
FROM menci/archlinuxarm

# Set the working directory to /app
WORKDIR /app

# Update the system and install necessary packages, use tuna source
RUN echo 'Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/$arch/$repo' > /etc/pacman.d/mirrorlist \
    && pacman -Syu --noconfirm \
    && pacman -S --noconfirm nodejs yarn wget \
    && rm -rf /var/cache/pacman/pkg/*
```