---
title: "APFS开启透明压缩"
date: 2022-1-1T10:23:02+08:00
draft: false
tags: [APFS]
categories: [apple]
---

```bash
brew install afsctool
afsctool -c -j8 **** #要压缩的目录
```
j是开启几个线程,m系列乞丐u就是4大4小,所以选了8.
我压缩的用户空间,主要是代码,干掉了10个g.

https://apple.stackexchange.com/questions/360120/apfs-how-do-i-enable-transparent-compression


感谢土豆!