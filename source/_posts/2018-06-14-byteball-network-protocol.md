---
title: ByteBall网络节点通信协议详解
date: 2018-06-14 22:39:55
categories: 谈点区块链
tags:
  - dag
  - blockchain
  - byteball
  - bitcoin
---

## ByteBall网络节点通信协议详解

P2P网络是区块链网络的基础，网络中各个节点通过相互交换消息实现各种功能，包括收发交易、数据同步等操作。本文将对ByteBall网络节点的通信接口进行详细分析。

ByteBall网络节点之间采用websocket连接，采用json格式消息进行通信，消息可表示为`{type: type, content: content}`。ByteBall中的消息类型主要包括三类，即消息的type包括三种`request`、`response`以及`justsaying`，当然也可以自定义其它类型的消息。下面对消息的具体格式和处理流程进行解析。

### request：请求消息

请求消息的格式为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: command,
        params: params
    }
}
```

其中，`tag`为request消息的BASE64哈希值。

发送请求消息的函数原型为：

```javascript
function sendRequest(ws, command, params, bReroutable, responseHandler)
// ws: 发送请求的websocket连接
// command: 请求命令
// params: 请求参数
// bReroutable: 是否重新路由
// responseHandler: 请求响应处理回调函数
```

通过`sendRequest()`发出的每个请求会生成一个`tag`作为该请求的标识，并在内存中保留该请求的相关处理信息：

```javascript
ws.assocPendingRequests[tag] = {
    request: request,
    responseHandlers: [responseHandler],
    reroute: reroute,
    reroute_timer: reroute_timer,
    cancel_timer: cancel_timer
}
```

当请求发出后，若对方没有回应时有两种选择，一种是采用`reroute`，另一种是`cancel`。`reroute`的`timeout`为5s，`cancel`的`timeout`时间为300秒。

当接收到响应后（位于函数`handleResponse()`），在调用相应回调函数`responseHandlers`后，请求信息`ws.assocPendingRequests[tag]`被删除。

如果相关的连接被关闭，则清理该连接上的所有请求信息（位于函数`cancelRequestOnClosedConnection()`）。

### response：响应消息

响应消息的格式为：

```json
{
    type: 'response',
    content: {
        tag: tag,
        response: response
    }
}
```

其中，`tag`对应于请求消息中的`tag`。

发送响应消息的函数原型为：

```javascript
function sendResponse(ws, tag, response)
// ws: 发送响应的websocket连接
// tag: 发送响应对应的tag
// response: 响应消息内容
```

当收到对端节点请求后，设置`ws.assocInPreparingResponse[tag]=true`，并在回复响应后删除。

### justsaying：其它消息

其它消息包括的内容比较多，比如版本、心跳、hub登录等，其格式为：

```json
{
    type: 'justsaying',
    content: {
        subject: subject,
        body: body
    }
}
```

发送其它消息的函数原型为：

```javascript
function sendJustsaying(ws, subject, body)
// ws: 发送消息的websocket连接
// subject: 消息主题
// body: 消息内容
```

### 常见通信接口

#### 获取见证人列表（get_witnesses）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_witnesses'
    }
}
```

收到请求后，从数据库中读取当前的见证人列表并返回

```javascript
myWitnesses.readMyWitnesses(function(arrWitnesses){
    sendResponse(ws, tag, arrWitnesses);
}, 'wait');
```

比如：

```javascript
[ 'BVVJ2K7ENPZZ3VYZFWQWK7ISPCATFIW3',
  'DJMMI5JYA5BWQYSXDPRZJVLW3UGL3GJS',
  'FOPUBEUPBC6YLIQDLKL6EW775BMV7YOH',
  'GFK3RDAPQLLNCMQEVGGD2KCPZTLSG3HN',
  'H5EZTQE7ABFH27AUDTQFMZIALANK6RBG',
  'I2ADHGP4HL6J37NQAD73J7E5SKFIXJOT',
  'JEDZYC2HMGDBIDQKG3XSTXUSHMCBK725',
  'JPQKPRI5FMTQRJF4ZZMYZYDQVRD55OTC',
  'OYW2XTDKSNKGSEZ27LMGNOPJSYIXHBHC',
  'S7N5FE42F6ONPNDQLCF64E2MGFYKQR2I',
  'TKT4UESIKTTRALRRLWS4SENSTJX6ODCW',
  'UENJPVZ7HVHM6QGVGT6MWOJGGRTUTJXQ' ]
```

