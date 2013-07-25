---
layout: post
title:  Openswan+xl2tpd折腾记
date: 2012-07-23 18:12:35
categories:
- Programming
tags:
- linux 
- vpn
- l2tp
---


花点时间研究下如何科学上网是每个大陆ITer必须的经历，太多聪明人绞尽脑汁投入大量精力制造着各种免费梯子，还有大量年轻人捧着各种免费梯子的使用手册读的如痴如醉。太浪费了，太浪费了，我决定花钱消灾，自建VPS+L2TP VPN，结果是掉进一个更大的坑里，光爬出来就花了5+个小时，最后还仅仅是勉强的解决了问题，但不完美。
<br>
所以，这是一篇Centos 6 X64的L2TP搭建爬坑笔记。

####困难点1：openswan与xl2tpd的兼容性问题

负责IPSEC的openswan的代码库位置：https://github.com/xelerance/Openswan
负责L2TP的xl2tpd的代码库位置：https://github.com/xelerance/xl2tpd

网上能搜到的L2TP“搭建教程”都用一句话带过“openswan和xl2tpd的版本兼容性问题”，如果你看到从代码库URL上看他们同属一家公司：xelerance 而放松了警惕，你就错了扑通一声你一只脚已经在坑里了；如果你觉得git pull一份最新代码自己编译能避开兼容性问题，抱歉你又错了现在你两条腿都掉坑里了；如果你觉得我用yum安装他们俩的系统推荐默认版本这总行了吧没有兼容问题了吧，然后你还是错了现在烂泥已经淹没到脖子了。

`fxxx xelerance ！！！`

我遇到的不兼容表现都是xl2tpd收不到本应由openswan送上来的数据包，在开启openswan的DEBUG模式后，能从/var/log中看到这一幕：openswan成功解密了ip包，并与远程客户端建立了链接，然后怎么都无法与xl2tpd联系上，最后默默等待10秒钟后无奈切断了客户端连接。让xl2tpd运行在DEBUG模式的方法是 xl2tpd -D，执行后能观察到它毫无响应一脸无辜的等待在那里。

在尝试编译了许多版本之后，我遇到的稍微稳定的组合是（x64）：
Openswan 2.6.38 + xl2tpd 1.3.1

####困难点2：ipsec verify

在尝试 Openswan 2.6.39的ipsec verify进行配置验证测试时，遇上错误提示：“TEST INCOMPLETE”，感觉这里应该是某段检测脚本，还是直接看源码吧：master分支 programs/verify/verify.in，原来是开发者计划把ipsec verify的检测脚本从Perl移植到Python，但是某项检测还没开发完……

    #Check if any NAT or MASQUERADE would accidentally mess up our packets
    def natcheck():
    printfun("Checking NAT and MASQUERADEing")
    print_result("WARN","TEST INCOMPLETE")

是程序开发INCOMPLETE，而不是测试脚本执行未完成。


####困难点3：安全漏洞

openswan在2013年春爆出个高危漏洞：CVE-2013-2053
http://www.securityfocus.com/bid/59838/info

但这个洞在2.6.39中才得以补上，但该版本与xl2tpd有兼容性问题……

####困难点44：系统稳定性问题

或许我没能找到xl2tpd与openswan的最佳组合，有时客户端发起VPN拨号时，会触发一个内核空指针错误，然后，BANG，系统爆炸了。XEN发现虚拟机爆炸了赶紧拿出扫帚把碎在一地的资源打扫收回了，然后重启了它。
好在爆炸的频率并不高，加上这台VPS并非生产系统，况且爬坑时间已经超时，于是暂时忍了。

    /var/log/messages
    kernel: BUG: unable to handle kernel NULL pointer dereference at 0000000000000020


