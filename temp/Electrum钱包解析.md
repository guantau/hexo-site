# Electrum钱包代码解析

## SimpleConfig

用于处理跟配置相关的所有操作，配置主要包括三个来源：

1. 命令行参数，优先级最高，可以在初始化对象时直接输入参数
2. 用户配置文件，优先级次高，默认为Electrum工作目录下的`config`文件
3. 系统配置文件（/etc），优先级最低，默认为`/etc/electrum.conf`

优先级是通过SimpleConfig中的`get`函数来实现，它会依次读取上述三个来源的数据，从而获取对应的配置值。

SimpleConfig的初始化函数为

```python
def __init__(self, options={}, read_system_config_function=None,
             read_user_config_function=None, read_user_dir_function=None):
```

其中，`options`为命令行参数，`read_system_config_function`为读取系统配置函数，`read_user_config_function`为读取用户配置函数，`read_user_dir_function`为获取Electrum工作目录函数

### Electrum工作目录

Electrum的默认工作目录可以通过命令行参数或者系统配置文件中的`electrum_path`进行设置，或者通过`read_user_dir_function`来获取，否则采用默认目录：

- Android: 外部设备的electrum目录
- Linux或Mac: 用户主目录下的.electrum目录
- 其它：环境变量`APPDATA`或`LOCALAPPDATA`下的Electrum目录

### Electrum钱包路径

Electrum钱包路径可以通过命令行参数`wallet_path`或者配置文件中的`default_wallet_path`进行设置，默认钱包路径为工作目录下的wallets目录，钱包文件为`default_wallet`

### 主要配置参数

`server`



## BlockChain

用于管理区块链头部数据，以及验证区块头部数据是否正确，代表的是本地区块链数据（账本）。

其初始化函数为：

```python
def __init__(self, config, checkpoint, parent_id):
```

其中，`config`为配置文件对象，`checkpoint`为分叉高度，`parent_id`为父链的起始高度。分叉链头部数据在Electrum工作目录下的`forks`目录存储，命名规则为`fork_{parent_id}_{checkpoint}`

采用变量`blockchains`存储所有的区块链及其分叉，`blockchains[0]`代表最原始的区块链，对于分叉链，`blockchains[parent_id]`代表其父链。

> 似乎，Electrum没有考虑，在同一高度分叉出多个链的情形



## Wallet

用于存储用户私钥及地址，方便用户对自己的资产进行管理与监控。



## Interface

interface通过socket与远程electrum服务器连接

interface有几种工作模式：

1. default：默认工作模式，监听新区块，新区块到达后对区块链进行更新
2. backward
3. binary
4. catch_up

当interface上收到新的区块时，检查该区块是否有效

## Network

典型的socket程序，electrum维护多个socket，并通过这些socket进行读写数据

从程序实现的角度来讲，Network对应的是一个Daemon Thread，它又包含的多个Interface，

工作流程：





Network中可能存在多个blockchain，区块链头部数据存储在Electrum工作目录下的`blockchain_headers`文件中（主链）以及`forks`目录下的文件中（分叉链）。

Network中维护了`interfaces`用于存储已连接或正在连接的服务器，`interface`是Network采用的主服务器。Network默认连接`num_server=10`个服务器，`server.json`中列出了候选的服务器地址。

这实际上是本地数据与多个远程服务器数据之间的同步问题，当不同的远程服务器上的数据发生更新时，本地进行数据更新，以期与远程服务器数据保持一致。

主`interface`与其它`interfaces`之间的关系？

当任何一个`interface`上有数据更新时，本地的区块链都会更新。

有三个过程需要关注：

1. 如何发送请求
2. 如何接收响应
3. 如何使用回调函数





### 接收响应

默认处理一些响应

```python
        if method == 'server.version':
            interface.server_version = result
        elif method == 'blockchain.headers.subscribe':
            if error is None:
                self.on_notify_header(interface, result)
        elif method == 'server.peers.subscribe':
            if error is None:
                self.irc_servers = parse_servers(result)
                self.notify('servers')
        elif method == 'server.banner':
            if error is None:
                self.banner = result
                self.notify('banner')
        elif method == 'server.donation_address':
            if error is None:
                self.donation_address = result
        elif method == 'blockchain.estimatefee':
            if error is None and result > 0:
                i = params[0]
                fee = int(result*COIN)
                self.config.update_fee_estimates(i, fee)
                self.print_error("fee_estimates[%d]" % i, fee)
                self.notify('fee')
        elif method == 'blockchain.relayfee':
            if error is None:
                self.relay_fee = int(result * COIN)
                self.print_error("relayfee", self.relay_fee)
        elif method == 'blockchain.block.get_chunk':
            self.on_get_chunk(interface, response)
        elif method == 'blockchain.block.get_header':
            self.on_get_header(interface, response)
```



定义的event类型

* banner

* fee

* interfaces

* servers

* status

* updated

  ​