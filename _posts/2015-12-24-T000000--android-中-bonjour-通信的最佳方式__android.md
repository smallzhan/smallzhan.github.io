---
layout: post
title: Android 中 Bonjour 通信的最佳方式
comments: true
excerpt:
categories:
  -  android
tags:
  -  android
---


# 背景， Zeroconf

`Bonjour` 是 `Zeroconf` 在 `Apple` 上的实现， `Zeroconf` 看名字就很屌， `zero-configuration` ，也就是零配置，主要解决的是如何动态发现一个网络中具有各种服务的设备，连接这种设备完成对应的功能。一般情况下，当网络中的设备连上网时会得到一个 `ip` ， `ip` 要么是手动配置的固定 `ip` ，要么是自动获取的动态 `ip` 。在网络中设备需要互联时，需要知道对方的 `ip` ，一个方案是手动设置网络中各设备的 `ip` 地址并记录起来。但是在一个动态网络中， `ip` 会经常变化，另一个方案是采用 `DNS` 系统，设置一个解析器，并且将不同服务的设备起一个容易记住的别名，在访问别名时，通过 `DNS` 的解析获得真实的 `ip` 。

对于大型网络，这么做没问题，也工作得很好。如果只是一个小小的局域网，甚至是临时组起来的网络，并没有条件来建议一个 `DNS` 服务器来提供服务。比如大家到了一个地方，连上了那里的无线，想使用网络中的打印机，一个方案是询问打印机 `ip` 然后连接，=ip= 一直变的话体验应该会很酸爽。

`Zeroconf` 就是为了解决这个问题，在一个动态的网络中，动态的发现各种服务。简单说来， `Zeroconf` 实现了不需要配置在网络中注册，发现设备的一套方法，类似一个分布式动态的 `DNS` 系统。

`Zeroconf` 在各系统下有不同的实现，主流操作系统中， `Apple` 里面是 `Bonjour` ， `unix-like` 系统里面是 `avahi` ， `windows` 系统中是 `LLMNR` 。


# Bonjour 的应用和开发环境

`Zeroconf` 类的实现一般都是通过 `DNSSD` ，即基于 `DNS` 的服务发现，与中心化的 `DNS` 系统不一样，是一个优化的专门用于发现来服务分布式方案。

`Apple` 的设备上大量的使用 `Bonjour` 协议， `Safari， iChat，Message` 都可以用它来发现附近的设备或进行 `p2p` 的连接。 `Android` 中使用也比较多，例如 `chromecast` 推送等。 `Apple` 开源了实现 `Bonjour` 的 `mDNSResponder` ，并且提供 `c` ， `java` 的绑定。 `Android` 呢，从 `4.1` 开始， `SDK` 中提供了 `NsdService` 的系统服务，可以用作设备发现。官方文档也比较齐全，根据示例代码能很快的写出一个发现设备的服务。

但是，但是，但是，重要的转折要说三遍。 `Android` 中的 `NsdService` 实现是不全的！缺的是其中最重要的一个东西，即 `txtRecords` 。通过 `Android` 的库写出来的应用，只能简单的发现设备，得到其名字等简单信息。而没有重要的附加信息 `txtRecords` 。 `txtRecords` 简单来说就是设备可以共享的一些附加信息，可能包含版本号，某设备当前运运行状态，如果是游戏中的共享，可能还有游戏对手的一些信息。 `txtRecords` 非常灵活，大小一般在 `200` 字节左右，能共享一些非常有用的东西。一般来讲，使用 `Zeroconf/Bonjour` 最重要的就是获取 `txtRecords` 来得知设备的一些状态， 所以一个不能解析 `txtRecords` 的实现就好比过年没有年夜饭一样。 很久之前这个 `bug` 就被发现并报告给 `Android` 官方了，但是时光流转，这么多年来 `google` 对此没有任何改进(详见[这里](https://code.google.com/p/android/issues/detail?id=35810) 还有 [这里](https://code.google.com/p/android/issues/detail?id=136099) ， `bug` 从 `2012` 年提到了 `2015` 年，没有任何改进)。要在 `Android` 里面完整支持设备发现，只能使用一个老旧的 `jmDNS` 的库。然而这个库也有各种问题，首先是好久没更新了，然后虽然可以提取出 `txtRecords` ，但是对 `txtRecords` 的解析有些小问题，导致解析出来的大部分信息都是空白。需要经过一定的修改才能用。而且，这个库非常慢。。。


# Apple 和 mDNSResponder

以上两种方法都不完美，最终完美的方法，是使用来自 `Apple` 的 `mDNSResponder` ， 上面提到苹果为它做了 `java` 的封装。这个封装的原理是通过 `jni` 调用 `c` 语言实现的 `libdns_sd.so` 。很讽刺，对吧， `Android` 里面自己也使用的一个标准协议，原生提供的编程接口竟然是一个阉割版，而完整功能还需要苹果来拯救。

集成方法是去 `Android` 源码里面查询 `mDNSResponder` 的版本，然后去 `Apple` 的开源网站上获取源码，然后编译 `jni` ，再将 `java wrapper` 放到项目里面使用。

其实折腾起来似乎也蛮复杂。。。不过好在有个小哥将这个事情都做好了，包括各个平台 `jni` 库，还写了一个示例 `app` 。这个 `app` 包含了一个模块，即 `mDNSResponder` 的 `java` 绑定，在 `android studio` 下面编译后会生成一个 `aar` ，图方便的话，可以直接使用这个 `aar` 。

其他不多说了，放上那小哥的 `github` 项目地址：[BonjourBrowser](https://github.com/andriydruk/BonjourBrowser)。 以及，他还贴心的写了一个 blog 来介绍这个，见 [Bonjour in Android Applications](http://andriydruk.github.io/post/mdnsresponder/)。
