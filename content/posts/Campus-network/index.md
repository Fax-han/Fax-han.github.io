+++
date = '2025-12-15T20:58:31+08:00'
draft = false
title = '关于校园网共享机制的原理探索以及解决办法'
tags = ["校园网", "共享"]  # 🔴 注意这里一定要用等于号 =
categories=["技术"]
+++

## **前言：**

目前众多高校依旧对校园网络共享行为进行严格限制，具体表现为：

- 对每人接入校园网络的设备数量进行限制
- 如果存在接入路由器并存在多人使用时，将采取断网、封禁等惩罚措施
- 以及侵犯隐私，能查到学生的流量走向

## **原理：**

参考了[SunBK201's Blog](https://blog.sunbk201.site/posts/ua3f/)这位作者的blog之后，目前校园网常见的共享机制有以下几种：

- TTL 字段的检测
- HTTP User-Agent 头部检测
- DPI 检测技术
- IPv4 Identification 字段
- 时钟偏移的检测技术
- Flash Cookie 检测

由于Flash已经退出历史舞台，以及现代设备的IPID不会递增，所以我们把重点放在其余四种机制上。

## **TTL字段的检测：**

### 原理：

存活时间（Time To Live，TTL），指一个数据包在经过一个路由器时，可传递的最长距离（跃点数）。 每当数据包经过一个路由器时，其存活次数就会被减一。当其存活次数为0时，路由器便会取消该数据包转发，IP网络的话，会向原数据包的发出者发送一个ICMP TTL数据包以告知跃点数超限。其设计目的是防止数据包因不正确的路由表等原因造成的无限循环而无法送达及耗尽网络资源。

## HTTP User-Agent 头部检测：

### 原理：

HTTP 数据包请求头存在一个叫做 User-Agent 的字段，该字段通常能够标识出操作系统类型，例如：

![origin-ua](/images/origin-ua.png)

这是原始的UA头，不同设备发出的HTTP数据包请求头里的UA字段都不同，因此校园网很容易就能够判断出一个宿舍里面有多少台设备在用同一个账号。

## DPI (Deep Packet Inspection) 深度包检测技术：

这个检测方案比较先进，检测系统会抓包分析应用层的流量，根据不同应用程序的数据包的特征值来判断出是否存在多设备上网。

具体可参考这里：[详解深度数据包检测 (DPI) 技术 - 知乎](https://zhuanlan.zhihu.com/p/607223743)

此种方式已确认在锐捷相关设备上应用，当由于此项功能极耗费性能，因此有些学校可能不会开启此项功能。

总而言之DPI既有ua检测也有ttl检测，以及查询目标ip地址等功能，集合了很多检测手段。

PS：博主所处的学校目前还未开启。

## 时钟偏移的检测技术：

不同主机物理时钟偏移不同，网络协议栈时钟与物理时钟存在对应关系，不同主机发送报文频率与时钟存在统计对应关系，通过特定的频谱分析算法，发现不同的网络时钟偏移来确定不同主机。

因此可以参考此专利：[CN111970173A - 一种基于时钟偏移的加密流量共享检测方法与装置 - Google Patents](https://patents.google.com/patent/CN111970173A/zh)

## Flash Cookie 检测：

Flash Cookie 会记录用户在访问 Flash 网页的时候保留的信息，只要当用户打开浏览器去上网，那么就能被 AC 记录到 Flash Cookie 的特征值，由于 Flash Cookie 不容易被清除，而且具有针对每个用户具有唯一，并且支持跨浏览器，所以被用于做防共享检测。

具体看：[深信服社区-专业、开放、共享](https://bbs.sangfor.com.cn/plugin.php?id=sangfor_databases:index&mod=viewdatabase&tid=627

## IPv4 Identification 字段：

IP 报文首部存在一个叫做 Identification 的字段，此字段用来唯一标识一个 IP 报文，在实际的应用中通常把它当做一个计数器，一台主机依次发送的IP数据包内的 Identification 字段会对应的依次递增，同一时间段内，而不同设备的 Identification 字段的递增区间一般是不同的，因此校园网可以根据一段时间内递增区间的不同判断出是否存在多设备共享上网。

具体可以参考此专利：[基于 IPID 和概率统计模型的 NAT 主机个数检测方法](https://patents.google.com/patent/CN104836700A/zh)

## 解决办法：

以博主所处学校为例子，只需要固定TTL以及修改UA，移除TCP时间戳便可以解除共享机制。

### 那么开始实践：

博主所用的设备是从咸鱼120淘回来的x86一体机主板，i3-5005u，卖家还送了4g内存以及32g硬盘还有一个网卡，对于软路由来说以及十分豪华了。

#### STEP1:

给x86主机刷上软路由系统，op，ikuai什么的都可以，博主这里用的istoreos

可在这里找到相关资料：[iStoreOS - x86_64 iStoreOS固件](https://site.istoreos.com/firmware/download?devicename=x86_64)

#### STEP2:

然后我们下载UA3F插件

下载地址：[SunBK201/UA3F: Advanced HTTP Header Rewriting Tool](https://github.com/SunBK201/UA3F)

新版的UA3F已经集成了UA修改，TTL固定以及抗DPI检测和移除TCP时间戳的功能了，也就是说一个插件就可以实现所有功能。

http://ua.233996.xyz/然后用这个来检测ua头是否被修改了。

#### STEP3:

开启UA3F，连接上网络，登录一个账号，和舍友享受500Mbps的网络。

由于博主用的ax211连接，路由器用的wifi5，所以速度只能在400多Mbps左右徘徊了

![download.png](/images/download.png)

## 最后：

最后感谢前辈的无私奉献 [SunBK201's Blog](https://blog.sunbk201.site/posts/crack-campus-network/)

后面还会有关于加密流量以及解释原理的blog……