`network.js`中在函数`initWitenessesIfNecessary()`中使用。

#### 获取节点列表（get_peers）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_peers'
    }
}
```

返回对端节点已连接的节点列表

```javascript
sendResponse(ws, tag, arrPeerUrls)
```

比如:

```javascript
[ 'wss://byteball.fr/bb',
  'wss://byteball-hub.com/bb',
  'wss://hub.byteball.ee' ]
```

`network.js`中可以使用`requestPeers()`函数发出请求。

#### 获取交易数据（get_joint）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_joint',
        params: unit
    }
}
```

若找到，则返回响应交易数据；否则，返回交易未找到。

```javascript
storage.readJoint(db, unit, {
    ifFound: function(objJoint){
        sendJoint(ws, objJoint, tag);
    },
    ifNotFound: function(){
        sendResponse(ws, tag, {joint_not_found: unit});
    }
});
```

比如:

```javascript
{ joint:
   { unit:
      { unit: 'OcrOftkwCwTYAaGq0zV8FecAK/CKED++Ddewh2c2M60=',
        version: '1.0',
        alt: '1',
        witness_list_unit: 'oj8yEksX9Ubq7lLc+p6F2uyHUuynugeVq4+ikT67X6E=',
        last_ball_unit: 'p7obFdGHM5pTWPytNeRIjmamBe7485aDTzuddWI9yWY=',
        last_ball: 'wmiP2BvT7FFFtwy5qB+UsA4j8I5KbmBVA6mFCgwQUas=',
        headers_commission: 344,
        payload_commission: 157,
        main_chain_index: 2791799,
        timestamp: 1528796161,
        parent_units: [Array],
        authors: [Array],
        messages: [Array] } } }
```

`network.js`中可以使用`requestJoints()`函数发出请求。

#### 发送交易数据（post_joint）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'post_joint',
        params: joint
    }
}
```

接收到请求后，对`joint`数据进行检查，若通过，则返回`accepted`；否则，返回相应错误。

```javascript
handlePostedJoint(ws, objJoint, function(error){
    error ? sendErrorResponse(ws, tag, error) : sendResponse(ws, tag, 'accepted');
});
```

使用方法可参考`network.js`中的`postJointToLightVendor()`函数。

#### 发送心跳数据（heartbeat）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'heartbeat'
    }
}
```

使用方法可参考`network.js`中的`heartbeat()`函数。

ByteBall网络中节点之间依靠心跳请求维持连接，当超过`HEARTBEAT_RESPONSE_TIMEOUT=60s`没有收到对端节点的消息时，将会关闭与该节点的连接。ByteBall中默认3-4s向对端节点发送心跳请求。

#### 订阅交易数据（subscribe）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'subscribe',
        params: {
            subscription_id: subscription_id, // 订阅编号
            last_mci: last_mci                // 订阅起始mci
        }
    }
}
```

接收到请求后，若`last_mci>0`，则返回从该mci起始的joints；否则，返回当前网络中的叶子交易。

```javascript
if (ValidationUtils.isNonnegativeInteger(params.last_mci))
    sendJointsSinceMci(ws, params.last_mci);
else
    sendFreeJoints(ws);
```

订阅后，当收到新交易时，会向已订阅的节点进行转发（`forwardJoint(ws, objJoint)`）。使用方法可参考`network.js`中的`subscribe()`函数。

#### 同步交易数据（catchup）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'catchup',
        params: {
            witnesses: witnesses,                 // 见证人列表
            last_stable_mci: last_stable_mci,     // 稳定mci
            last_known_mci: last_known_mci        // 已知mci
        }
    }
}
```

