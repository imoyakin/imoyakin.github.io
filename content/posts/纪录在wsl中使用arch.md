---
title: "纪录在wsl中使用arch"
date: 2022-04-18T18:06:05+08:00
draft: true
---

### 使用清华源
```bash
pacman-mirrors --country China && pacman -Syyu
```
```todo
在Arch Linux系统中使用Archlinuxcn源(清华源)的操作步骤

1、在pacman.conf文件中增加两行代码

在终端中运行：# sudo gedit /etc/pacman.conf

然后在pacman.conf文件的最后增加以下两段代码：

[archlinuxcn]

Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

2、安装archlinuxcn-keyring包导入GPG key

之所以要安装archlinuxcn-keyring包导入GPG key是因为要安装某些软件的时候会提示gpg签名错误损坏等问题，这个是因为没有导入key的原因，所以我们需要导入一下GPG KEY，在终端中运行以下命令即可：

# sudo pacman -S archlinuxcn-keyring

3、更新一下源

在终端中运行：# sudo pacman -Sy

至此，能在Arch Linux系统中使用清华源了。
```