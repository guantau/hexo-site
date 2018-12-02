---
title: 再论ByteBall钱包
date: 2018-06-26 12:42:55
categories: 谈点区块链
tags:
  - dag
  - blockchain
  - byteball
  - bitcoin
---


ByteBall的钱包类型分为两种：

1. 单设备钱包：该类型钱包仅由单一设备控制，它与特定设备地址是绑定的，钱包中的地址为单签名地址；
2. 多设备钱包：该类型钱包由多个设备共同控制，它与若干个设备地址绑定，钱包中的地址为多签名地址。

在之前的文章[《ByteBall钱包详解》](https://bbfans.org/2018/06/17/byteball%E9%92%B1%E5%8C%85%E8%AF%A6%E8%A7%A3/)中，我们提到，ByteBall的地址分为三种：

1. 普通地址：地址定义中不包含其它地址，采用BASE32编码，长度为32，比如`A2WWHN7755YZVMXCBLMFWRSLKSZJN3FU`；
2. 共享地址：地址定义中包含了其它地址，地址格式与普通地址相同，通常用作智能合约地址；
3. 设备地址：生成方法及地址格式类似普通地址，但在设备地址在头部添加了一个0，长度为33，比如`05FV4WNIEU4OHIAIF7XEIRC2QRRLFPAC3`。

因此，ByteBall的钱包与地址可以总结为下面这张图：

![byteball-wallet](http://oc7urqs4c.bkt.clouddn.com/2018-06-25-144238.png)

其中：设备具有唯一的设备地址；普通地址包括单签名地址和多签名地址，单设备钱包生成单签名地址，多设备钱包生成多签名地址；多个普通地址可以共同构成共享地址。

## 单签名地址

单设备钱包由单一设备生成，假设设备地址为`DEVICE_ADDRESS`，单设备钱包中单签名地址定义的模板为：

```json
["sig", {pubkey: '$pubkey@DEVICE_ADDRESS'}]
```

具体在生成地址时，`$pubkey@DEVICE_ADDRESS`会替换成相应的公钥。

## 多签名地址

多设备钱包由多个设备共同控制。假设3个设备的地址分别为`DEVICE_A_ADDRESS`、`DEVICE_B_ADDRESS`以及`DEVICE_C_ADDRESS`。我们需要生成一个`2-3`的多设备钱包，即3个设备中至少需要2个设备签名才可以生效，则相应的多设备钱包中多签名地址定义的模板为：

```json
["r of set", {
    required: 2,
    set: [
        ["sig", {pubkey: '$pubkey@DEVICE_A_ADDRESS'}],
        ["sig", {pubkey: '$pubkey@DEVICE_B_ADDRESS'}],
        ["sig", {pubkey: '$pubkey@DEVICE_C_ADDRESS'}],
    ]
}]
```

在创建多设备钱包时，设备之间会通过加密消息相互交换`xPubKey`。这样，不同的设备可以依据相同的地址路径生成相同的地址。

## 共享地址

共享地址本质上可以认为是智能合约的地址，例如`flight delay insurance`的地址定义（或者智能合约）示例为：

```json
["or",[
    ["and",[
       ["seen",{"what":"output","address":"this address","asset":"base","amount":22664}],
       ["or",[
           ["and",[
               ["address","TTD2AVY4W2VH62NJXIP7R67XBHWZRQRJ"],
               ["in data feed",[["GFK3RDAPQLLNCMQEVGGD2KCPZTLSG3HN"],"MU5152-2018-06-18",">","60",2810590]]]],
           ["and",[
               ["address","4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ"],
               ["in data feed",[["I2ADHGP4HL6J37NQAD73J7E5SKFIXJOT"],"timestamp",">",1529442000000]]]],
           ["and",[
               ["address","4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ"],
               ["in data feed",[["GFK3RDAPQLLNCMQEVGGD2KCPZTLSG3HN"],"MU5152-2018-06-18","<=","60",2810590]]]]]]]],
    ["and",[
        ["address","4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ"],
        ["not",["seen",{"what":"output","address":"this address","asset":"base","amount":22664}]],
        ["in data feed",[["I2ADHGP4HL6J37NQAD73J7E5SKFIXJOT"],"timestamp",">",1529034037936]]]]]
```

上述地址定义中共涉及到两个地址`TTD2AVY4W2VH62NJXIP7R67XBHWZRQRJ`及`4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ`，它是这两个地址的共享地址。

通过上述地址定义，可以得到该共享地址为`WDCIIWRDHSNNE2DQZ7YVU53USELZBLGV`。其中，地址的签名路径包括：

```json
{
    r.0.1.0.0: "TTD2AVY4W2VH62NJXIP7R67XBHWZRQRJ",
    r.0.1.1.0: "4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ"
    r.0.1.2.0: "4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ"
    r.1.0: "4JZOKE43GALLZA4P63NXT7NYAJLSMNYZ"
}
```

只有满足合约中相应的条件，且具有相应路径的签名，才可以对共享地址中的资产进行操作。