必须在订阅状态下使用，接收到请求后，返回主链序号从`last_known_mci`至`last_stable_mci`之间的交易数据及其相关信息`objCatchupChain`。

```javascript
catchup.prepareCatchupChain(catchupRequest, {
    ifError: function(error){
        sendErrorResponse(ws, tag, error);
        unlock();
    },
    ifOk: function(objCatchupChain){
        sendResponse(ws, tag, objCatchupChain);
        unlock();
    }
});
```

`objCatchupChain`包括以下几部分信息：

```javascript
var objCatchupChain = {
    unstable_mc_joints: [], // 候选主链上未稳定的单元列表
    stable_last_ball_joints: [], // 已稳定的单元列表
    witness_change_and_definition_joints: [] // 见证人或地址定义发生变化的单元列表
};
```

使用方法可参考`network.js`中的`requestCatchup()`函数。

#### 获取哈希树（get_hash_tree）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_hash_tree',
        params: {
            from_ball: from_ball,
            to_ball: to_bal
        }
    }
}
```

必须在订阅状态下使用，接收到请求后，返回从`from_ball`至`to_ball`主链序号之间的所有ball交易数据。使用方法可参考`network.js`中的`requestNextHashTree()`函数。

#### 获取主链序号（get_last_mci）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_last_mci'
    }
}
```

返回目前最大的主链序号。

#### 通过Hub向设备发送消息（hub/deliver）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/deliver',
        params: {
            to: device_address,
            pubkey: pubkey,
            signature: signature,
            encrypted_package: encrypted_message
        }
    }
}
```

通过Hub向其它设备发送加密消息，使用方法可以参考`device.js`中的`sendPreparedMessageToConnectedHub()`函数。

#### 从Hub获取配对设备临时公钥（hub/get_temp_pubkey）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_temp_pubkey',
        params: permanent_pubkey
    }
}
```

通过永久公钥`permanent_pubkey`从Hub处获取对端设备的临时公钥。

#### 设备向Hub更新临时公钥（hub/temp_pubkey）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/temp_pubkey',
        params: {
            temp_pubkey: temp_pubkey,
            pubkey: permanent_pubkey,
            signature: signature
        }
    }
}
```

向Hub更新本设备的临时公钥，设备必须登陆到Hub上。接收到请求后，对参数及签名进行验证，如果成功则返回响应消息`updated`。

#### 开启通知（hub/enable_notification）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/enable_notification',
        params: registrationId
    }
}
```

设备开启推送通知，目前还未使用。

#### 关闭通知（hub/disable_notification）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/disable_notification',
        params: registrationId
    }
}
```

设备关闭推送通知，目前还未使用。

#### 获取机器人列表（hub/get_bots）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_bots'
    }
}
```

返回Hub节点上的机器人列表，比如：

```json
[...
 { id: 27,
    name: 'Worldopoly ICO',
    pairing_code: 'AwyoKVsyxajATgLXa9Jhh8NBRTnUaZNHdi85c43g+GoJ@byteball.org/bb#0000',
    description: 'Worldopoly is the world’s first mobile game combining AR, AI, Geolocationing, Blockchain, and DAG. The ICO is active until 17 May 2018, and you can buy WPT tokens with Bytes, BTC, or Ether.  WPT token is issued both on Byteball and Ethereum platforms but investors on Byteball platform receive increased bonus (even if they pay in ETH or BTC) for investments up to 30 ETH.\n\nWebsite: https://worldopoly.io' },
  { id: 28,
    name: 'WhiteLittle Airdrop 小白币糖果机器人',
    pairing_code: 'Ahe4jkq5GvgLQ2h5ftqRMjWBBumUEN96tWoSfEQ9TGHF@byteball.org/bb#0000',
    description: '小白链机器人送出小白币糖果。小白链是专门为想进入区块链行业的小白们量身打造的帮助平台，其目的是建立一个基于字节雪球技术为小白们提供有效帮助的信息发现生态平台。小白链的设计初衷是构建一套合理的激励机制，能够及时得到帮助，又让提供帮助的区块链从业者得到合理的回报。\n\n开发者：123cb.net' },
  { id: 32,
    name: 'Exchange bot for dual-chain tokens',
    pairing_code: 'A+dAU2j/Tm9lnlmc2SryltsfVzOq9GLPxccAq+dClCxr@byteball.org/bb#f7b42a61a3ba6a34cbeeb18d37979927ad1103fc',
    description: 'For tokens issued both on Byteball and Ethereum platforms, the bot enables seamless exchange between Byteball and Ethereum tokens.\n\nDeveloper: HDRProtocol, https://github.com/HDRProtocol/exchanger' },
 ...]
```

