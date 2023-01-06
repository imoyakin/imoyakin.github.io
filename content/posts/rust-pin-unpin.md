---
title: "Rust Pin Unpin"
date: 2020-10-31T10:03:02+08:00
draft: false
tags: [rust, smart pointer]
categories: [rust]
---

# rust的Pin指针

pin指针是一个在销毁前永远不会移动的指针。配合该指针可以更让结构体中包含指向它自己内存的指针--因为移动了后结构体中的这个指针就会变成悬垂引用，这就与rust安全的设计相冲突。也就是说，声明pin指针就是创建一个在生命周期结束前不会被移动的内存。

## 为什么需要Pin指针
rfc中明确的概念是，提了一个使用futures(async/await)的例子：正在poll的future结构体，希望其能保存指向自己的引用。事实上我并没有明白其想表达的意思，但结合其他[博文](https://zhuanlan.zhihu.com/p/67803708), 大致明白了需求的具体背景情形。

> 实现async/await函数时，由于需要记录当前所处的状态(每次await的时候都会导致一个状态)，所以编译器往往生成的是一个匿名的enum，每个enum变体保存从外部或者之前的await点捕获的变量。

在这篇文章的例子中，提到了await前的会被从stack中清理，await后能调用到被清理的数据是因为这些数据保存在了编译器生成的匿名enum，其充当了虚拟栈。如果await前有一个变量v，再有一个引用变量v的rv，那么这俩都会保存在这个匿名enum中，即这个enum发生了自引用的情景，也是pin指针要发挥作用的情景。

## 题外？为什么是pin指针？
rfc中提到了解决自引用的另外有趣的办法。他们想设计一个名为`move`或`?move`的trait，只有实现这个trait的类型才有搬家的权力或者被禁止搬家。但是他们认为会带来使得一些API无法向后兼容的改变，或者或者因为过于灵活带来隐患。但这些个人认为或许在语言发展到一定程度后才能看出那个更优，而现在则感觉rust会加入太多的指针（丢人的表示写了后面忘了前面，预定一个指针总结笔记）,所以毒奶一口会在rust2中回归（x

参考资料:
1. [Pin概念解析](https://zhuanlan.zhihu.com/p/67803708)
2. [rust rfc 2349](https://github.com/rust-lang/rfcs/pull/2349)
3. [The Why, What, and How of Pinning in Rust //需要复习](https://www.youtube.com/watch?v=DkMwYxfSYNQ)