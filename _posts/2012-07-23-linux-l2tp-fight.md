---
layout: post
title:  Linux L2TP/IPsec Server搏斗记
date: 2012-07-23
categories:
- Develop
tags:
- linux 
- vpn
- l2tp
---

###前期准备

如果期望能“轻点鼠标啪啪啪”解决问题，必须出门左转去隔壁微软家。

如果期望用某个bash脚本能BIU一下一站式解决L2TP搭建问题……目前还没有找到合适的。

> * xiaolai的脚本：https://www.linode.com/stackscripts/view/?StackScriptID=2660 ，其中的l2tp-control包在新版xl2tpd中已经不再需要自己单独编译了。

> * vpsyou的脚本： http://www.vpsyou.com/2010/10/04/l2tp-vpn.html ，包也有些老了。
>
>  另外，这两个脚本都忽略了“iptables的感受”。

如要坚持完美主义来搭建L2TP服务器，需要彻底弄懂下面知识点：

1. iptables 如果这关过不去，会被搜索出来的各种自说自话的L2TP搭建教程弄晕。
2. 负责IPsec的openswan，代码库:https://github.com/xelerance/Openswan
3. 负责L2TP Server的xl2tpd，代码库：https://github.com/xelerance/xl2tpd
4. ppp
5. L2TP/IPsec流程：http://en.wikipedia.org/wiki/Layer_2_Tunneling_Protocol

这过程中的坑是如此之多，所以弄清如何看到DEBUG信息是非常重要的

openswan开启DEBUG：
`vim /etc/ipsec.conf`
> config setup 节下添加：
>
>     klipsdebug=all
>     plutodebug=all
> 会在/var/log/中记录大量有用信息

xl2tpd开启DEBUG：

直接运行：`xl2tpd -D` DEBUG信息会输出到终端。

###经验1：ipsec verify

在尝试 Openswan 2.6.39的ipsec verify进行配置验证测试时，遇上错误提示：“TEST INCOMPLETE”，还是直接看源码吧：master分支 programs/verify/verify.in，原来是开发者计划把ipsec verify的检测脚本从Perl移植到Python，但是该检测还没移植完。

    #Check if any NAT or MASQUERADE would accidentally mess up our packets
    def natcheck():
    printfun("Checking NAT and MASQUERADEing")
    print_result("WARN","TEST INCOMPLETE")

是程序开发INCOMPLETE，而不是测试脚本执行未完成。
从搜索引擎的结果看，思路被卡在ipsec verify的人相当多，如果不明白发生了什么，还是去看源码吧，2.6.39的检测脚本由Python完成，2.6.38及以前由Perl编写的，它们做的大致都是执行某个命令，然后解析结果。


###经验2：openswan与xl2tpd的兼容性问题

负责IPSEC的openswan的代码库位置：https://github.com/xelerance/Openswan
负责L2TP的xl2tpd的代码库位置：https://github.com/xelerance/xl2tpd

从代码库上看他们同属一家公司：xelerance，从而让人对它俩之间的兼容性放松了警惕。其实它俩之间的兼容问题是让人最头大的。

我遇到的不兼容表现都是xl2tpd收不到本应由openswan送上来的数据包，在开启openswan的DEBUG模式后，能从/var/log中看到这一幕：openswan成功解密了ip包，并与远程客户端建立了链接，然后怎么都无法与xl2tpd联系上，最后默默等待10秒钟后无奈切断了客户端连接。与此同时，`xl2tpd -D` 一脸无辜的等待在那里，什么包都没拿到。

在尝试编译了许多版本之后，我遇到的稍微稳定的组合是（x64）：
Openswan 2.6.38 + xl2tpd 1.3.1



###经验3：安全漏洞

openswan在2013年春爆出个高危漏洞：CVE-2013-2053
http://www.securityfocus.com/bid/59838/info

但这个洞在2.6.39中才得以补上，但该版本与xl2tpd 1.3.1有兼容性问题……


(待续)