#### 获取资产元数据（hub/get_asset_metadata）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_asset_metadata',
        params: asset
    }
}
```

从Hub节点上获取已发布资产的信息，比如：

```json
{ metadata_unit: 'Rg3DNkDJJ2DWfIzTj3Ypz8CBGi617wl07QkJq7Z5soc=',
  registry_address: 'AM6GTUKENBYA54FYDAKX2VLENFZIMXWG',
  suffix: null }
```

具体使用方法可以参考`wallet.js`中的`fetchAssetMetadata()`函数。

#### 从Hub获取交易历史（light/get_history）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_asset_metadata',
        params: {
            witnesses: witnesses,
            requested_joints: joints,
            addresses: addresses
        }
    }
}
```

请求特定交易或地址的历史数据，仅适用于轻节点，具体使用方法可以参考`network.js`中的`requestHistoryFor()`函数。轻钱包可以根据获取的历史数据构建证据链，从而验证交易的可靠性。

对于轻钱包而言，它本身不保存所有的交易数据，而需要采用`requestFromLightVendor()`从`Vendor`处获取数据，目前`Vendor`为`Hub`节点。

#### 获取连接证明（light/get_link_proofs）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_link_proofs',
        params: units
    }
}
```

获取特定交易的连接证明，仅适用于轻节点，具体使用方法可以参考`network.js`中的`checkThatEachChainElementIncludesThePrevious()`。

#### 获取父单元及见证单元（light/get_parents_and_last_ball_and_witness_list_unit）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_parents_and_last_ball_and_witness_list_unit',
        params: {
            witnesses: witnesses
        }
    }
}
```

轻节点获取这些信息后，可用于构造自己的交易数据。返回的响应数据，比如：

```json
{ parent_units: [ 'ovIwMvA9MMKgxiHrslJQhQGzUAIXlRs47grVdG/er3s=' ],
  last_stable_mc_ball: 'osJhTYZv+HS5hGhH01A/3PKpyaxPRbtcxQEaUj/a/h4=',
  last_stable_mc_ball_unit: '+9fxcmxM90mhIxvtJ3yo++tAoYofyoDmPqBQEwbEHDA=',
  last_stable_mc_ball_mci: 2805208,
  witness_list_unit: 'oj8yEksX9Ubq7lLc+p6F2uyHUuynugeVq4+ikT67X6E=' }
```

具体使用方法可以参考`composer.js`中的`composeJoint()`函数。

#### 查询用户认证信息（light/get_attestation）

请求消息为：

```json
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_attestation',
        params: {
        	attestor_address: attestor_address,
        	field: field,
        	value: value
    	}
    }
}
```

接收到请求后，如果Hub查询到相应的认证信息，返回认证信息所在的交易单元。

#### 发送版本信息（version）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'version',
        body: {
            protocol_version: protocol_version,
            alt: alt,
            library: name,
            library_version: version,
            program: program,
            program_version: program_version
        }
    }
}
```

具体用法可以参考`network.js`中的`sendVersion()`函数。

#### 已发送完所有交易（free_joints_end）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'free_joints_end',
        body: null
    }
}
```

