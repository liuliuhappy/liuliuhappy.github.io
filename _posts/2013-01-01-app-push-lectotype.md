---
layout: post
title:  手机客户端消息推送方案选型记
date: 2013-01-01
categories:
- Develop
tags:
- app
- android
- ios
---

### 前言
这是一次简单粗暴的选型过程，充斥各种资料搜集结果，记录下来还是蛮有意义的。

------
### 先说IOS

如果每次有消息就去连接苹果服务器然后发之的话，干多了苹果会视之为DDOS而封掉我们

> “Keep your connections with APNs open across multiple notifications; don’t repeatedly open and close connections. APNs treats rapid connection and disconnection as a denial-of-service attack. You should leave a connection open unless you know it will be idle for an extended period of time—for example, if you only send notifications to your users once a day it is ok to use a new connection each day.”

> ** 来源: [苹果官方IOS开发文档：RemoteNotifications节](http://developer.apple.com/library/ios/#documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/CommunicatingWIthAPS.html#//apple_ref/doc/uid/TP40008194-CH101-SW1) **

这意味着显然要有守护进程来干这个事情，我们看看豆瓣咋干的：

> “豆瓣的做法是做了个APNS的服务，基于Gevent和APNSWrapper，每个应用保持一个长连接到APPLE服务器，挂了自动重连，然后应用走一个接口push message到这个server，这个server负责push到apple server。”

> ** 来源：http://www.v2ex.com/t/23837 **

Apple那边的推送服务性能不俗：

> “苹果的apns server(gateway.push.apple.com)非常强劲,300万用户,8台服务器5分钟可以推完,我简单算了一下每秒钟推送流量= 256 * 3000000/300.0/(1024 * 1024)=2.44M byte/s,差不多占用19.5MB/s的带宽,可以用虚拟机来做,不必要浪费这么多服务器”

> ** 来源 ：http://lutaf.com/153.htm **

--------
### 再说Android

官方androidpn被国内做安卓推送服务的运营商用软文骂的狗血喷头，感兴趣请看这里：
http://blog.jpush.cn/androidpn_android_push_problem/

##### MQTT
MQTT是IBM在1999年公布的一个协议，某个消息队列服务器只要实现了MQTT协议，就可以被当成MQTT服务端。

官方介绍：http://www.slideshare.net/andypiper/introducing-mqtt

[MQTT官网](http://mqtt.org/software) 列出了一大堆支持MQTT协议的服务器，很眼熟的有：IBM WebSphere、Apache ActiveMQ、RabbitMQ……可惜没有Beanstalkd。

##### MQTT可能存在的问题：

* 问题1：架构设计

比如ActiveMQ这样的消息队列服务器在企业服务器中的地位类似数据库服务器，是非常【后端的系统】，它前面要一般隔着层层防火墙，从攻击的角度看，它天生就是脆弱的，如果任由成千上万设备从Public Network访问是不合适的。

* 问题2：安全

手机A能否冒充手机B，去收B的消息？

* 问题3：部署困难

企业内部复杂的内网安全策略，能否将流量中的MQTT协议加入白名单？

* 问题4：避开假牙服务器

如果使用IBM在官方网页放出来的RSMB，评测显示连接数1000左右服务即挂
> “RSMB is indeed limited to 1024 total open connections. This is most likely due to them using the select() call to multiplex socket connections. It could theoretically be changed by recompiling glibc on Linux to support a greater number of sockets, but not something you want to do in practice.”

> * 来源：http://stackoverflow.com/questions/7094774/how-many-users-and-push-messages-can-mqtt-handle-or-rsmb-can-handle *

>“I believe rsmb is limited to approximately 1000 clients but you should take notice of its licence which states it is for evaluation only.”

> * 来源： http://stackoverflow.com/questions/11792735/mqtt-rsmb-and-mosquitto-max-connection *

##### MQTT开源服务器：Mosquitto（蚊子）

Mosquitto的口碑很好：

> “开源的mqtt服务器，性能很棒，单机27万连接没问题。”

> * 来源：http://www.mengxiansheng.net/index.php/archives/222 *

不知Mosquitto在安全性上表现如何。

————————————————————————————————————————
#### 第三方服务

#####国内

百度家的：
http://openapi.baidu.com/wiki/index.php?title=docs/cplat/push

还有某些小厂……

发现两家国内推送服务商软文雷同，就没兴趣研究他们了，只是猜测他们会通过偷发小广告、用客户的手机刷PV挣钱吧。

智游推送： http://blog.sina.com.cn/s/blog_9bf647c1010184qg.html

极光推送： http://cloudbbs.org/forum.php?mod=viewthread&tid=8695&extra=page%3D2

#####国外
下面隆重说说国外的，我曾认为因为有墙的存在，国外的推送服务商在国内吃不开，不过貌似并非如此：

> “我是做iOS应用开发的，本来使用的urbanairship的服务，确实方便。但是一轮限免活动下来，用户很快要超过百万，这个数量已超过urbanairship提供的免费服务范围，如果继续使用push功能，价格就太贵了，实在用不起。我开始考虑自己搭建APNS，发现已不少方案，……”

> “谢谢阿，已经看过这个了。不过大家认同的还是使用parse等服务，这个确实好，但是一旦超过免费范围，那价格。。”

> “每个月100万条，对于每月群发一次的单个应用一定是够的。urbanairship的计费是只统计active token的，你会保证每个月都有100万的active token么（是的话恭喜你了）？”

> 来源： http://www.v2ex.com/t/43511

两家被国人用的最多的外国推送服务商浮出水面：

[Urbanairship](http://urbanairship.com/products/push-messaging)

[Prase](https://parse.com/plans)

尤其是Parse，刚被Facebook收购，正春风得意的服务着超过6w个APP。

这些推送服务商的特点是：

* 全平台/设备制霸，恨不得支持Nokia 1100；
* 讨好开发者，各种语言的易上手SDK；
* 取悦老板，数据挖掘分析报表样样有；
* 猪养肥再杀，月发送量<100w的免费用。