---
layout: post
title:  "方正宽带DNS问题及办公区WiFi调优记"
date: 2015-02-10 15:38:00
---

公司办公室里用了方正宽带, 并使用"关闭DHCP"的方式部署了两个[TP-Link WR842N](http://item.jd.com/836677.html)无线路由器作为AP热点, DHCP由[TP-Link ER6110](http://item.jd.com/333823.html)提供, 最上方有一台单独的有线路由器专门用于接方正的PPPoE或专线. 

## 1 "请在已购页面再试一次"

在Mac的AppStore里, 安装软件, 经常提示`请在已购页面再试一次`. 

思考了一下, 应该是鸡贼的方正宽带为了节约出口流量, 屏蔽了部分CDN的IP, 转而用内部的缓存服务器地址. 

最极端的情况是, 如果用了VPN的网络, 并且把DNS设置为`8.8.8.8`, 那么解析回来的`www.baidu.com`是百度对国际用户提供的IP地址, 而方正把这个IP地址屏蔽了. 从而就会出现拨通VPN之后, 配置国内国际路由, 反而百度访问不了的奇异情况. 

### 1.1 普通路由器的对策

由于公司的网络是由一台单独的有线路由器提供DHCP服务的, 里面的DNS被网管写成了`114.114.114.114`, 改为PPPoE路由器的地址`192.168.0.1`即得以解决. 

### 1.2 Linux路由器的对策

解决方案可以是在`/etc/dnsmasq.d/`里加一个`fzkd.conf`, 里面把需要用本地DNS解析的网站白名单写好. 

例如:

    server=/baidu.com/192.168.0.1
    server=/www.baidu.com/192.168.0.1
    server=/sohu.com/192.168.0.1
    server=/youku.com/192.168.0.1
    server=/renren.com/192.168.0.1
    server=/163.com/192.168.0.1
    server=/126.com/192.168.0.1
    server=/qq.com/192.168.0.1
    server=/sina.com/192.168.0.1
    server=/56.com/192.168.0.1
    
最后, 不要忘记在/etc/dnsmasq.conf里加入一行:

    conf-dir=/etc/dnsmasq.d


另外, 大厦只允许安装方正宽带实在是SB, 不能因为办公室南边一条街就是方正大厦, 物业就被买通成这样嘛. 

配置好之后, 之前升级 OS X Yosemite 需要2天, 一下子就变成1小时了. 

## 2 WiFi 调优

DNS问题解决之后, 发现WiFi到本地网关`192.168.0.1`的链路太差, 最差可以到1000ms. 

由于办公大厦其它公司的WiFi非常多, 基本上在一个位置可以搜到十几个SSID. 

因此直接就联想到是同频干扰的问题. 当然, 解决这个问题目前来说, 最好的是直接换用5GHz的路由器. 

现有的842N只支持2.4GHz, 于是尝试通过修改信道的方式来优化. 

进到设置页面里发现, 出厂默认设置是"自动选择信道". 

![](http://upload.wikimedia.org/wikipedia/commons/8/8c/2.4_GHz_Wi-Fi_channels_%28802.11b%2Cg_WLAN%29.svg)

由于在[802.11g](http://en.wikipedia.org/wiki/IEEE_802.11g-2003)标准中, WiFi频谱一共被划分为13个信道, 其中12和13信道在中国不可合法使用(在部分国家地区如日本是合法的). 于是, 国内路由器厂商可用的是1-11信道. 而由于802.11g(在这里只以g来举例, 802.11n的同理)频谱占用是22MHz, 这11个信道的频谱并不是全部独立无干扰的, 看上面这张图就知道, 1-6-11信道或2-7-12等等组合是完整互不干扰的. 于是, 无论是运营商如中国移动, 或TP-Link等厂商, 都会经常用这1-6-11三个信道的组合, 来防止自家部署的AP(Access Point)互相干扰. 


但是! 朋友们呐! 仅仅自家AP的不干扰是不够的啊, 随便一个办公场所就有好多家同时在部署AP. 

另外我猜测, TP-Link的自动选择信道的算法一定是这样的:

    有人用1信道?  - 有
    有人用2信道?  - 有
    有人用3信道?  - 有
    有人用4信道?  - 有
    有人用5信道?  - 有
    有人用6信道?  - 有
    有人用7信道?  - 有
    有人用8信道?  - 有
    有人用9信道?  - 有
    有人用10信道? - 有
    有人用11信道? - 有
    好吧呀, 那咱用11信道吧! 

估测大部分TP-Link路由器都自动选成了11信道. 

### 2.1 解

解决方案非常简单直接, 我把一个AP配置成4信道, 另一个配置成了9信道. 完美解决. 

实测把电脑放在842N旁边, 20厘米, ping网关最差可达1000ms, 手动选成9信道后降为~2ms. 估计楼内其它热点都在用自动选择信道功能, 它只会在1 6 11 这三个号称"互相不干扰"的信道之间选来选去. 

补充一下, 如果用无线网卡查看周围的频谱情况, 可以使用[inSSIDer](http://www.inssider.com)工具来查看. 

