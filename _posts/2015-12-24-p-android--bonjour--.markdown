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

Bonjour 是 Zeroconf 在 Apple 上的实现，Zeroconf 看名字就很屌，zero-configuration，零配置，主要解决的是如下的问题。网络里面的任何设备，连上网络的时候会得到一个 ip，这个 ip 可以是手动配置的，也可以是 DHCP 得到的。网络中设备需要互联的时候，需要知道对方的 ip，但是在一个动态网络中，ip 会经常变化，这样，除非手动设置各种设备的 ip 地址并记录起来，或者对设备起一个不变，容易记住的别名，直接访问别名，这就是 DNS 的原理。有了 DNS 系统，设备的访问等就比较方便了。

对于大型网络，网站等，这么做没问题，也工作得很好。问题在于，如果只是一个小小的局域网，甚至是临时连起来的网络，比如大家到了一个地方，连上了那里的无线，网络中有打印机，需要在网络中打印。一个方案是询问打印机 ip 然后连接，ip 一直变的话体验应该会很酸爽。

当然，现在的操作系统都提供一个在网络中搜寻的功能，这个功能就是靠 zeroconf 来实现的。简单说来，zeroconf 实现了不需要配置在网络中注册，发现设备的一套方法，使得需要通信，使用的时候不需要配置 DNS 或人肉记录 IP 而提供服务。

Zeroconf 在各系统下有不同的实现，主流操作系统中，Apple 里面是 bonjour，unix-like 类系统里面是 avahi，windows 里面也有，是 LLMNR，等等。

# bonjour 的应用和开发环境

zeroconf 类的实现一般都是通过 DNSSD，即基于 DNS 的服务发现，与熟悉的中心化的 DNS 系统不一样，是一个优化的专门为设备发现来服务的。

Apple 的设备上大量的使用 bonjour 协议，Safari， iChat，Message 都可以用它来发现附近的设备或进行 p2p 的连接。Android 中使用也比较多，例如 chromecast 推送等。Apple 自家有 mDNSResponder 的实现，并且提供 c， java 等帮定。而且这个项目是开源的。Android 呢，从 4.1 开始，提供了 NsdService 的系统服务，可以用作设备发现。官方文档也比较齐全，按照其中的示例代码也能很快的写出一个发现设备的服务。

但是，但是，但是，重要的转折要说三遍。android 中的 NsdService 实现是不全的！缺的确实最重要的一个东西，txtRecords，通过 android 的库写出来的应用，只能简单的发现设备得到名字等简单信息。最重要的附加信息 txtRecords 是没有的。txtRecords 是什么东西呢？简单来说就是设备可以共享的一些附加信息，可能有版本号，某设备当前运动状态，如果是游戏共享，可能还有对手的一些信息。txtRecords 非常灵活，大小一般在 200 字节左右，可以共享一些非常有用的东西。一般来讲，使用 zeroconf/bonjour 最重要的就是这个 txtRecords， 一个不能解析 txtRecords 的实现就好比过年没有年夜饭一样。longlong ago 这个 bug 就被发现并报告给 android 官方了，但是时光流转，这么多年来 google 对此没有任何改进(详见[这里](https://code.google.com/p/android/issues/detail?id=35810) 还有 [这里](https://code.google.com/p/android/issues/detail?id=136099) ，bug 从2012 年提到2015 年，没有任何改进)。要在 android 里面完整支持 bonjour 设备发现，只能使用一个老旧的 jmDNS 的库。然而这个库也有各种问题，首先就是好久没更新了，然后是虽然他可以提取出 txtRecords，也能得到对应字串，但是对 txtRecords 的解析有些小问题，导致解析出来的 Map 是空。需要经过一定的折腾才能用。而且，这个库很慢。。。

# Apple 和 mDNSResponder

以上两种方法都不完美，最终完美的方法，是使用来自 apple 的 mDNSResponder, 苹果为它做了 java 的封装。这个封装通过 jni 调用 libdns<sub>sd</sub> 来实现功能。很讽刺，对吧，android 里面自己也使用的一个标准协议，自己提供的编程接口竟然是一个被阉割的货，而完整功能还需要苹果来拯救。

不吐槽了，主要方式是去 android 源码里面查询 mDNSResponder 的版本，然后去 apple 的开源网站上获取源码，然后编译 jni，再将 java wrapper 放到项目里面使用。

额，其实也蛮复杂。。。不过好在有个小哥将这个事情都做好了，包括各个平台 jni 库，还写了一个示例 app，mDNSResponder 的 java 绑定是其中的一个模块。这个模块在 android studio 下面编译后会生成一个 aar，图方便的就可以直接使用这个 aar 库了。

多的不多说了，放上那小哥的 github 项目地址：[BonjourBrowser](https://github.com/andriydruk/BonjourBrowser), 以及，他还贴心的写了一个 blog 来介绍这个，见 [Bonjour in Android Applications](http://andriydruk.github.io/post/mdnsresponder/)
