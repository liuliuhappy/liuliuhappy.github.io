---
layout: post
title:  Openswan+xl2tpd���ڼ�
date: 2012-07-23 18:12:35
categories: linux
---

����ʱ���о�����ο�ѧ������ÿ����½ITer����ľ�����̫������˽ʾ���֭Ͷ��������������Ÿ���������ӣ����д������������Ÿ���������ӵ�ʹ���ֲ�����������̫�˷��ˣ�̫�˷��ˣ��Ҿ�����Ǯ���֣��Խ�VPS+L2TP VPN������ǵ���һ������Ŀ�����������ͻ���5+��Сʱ����󻹽�������ǿ�Ľ�������⣬����������
<br>
���ԣ�����һƪCentos 6 X64��L2TP����ӱʼǡ�

####���ѵ�1��openswan��xl2tpd�ļ���������

����IPSEC��openswan�Ĵ����λ�ã�https://github.com/xelerance/Openswan
����L2TP��xl2tpd�Ĵ����λ�ã�https://github.com/xelerance/xl2tpd

�������ѵ���L2TP����̡̳�����һ�仰������openswan��xl2tpd�İ汾���������⡱������㿴���Ӵ����URL�Ͽ�����ͬ��һ�ҹ�˾��xelerance �������˾��裬��ʹ�����ͨһ����һֻ���Ѿ��ڿ����ˣ���������git pullһ�����´����Լ������ܱܿ����������⣬��Ǹ���ִ��������������ȶ��������ˣ�������������yum��װ��������ϵͳ�Ƽ�Ĭ�ϰ汾�������˰�û�м��������˰ɣ�Ȼ���㻹�Ǵ������������Ѿ���û�������ˡ�

`fxxx xelerance ������`

�������Ĳ����ݱ��ֶ���xl2tpd�ղ�����Ӧ��openswan�����������ݰ����ڿ���openswan��DEBUGģʽ���ܴ�/var/log�п�����һĻ��openswan�ɹ�������ip��������Զ�̿ͻ��˽��������ӣ�Ȼ����ô���޷���xl2tpd��ϵ�ϣ����ĬĬ�ȴ�10���Ӻ������ж��˿ͻ������ӡ���xl2tpd������DEBUGģʽ�ķ����� xl2tpd -D��ִ�к��ܹ۲쵽��������Ӧһ���޹��ĵȴ������

�ڳ��Ա��������汾֮������������΢�ȶ�������ǣ�x64����
Openswan 2.6.38 + xl2tpd 1.3.1

####���ѵ�2��ipsec verify

�ڳ��� Openswan 2.6.39��ipsec verify����������֤����ʱ�����ϴ�����ʾ����TEST INCOMPLETE�����о�����Ӧ����ĳ�μ��ű�������ֱ�ӿ�Դ��ɣ�master��֧ programs/verify/verify.in��ԭ���ǿ����߼ƻ���ipsec verify�ļ��ű���Perl��ֲ��Python������ĳ���⻹û�����ꡭ��

    #Check if any NAT or MASQUERADE would accidentally mess up our packets
    def natcheck():
    printfun("Checking NAT and MASQUERADEing")
    print_result("WARN","TEST INCOMPLETE")

�ǳ��򿪷�INCOMPLETE�������ǲ��Խű�ִ��δ��ɡ�


####���ѵ�3����ȫ©��

openswan��2013�괺��������Σ©����CVE-2013-2053
http://www.securityfocus.com/bid/59838/info

���������2.6.39�вŵ��Բ��ϣ����ð汾��xl2tpd�м��������⡭��

####���ѵ�44��ϵͳ�ȶ�������

������û���ҵ�xl2tpd��openswan�������ϣ���ʱ�ͻ��˷���VPN����ʱ���ᴥ��һ���ں˿�ָ�����Ȼ��BANG��ϵͳ��ը�ˡ�XEN�����������ը�˸Ͻ��ó�ɨ�������һ�ص���Դ��ɨ�ջ��ˣ�Ȼ������������
���ڱ�ը��Ƶ�ʲ����ߣ�������̨VPS��������ϵͳ����������ʱ���Ѿ���ʱ��������ʱ���ˡ�

    /var/log/messages
    kernel: BUG: unable to handle kernel NULL pointer dereference at 0000000000000020