用于通知对方所有的叶子交易已发送完毕，具体用法可参考`network.js`中的`sendFreeJoints()`及`sendJointsSinceMci()`

#### 发送隐私交易（private_payment）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'private_payment',
        body: privateElement
    }
}
```

用来向对方发送隐私交易的证据链，但该条`justsaying`消息并没有使用。实际上隐私资产是在两个设备之间通过加密消息进行点对点发送的，相关代码位于`wallet_general.js`的`sendPrivatePayments()`中，发送加密消息采用的是`device.js`中的`sendMessageToDevice()`（底层使用的是`hub/deliver`接口）。

#### 登录Hub（hub/login）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'hub/login',
        body: {
            challenge: challenge,
            pubkey: pubkey,
			signature: signature
        }
    }
}
```

该消息用于设备登录Hub。

#### 获取设备新消息（hub/refresh）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'hub/refresh',
        body: null
    }
}
```

用于从Hub上获取该设备还未接收的消息。

#### 发送配对消息（hub/challenge）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'hub/challenge',
        body: challenge
    }
}
```

由Hub发送给设备，用于设备登录。

#### 发送设备消息（hub/message）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'hub/message',
        body: {
            message_hash: message_hash,
            message: message
        }
    }
}
```

由Hub转发给设备的消息，可配合`hub/deliver`使用，具体用法可参考`sendStoredDeviceMessages()`。

#### 轻钱包交易更新消息（light/have_updates）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'light/have_updates',
        body: null
    }
}
```

当轻钱包使用`light/get_history`从Hub上请求交易历史时，Hub会在`watched_light_addresses`中记录下请求的地址列表或者`watched_light_units`中记录下请求的交易列表。当相关的交易达到稳定时，Hub将通过`light/have_updates`消息通知轻钱包。然后，轻钱包可以通过`light/get_history`确定已达到稳定的交易单元。

#### 添加轻钱包监视地址（light/new_address_to_watch）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'light/new_address_to_watch',
        body: address
    }
}
```

轻钱包向Hub请求将地址加入`watch_light_addresses`表中，从而可以从Hub接收相应地址的交易信息，具体用法可参考`network.js`中的`addLightWatchedAddress()`函数。

#### 交易价格消息（exchange_rates）

消息格式为：

```javascript
{
    type: 'justsaying',
    content: {
        subject: 'exchange_rates',
        body: exchangeRates
    }
}
```

由Hub广播的交易价格消息，比如：

```json
[ 'justsaying',
  { subject: 'exchange_rates',
    body: { GBYTE_USD: 119.1914071839, GBB_USD: 3.6949336227009 } } ]
