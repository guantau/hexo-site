---
title: 在小米路由器mini上实现去广告
date: 2016-08-21 23:31:01
categories: 弄点工具
tags:
  - 小米路由器
  - 去广告
---

所谓去广告，指的是去掉网页上弹出的乱七八糟的广告，还有视频网站播放前的广告。常见的几款去广告软件包括：
1. ADSafe：目前支持Windows和Android，去除效果还不错。
2. AD Muncher（奶牛）：老牌的广告去除软件，原来收费，现在已免费，仅支持Windows。
3. bloxy（保护伞）：仅支持Windows。
4. adbyby：目前支持平台最全的，包括Windows、Linux、Mac、Openwrt。
5. ADMon：目前支持Windows平台。
6. Adblock：浏览器插件，也就是只能在浏览器上使用。

对于非插件的去广告软件来讲，其原理是用带过滤功能的开源代理服务器privoxy加上Adblock-Easylist China及Easylist的开源广告规则库来实现的，通过正则表达式和CSS进行过滤。

在路由器上部署是最理想的，这样所有的设备都能共享了，这么看来只有用adbyby了。

小米路由器mini开通ssh后就可以干活了，用scp把下载的adbyby传到/tmp目录下，修改配置文件adhook.ini中扩展规则为 https://easylist-downloads.adblockplus.org/easylistchina+easylist.txt 即可。

广告无踪影，生活顿时又美好了一点点！
