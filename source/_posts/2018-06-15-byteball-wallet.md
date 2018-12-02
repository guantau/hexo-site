---
title: ByteBall钱包详解
date: 2018-06-15 23:48:55
categories: 谈点区块链
tags:
  - dag
  - blockchain
  - byteball
  - bitcoin
---


## 钱包结构

顾名思义，钱包是用来保存钱的。但在数字货币的世界中，钱包里面并没有“钱”。钱包账户里有多少“钱”都是记录在区块链上的，钱包里只是存储了账户对应的**私钥**，账户是从私钥相应的公钥衍生出来的。只要有了私钥，你就可以在数字货币世界里证明你的身份，发送区块链上属于你的资产。因此，**钱包实际上是管理和存储私钥的工具**。

ByteBall钱包结构与Bitcoin类似，Bitcoin对管理和存储私钥以及通过私钥生成地址制定了一系列标准（BIP, Bitcoin Improvement Proposals），主要包括：

- [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)：定义了HD钱包（Hierarchical Deterministic Wallet），允许从单一种子产生一树状结构储存多组私钥和公钥，可以方便的备份、转移以及分层权限控制等。HD钱包包含以树状结构衍生的密钥，使得父密钥可以衍生一系列子密钥，每个子密钥又可以衍生出一系列孙密钥，以此类推，无限衍生。下图展示了HD钱包的树状结构：

  ![bip33-hd-wallet](http://oc7urqs4c.bkt.clouddn.com/2018-06-17-082216.png)

- [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)：将种子用方便记忆和书写的单词表示，一般由12个单词组成，称为助记词或助记码。由一系列单词生成种子是个标准化的方法，这样易于在钱包中转移、导出和导入。比如`shrimp make call path pink draw gym song select brother social base`，其相应的私钥为`xprv9s21ZrQH143K4HQ8sBPNNkXHNuVYfivR96gtpJzMWCJVF5hdYjzT5nVQu6GfHAQLPstQtFp9GTWADUcKgzaV5EyL5JxT2rxkyg51ReGTVg8`。

- [BIP43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)：为HD钱包路径引入`purpose`字段，路径记作`m / purpose' / *`，比如`m / 0' / *`表示BIP32的默认账户。

- [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)：定义HD钱包路径字段，赋予树状结构中的各层特殊的意义，从而支持多币种、多账户，路径定义为：

  `m / purpose' / coin_type' / account' / change / address_index`

  其中，`purpose'`固定是`44'`，代表使用BIP44；`coin_type'`用来表示不同币种，例如Bitcoin就是`0'`，Bitcoin Testnet是`1'`，Ethereum是`60'`；`account'`表示账户编码；`change`用来表示该地址是否为找零地址；`address_index`为地址编号。

- [BIP45](https://github.com/bitcoin/bips/blob/master/bip-0045.mediawiki)：定义多签名HD钱包路径字段，从而支持多签名地址，路径定义为：

  `m / purpose' / cosigner_index / change / address_index`

  其中，`purpose`固定是`45'`，代表使用BIP45；`cosigner_index`表示多签名用户编号；其余字段与BIP44相同。



ByteBall钱包采用的是符合BIP44标准的HD钱包，其采用的路径结构为：

```
m / 44' / 0' / account' / change / address_index
```

此外，除了使用`account`来标识账户外，还使用了该账户公钥的哈希值对账户进行唯一表示，表示方法为：

```javascript
var wallet = crypto.createHash("sha256").update(xPubKey, "utf8").digest("base64");
```

ByteBall中还有一类特殊的地址，即设备地址，其采用的路径结构为：

```
m / 1' / *
```



ByteBall钱包使用了`bitcore-lib`和`bitcore-mnemonic`两个库进行钱包管理，简单的使用方法如下：

```javascript
var Mnemonic = require('bitcore-mnemonic');
var Bitcore = require('bitcore-lib');

// 生成助记词
// 'fee decorate step culture autumn game social very lemon drum embrace much'
var mnemonic = new Mnemonic();

// 生成HD钱包私钥
// <HDPrivateKey: xprv9s21ZrQH143K2P4n8N1MYXW4NPfPuH7K3z4uYe6iRxHHXMEUJQRriizTdMuN6rh5XLH4okZm69c7THSN7M5Lk8WSFXDd6b9DBcb6nAdxw7n>
var xPrivKey = mnemonic.toHDPrivateKey();

// 生成HD钱包公钥，account=0
// <HDPublicKey: xpub6BqpEDyiVUcuzwzhn3jznUooytK5k9weeZqoM9vRpKbaHJGdyXZ3uk6HzpwYfCbxUuTsn41RgTpsTqA44uC4Ce2vRAZAcr9n5EyRtZxczbs>
var xPubKey = Bitcore.HDPublicKey(xPrivKey.derive("m/44'/0'/0'"));

// 生成设备私钥，用于产生设备地址
// <HDPrivateKey: xprv9u8BXQraRhyRypxQj4ffhEohzDyjLbqezoCuFCHpLHazDbi9ymLb1zp6aKmDpTpyxXw8Uc6HbCRSZMEtW71Eie1QPi2T3RweoKWrdymxGEf>
var devicePrivKey = xPrivKey.derive("m/1'");

```



## 地址生成

ByteBall中包括三类地址：

- 普通地址
- 共享地址
- 设备地址

其中，普通地址和共享地址都是通过地址定义脚本生成的，二者的区别类似于Bitcoin中的P2PKH地址和P2SH地址，具体细节可参考[《ByteBall原理解析（三）地址、脚本及合约》](https://bbfans.org/2018/05/21/byteball%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90%EF%BC%88%E4%B8%89%EF%BC%89%E5%9C%B0%E5%9D%80%E3%80%81%E8%84%9A%E6%9C%AC%E5%8F%8A%E5%90%88%E7%BA%A6/)；设备地址是采用设备公钥生成的，地址格式和校验规则与普通地址相同，只是前缀添加了`0`。

给定需要生成地址的对象，ByteBall生成地址时采用的是`ripemd160`哈希，并给地址加上校验，使用的是`byteballcore/object_hash.js`中的函数`getChash160()`函数。

具体来说，设备地址生成方法：

```javascript
var ecdsa = require('secp256k1');  // 椭圆曲线非对称加密

// 获取设备公钥
var pubkey = ecdsa.publicKeyCreate(devicePrivKey.privateKey.bn.toBuffer({size:32}), true).toString('base64');

// 生成设备地址
var objectHash = require('byteballcore/object_hash.js');
var device_address = '0' + objectHash.getChash160(pubkey);
```

普通地址生成方法：

```javascript
// change=0 address_index=0，生成公钥
var pubkey = xPubKey.derive('m/0/0').publicKey.toBuffer().toString('base64');

// 设定地址定义脚本，为单签名
var arrDefinition = ["sig", {"pubkey": pubkey}];

// 生成地址
var address = objectHash.getChash160(arrDefinition);
```

共享地址的生成方法与普通地址基本一致，唯一的差别在于地址定义脚本要更加复杂一些，具体细节可参考前面给出的那篇文章。