```



### 通信接口列表

| type       | content                                                      | function                 | 功能      |
| ---------- | ------------------------------------------------------------ | ------------------------ | ------------ |
| request    | {tag: tag, command: command, params: params}                 | sendRequest()            | 通用发送请求 |
|            | {tag: tag, command: 'get_peers'}                             | requestPeers()           | 请求节点列表 |
|            | {tag: tag, command: 'get_witnesses'}             | initWitenessesIfNecessary()          | 请求见证人列表 |
|            | {tag: tag, command: 'get_joint', params: unit}             | requestJoints()          | 请求交易信息 |
|            | {tag: tag, command: 'post_joint', params: joint}             | postJointToLightVendor()          | 发送交易信息 |
|            | {tag: tag, command: 'heartbeat'}                             | heartbeat()              | 心跳请求     |
|            | {tag: tag, command: 'subscribe', params: {subscription_id, last_mci}} | subscribe()              | 订阅交易     |
|            | {tag: tag, command: 'catchup', params: {witnesses, last_stable_mci, last_known_mci}}             | requestCatchup()          | 同步数据 |
|            | {tag: tag, command: 'get_hash_tree', params: {from_ball,to_ball}}             | requestNextHashTree()          | 请求哈希树 |
|            | {tag: tag, command: 'get_last_mci'}                             |               | 获取主链序号     |
|            | {tag: tag, command: 'hub/deliver', params: {encrypted_package, to, pubkey, signature}} |               | 通过Hub向设备发送消息   |
|            | {tag: tag, command: 'hub/get_temp_pubkey', params: pubkey} |               | 从Hub获取配对设备临时公钥     |
|            | {tag: tag, command: 'hub/temp_pubkey', params: {temp_pubkey, pubkey, signature}} |               | 设备向Hub更新临时公钥     |
|            | {tag: tag, command: 'hub/enable_notification'} |               | 开启通知 |
|            | {tag: tag, command: 'hub/disable_notification'} |               | 关闭通知 |
|            | {tag: tag, command: 'hub/get_bots'} |               | 获取机器人列表 |
|            | {tag: tag, command: 'hub/get_asset_metadata', params: asset} | fetchAssetMetadata() | 获取资产元数据 |
|            | {tag: tag, command: 'light/get_history', params: {witnesses, requested_joints, addresses}} | requestHistoryFor() | 从Hub获取交易历史     |
|            | {tag: tag, command: 'light/get_link_proofs', params: units |               | 获取连接证明     |
|            | {tag: tag, command: 'light/get_parents_and_last_ball_and_witness_list_unit', params: {witnesses}} | composeJoint()              | 获取父单元及见证单元     |
|            | {tag: tag, command: 'light/get_attestation', params: { attestor_address, field, value}} |               | 查询用户认证信息 |
| response   | {tag: tag, response: response}                               | sendResponse()           | 通用发送响应 |
|            | {tag: tag, response: {error: error}}                         | sendErrorResponse()      | 发送响应错误 |
| justsaying | {subject: subject, body: body}                               | sendJustsaying()         | 发送其它消息 |
|            | {subject: 'error', body: error}                              | sendError()              | 发送错误消息 |
|            | {subject: 'info', body: content}                             | sendInfo()               | 发送通知消息 |
|            | {subject: 'result', body: content}                           | sendResult()             | 发送结果消息 |
|            | {subject: 'result', body: {unit, result: 'error', error}} | sendErrorResult()        | 发送错误结果 |
|            | {subject: 'version', body: {protocol_version, alt, library, library_version, program, program_version}} | sendVersion()            | 发送版本信息 |
|            | {subject: 'new_version', body: {version}} |             | 发现新版本 |
|            | {subject: ' hub/push_project_number', body: { projectNumber}} |             | 推送API编号 |
|            | {subject: 'bugreport', body: {message, exception}} |             | 报告bug |
|            | {subject: 'free_joints_end', body: null} | sendFreeJoints() | 已发送完所有交易 |
|            | {subject: 'private_payment', body: privateElements} |             | 发送隐私交易 |
|            | {subject: 'my_url', body: url} |             | 发送连接地址 |
|            | {subject: 'want_echo', body: echo_string} |             | 请求对方回应 |
|            | {subject: 'your_echo', body: echo_string} |             | 发送回应信息 |
|            | {subject: 'hub/login', body: {challenge, pubkey, signature}} |             | 登录Hub |
|            | {subject: 'hub/refresh', body: null} |  | 获取设备新消息 |
|            | {subject: 'hub/delete', body: message_hash} |  | 删除设备消息 |
|            | {subject: 'hub/challenge', body: challenge} |  | 发送配对消息 |
|            | {subject: 'hub/message', body: {message_hash, message}} | sendStoredDeviceMessages() | 发送设备消息 |
|            | {subject: 'hub/message_box_status', body: 'has_more'/'empty'} | sendStoredDeviceMessages() | 设备消息状态 |
|            | {subject: 'light/have_updates', body: null} |  | 轻钱包交易更新消息 |
|            | {subject: 'light/new_address_to_watch', body: address}       | addLightWatchedAddress() | 添加轻钱包监视地址 |
|            | {subject: 'exchange_rates', body: exchangeRates} |  | 交易价格消息 |
|            | {subject: 'upgrade_required', body: null} |  | 强制升级消息 |









