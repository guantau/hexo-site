---
title: WiFi密码破解笔记
date: 2016-08-21 23:32:38
categories: 弄点工具
tags:
  - WiFi
---

WiFi常用的安全机制有三种：WEP、WPA、WPA2。

由于存在协议漏洞，WEP只要抓取到足够多的初始向量数据包（一般2w个），即可实现完全破解，这类的工具非常多。

WPA和WPA2则相对安全得多，目前破解的思路有两种：

<!-- more -->

## 1. 字典攻击：

获取客户端和路由器之间得握手协议包，采用字典攻击的方式暴力破解密码。

当采用一些简单的生日、单词、电话号码作为密码时，很容易实现破解，网上已有相关字典。若采用稍微复杂的密码，如密码采用8位大小写字母和数字混合，在运算速度为500K个/秒时，暴力破解需要时间14年（暴力破解时间可参考  [http://lastbit.com/pswcalc.asp][1] ）。

WPA/WPA2密码支持8-63位，如果设置较长较复杂的密码，字典攻击破解难度将很大。
字典攻击方法需要两步，一是抓取握手协议包，二是穷举字典破解。

常用的工具：
* 开源工具aircrack-ng（[http://www.aircrack-ng.org/][2]），包括了无线抓包和密码破解程序
* 抓包工具Omnipeek（[http://www.wildpackets.com/products/omnipeek_network_analyzer][3]）
* 国产软件无线网络分析软件科来（[http://www.colasoft.com.cn/download/capsa.php][4]）
* 密码计算软件ewsa（[http://cn.elcomsoft.com/ewsa.html][5]）优势是可以用GPU加速密码破解过程

网上有很多公开的字典库，直接搜索即可下载。还有一些字典生成器，如黑刀、万能钥匙、木头字典生成器等等。

## 2. WPS攻击：

当路由器开启了WPS功能，可采用WPS攻击，破解路由器PIN码，从而获得其密码。

采用字典攻击的方式无疑是一种效率很低的方法，如果没有估计到别人的密码模式，要破解是十分困难的。现在很多路由器开启了WPS，通过攻击路由器PIN码这种方式可以将破解效率大大提高。路由器PIN码只有8位，破解时可分为前后4位分别破解，只需要11000次的尝试即可实现破解（每次尝试可能需要数秒，视信号强度和路由器而定）。一旦已知PIN码，就可立即计算出WiFi密码。即使更换了WiFi密码，只要PIN码不变，也能立刻算出密码。

主要工具为：
* Reaver（WPS Cracker，[http://code.google.com/p/reaver-wps/][6]）

---------------
WiFi密码破解的方法就是这些，总的来说，要保证WiFi的足够安全，原则就是：关闭WEP，关闭WPS，使用WPA2，提高密码强度。

现在有一些系统把WiFi破解的工具进行了集成，并做了图形化界面方便使用，比如XiaoPanOS、CDLinux、Beini等。这些系统镜像都比较小，可以直接烧制在U盘上随身携带，以后到了其他地方，就不愁没有WiFi了。：）


  [1]: http://lastbit.com/pswcalc.asp
  [2]: http://www.aircrack-ng.org/
  [3]: http://www.wildpackets.com/products/omnipeek_network_analyzer
  [4]: http://www.colasoft.com.cn/download/capsa.php
  [5]: http://cn.elcomsoft.com/ewsa.html
  [6]: http://code.google.com/p/reaver-wps/
