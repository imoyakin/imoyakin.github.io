---
title: "Test"
date: 2022-02-28T16:10:18+08:00
draft: true
tags: [windows, WSL2]
categories: [windows]
---
# 释放wsl占用的对于硬盘空间

1. 要找到wsl的vhd文件: ext4.vhdx(%PROFILE%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\).
2. 打开PowerShell, 输入命令：
```powershell
wsl --shutdown
diskpart
# open window Diskpart
select vdisk file="C:\WSL-Distros\…\ext4.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

> https://zhuanlan.zhihu.com/p/358528257
> https://docs.microsoft.com/en-us/windows/wsl/vhd-size