# 命令列表

## == Blockchain ==
### getbestblockhash

* 说明：获取最高区块哈希值
* 参数：无
* 返回值：
    * hex（string）区块链顶部区块哈希值
* 示例：

```shell
> bitcoin-cli getbestblockhash
```

### getblock "blockhash" ( verbosity )

* 说明：获取区块详细数据
* 参数：
    * blockhash（string，必选） 区块哈希值
    * verbosity（numeric，可选） 0-hex编码数据，1-json数据，2-json数据（解析交易数据）
* 返回值：
    * verbosity=0
        * data（string）hex编码数据
    * verbosity=1

    ```
    {
      "hash" : "hash",     (string) the block hash (same as provided)
      "confirmations" : n,   (numeric) The number of confirmations, or -1 if the block is not on the main chain
      "size" : n,            (numeric) The block size
      "strippedsize" : n,    (numeric) The block size excluding witness data
      "weight" : n           (numeric) The block weight as defined in BIP 141
      "height" : n,          (numeric) The block height or index
      "version" : n,         (numeric) The block version
      "versionHex" : "00000000", (string) The block version formatted in hexadecimal
      "merkleroot" : "xxxx", (string) The merkle root
      "tx" : [               (array of string) The transaction ids
         "transactionid"     (string) The transaction id
         ,...
      ],
      "time" : ttt,          (numeric) The block time in seconds since epoch (Jan 1 1970 GMT)
      "mediantime" : ttt,    (numeric) The median block time in seconds since epoch (Jan 1 1970 GMT)
      "nonce" : n,           (numeric) The nonce
      "bits" : "1d00ffff", (string) The bits
      "difficulty" : x.xxx,  (numeric) The difficulty
      "chainwork" : "xxxx",  (string) Expected number of hashes required to produce the chain up to this block (in hex)
      "previousblockhash" : "hash",  (string) The hash of the previous block
      "nextblockhash" : "hash"       (string) The hash of the next block
    }
    ```

    * verbosity=2

    ```
    {
      ...,                     Same output as verbosity = 1.
      "tx" : [               (array of Objects) The transactions in the format of the getrawtransaction RPC. Different from verbosity = 1 "tx" result.
             ,...
      ],
      ,...                     Same output as verbosity = 1.
    }
    ```

* 示例：

```shell
> bitcoin-cli getblock "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09" 2
```

### getblockchaininfo

* 说明：获取区块链信息
* 参数：无
* 返回值：区块链状态

```
{
  "chain": "xxxx",        (string) current network name as defined in BIP70 (main, test, regtest)
  "blocks": xxxxxx,         (numeric) the current number of blocks processed in the server
  "headers": xxxxxx,        (numeric) the current number of headers we have validated
  "bestblockhash": "...", (string) the hash of the currently best block
  "difficulty": xxxxxx,     (numeric) the current difficulty
  "mediantime": xxxxxx,     (numeric) median time for the current best block
  "verificationprogress": xxxx, (numeric) estimate of verification progress [0..1]
  "chainwork": "xxxx"     (string) total amount of work in active chain, in hexadecimal
  "pruned": xx,             (boolean) if the blocks are subject to pruning
  "pruneheight": xxxxxx,    (numeric) lowest-height complete block stored
  "softforks": [            (array) status of softforks in progress
     {
        "id": "xxxx",        (string) name of softfork
        "version": xx,         (numeric) block version
        "reject": {            (object) progress toward rejecting pre-softfork blocks
           "status": xx,       (boolean) true if threshold reached
        },
     }, ...
  ],
  "bip9_softforks": {          (object) status of BIP9 softforks in progress
     "xxxx" : {                (string) name of the softfork
        "status": "xxxx",    (string) one of "defined", "started", "locked_in", "active", "failed"
        "bit": xx,             (numeric) the bit (0-28) in the block version field used to signal this softfork (only for "started" status)
        "startTime": xx,       (numeric) the minimum median time past of a block at which the bit gains its meaning
        "timeout": xx,         (numeric) the median time past of a block at which the deployment is considered failed if not yet locked in
        "since": xx,           (numeric) height of the first block to which the status applies
        "statistics": {        (object) numeric statistics about BIP9 signalling for a softfork (only for "started" status)
           "period": xx,       (numeric) the length in blocks of the BIP9 signalling period
           "threshold": xx,    (numeric) the number of blocks with the version bit set required to activate the feature
           "elapsed": xx,      (numeric) the number of blocks elapsed since the beginning of the current period
           "count": xx,        (numeric) the number of blocks with the version bit set in the current period
           "possible": xx      (boolean) returns false if there are not enough blocks left in this period to pass activation threshold
        }
     }
  }
}
```

示例

```shell
> bitcoin-cli getblockchaininfo
```

### getblockcount

* 说明：获取区块链最大高度
* 参数：无
* 返回值：目前区块链最大高度
* 示例：

```shell
> bitcoin-cli getblockcount
```

### getblockhash height

* 说明：获取特定高度区块哈希值
* 参数：
    * height（numeric，必选）区块高度
* 返回值：
    * hash（string）区块哈希值
* 示例：

```shell
> bitcoin-cli getblockhash 1000
```

### getblockheader "hash" ( verbose )

* 说明：获取特定区块头部
* 参数：
    * hash（string，必选）区块哈希值
    * verbose（boolean，可选，默认为true）true-返回json对象，false-返回hex编码数据
* 返回值：
    * verbose=false：hex编码数据
    * verbose=true

    ```
    {
      "hash" : "hash",     (string) the block hash (same as provided)
      "confirmations" : n,   (numeric) The number of confirmations, or -1 if the block is not on the main chain
      "height" : n,          (numeric) The block height or index
      "version" : n,         (numeric) The block version
      "versionHex" : "00000000", (string) The block version formatted in hexadecimal
      "merkleroot" : "xxxx", (string) The merkle root
      "time" : ttt,          (numeric) The block time in seconds since epoch (Jan 1 1970 GMT)
      "mediantime" : ttt,    (numeric) The median block time in seconds since epoch (Jan 1 1970 GMT)
      "nonce" : n,           (numeric) The nonce
      "bits" : "1d00ffff", (string) The bits
      "difficulty" : x.xxx,  (numeric) The difficulty
      "chainwork" : "0000...1f3"     (string) Expected number of hashes required to produce the current chain (in hex)
      "previousblockhash" : "hash",  (string) The hash of the previous block
      "nextblockhash" : "hash",      (string) The hash of the next block
    }
    ```

* 示例

```shell
> bitcoin-cli getblockheader "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09"
```

### getchaintips

* 说明：获取区块链所有信息，包括主链和分支
* 参数：无
* 返回值：

```
Result:
[
  {
    "height": xxxx,         (numeric) height of the chain tip
    "hash": "xxxx",         (string) block hash of the tip
    "branchlen": 0          (numeric) zero for main chain
    "status": "active"      (string) "active" for the main chain
  },
  {
    "height": xxxx,
    "hash": "xxxx",
    "branchlen": 1          (numeric) length of branch connecting the tip to the main chain
    "status": "xxxx"        (string) status of the chain (active, valid-fork, valid-headers, headers-only, invalid)
  }
]
```

其中，状态`status`的取值为：

1.  "invalid"               This branch contains at least one invalid block
2.  "headers-only"          Not all blocks for this branch are available, but the headers are valid
3.  "valid-headers"         All blocks are available for this branch, but they were never fully validated
4.  "valid-fork"            This branch is not part of the active chain, but is fully validated
5.  "active"                This is the tip of the active main chain, which is certainly valid

* 示例：

```shell
bitcoin-cli getchaintips
```

### getchaintxstats ( nblocks blockhash )

* 说明：统计区块链中交易数量以及发送速率
* 参数：
    * nblocks（numeric，可选，默认一个月）用于统计的区块数量
    * blockhash（stirng，可选）统计窗口的结束区块哈希值
* 返回值：

```
{
  "time": xxxxx,        (numeric) The timestamp for the statistics in UNIX format.
  "txcount": xxxxx,     (numeric) The total number of transactions in the chain up to that point.
  "txrate": x.xx,       (numeric) The average rate of transactions per second in the window.
}
```

* 示例：

```shell
> bitcoin-cli getchaintxstats
```

### getdifficulty

* 说明：获取POW的难度
* 参数：无
* 返回值：
    * n.nnn（numeric）最小的难度的倍数
* 示例

```shell
> bitcoin-cli getdifficulty
```

### getmempoolancestors txid (verbose)

* 说明：如果txid在内存池中，返回内存池中其所有父节点
* 参数：
    * txid（string，必选）交易id，必须在内存池中
    * verbose（boolean，可选，默认false）true-返回json对象，false-返回交易id数组
* 返回值：
    * verbose=false

    ```
    [                       (json array of strings)
      "transactionid"           (string) The transaction id of an in-mempool ancestor transaction
      ,...
    ]
    ```

    * verbose=true

    ```
    {                           (json object)
      "transactionid" : {       (json object)
        "size" : n,             (numeric) virtual transaction size as defined in BIP 141. This is different from actual serialized size for witness transactions as witness data is discounted.
        "fee" : n,              (numeric) transaction fee in BTC
        "modifiedfee" : n,      (numeric) transaction fee with fee deltas used for mining priority
        "time" : n,             (numeric) local time transaction entered pool in seconds since 1 Jan 1970 GMT
        "height" : n,           (numeric) block height when transaction entered pool
        "descendantcount" : n,  (numeric) number of in-mempool descendant transactions (including this one)
        "descendantsize" : n,   (numeric) virtual transaction size of in-mempool descendants (including this one)
        "descendantfees" : n,   (numeric) modified fees (see above) of in-mempool descendants (including this one)
        "ancestorcount" : n,    (numeric) number of in-mempool ancestor transactions (including this one)
        "ancestorsize" : n,     (numeric) virtual transaction size of in-mempool ancestors (including this one)
        "ancestorfees" : n,     (numeric) modified fees (see above) of in-mempool ancestors (including this one)
        "depends" : [           (array) unconfirmed transactions used as inputs for this transaction
            "transactionid",    (string) parent transaction id
           ... ]
      }, ...
    }
    ```

* 示例

```shell
> bitcoin-cli getmempoolancestors "mytxid"
```

### getmempooldescendants txid (verbose)

* 说明：如果txid在内存池中，返回内存池中其所有子节点
* 参数：
    * txid（string，必选）交易id，必须在内存池中
    * verbose（boolean，可选，默认false）true-返回json对象，false-返回交易id数组
* 返回值：
    * verbose=false

    ```
    [                       (json array of strings)
      "transactionid"           (string) The transaction id of an in-mempool descendant transaction
      ,...
    ]
    ```

    * verbose=true

    ```
    {                           (json object)
      "transactionid" : {       (json object)
        "size" : n,             (numeric) virtual transaction size as defined in BIP 141. This is different from actual serialized size for witness transactions as witness data is discounted.
        "fee" : n,              (numeric) transaction fee in BTC
        "modifiedfee" : n,      (numeric) transaction fee with fee deltas used for mining priority
        "time" : n,             (numeric) local time transaction entered pool in seconds since 1 Jan 1970 GMT
        "height" : n,           (numeric) block height when transaction entered pool
        "descendantcount" : n,  (numeric) number of in-mempool descendant transactions (including this one)
        "descendantsize" : n,   (numeric) virtual transaction size of in-mempool descendants (including this one)
        "descendantfees" : n,   (numeric) modified fees (see above) of in-mempool descendants (including this one)
        "ancestorcount" : n,    (numeric) number of in-mempool ancestor transactions (including this one)
        "ancestorsize" : n,     (numeric) virtual transaction size of in-mempool ancestors (including this one)
        "ancestorfees" : n,     (numeric) modified fees (see above) of in-mempool ancestors (including this one)
        "depends" : [           (array) unconfirmed transactions used as inputs for this transaction
            "transactionid",    (string) parent transaction id
           ... ]
      }, ...
    }
    ```

* 示例

```shell
> bitcoin-cli getmempooldescendants "mytxid"
```

### getmempoolentry txid

* 说明：如果txid在内存池中，返回交易数据
* 参数：
    * txid（string，必选）交易id，必须在内存池中
* 返回值：

```
{                           (json object)
    "size" : n,             (numeric) virtual transaction size as defined in BIP 141. This is different from actual serialized size for witness transactions as witness data is discounted.
    "fee" : n,              (numeric) transaction fee in BTC
    "modifiedfee" : n,      (numeric) transaction fee with fee deltas used for mining priority
    "time" : n,             (numeric) local time transaction entered pool in seconds since 1 Jan 1970 GMT
    "height" : n,           (numeric) block height when transaction entered pool
    "descendantcount" : n,  (numeric) number of in-mempool descendant transactions (including this one)
    "descendantsize" : n,   (numeric) virtual transaction size of in-mempool descendants (including this one)
    "descendantfees" : n,   (numeric) modified fees (see above) of in-mempool descendants (including this one)
    "ancestorcount" : n,    (numeric) number of in-mempool ancestor transactions (including this one)
    "ancestorsize" : n,     (numeric) virtual transaction size of in-mempool ancestors (including this one)
    "ancestorfees" : n,     (numeric) modified fees (see above) of in-mempool ancestors (including this one)
    "depends" : [           (array) unconfirmed transactions used as inputs for this transaction
        "transactionid",    (string) parent transaction id
       ... ]
}
```

* 示例

```shell
> bitcoin-cli getmempoolentry "mytxid"
```

### getmempoolinfo

* 说明：获取交易内存池的状态
* 参数：无
* 返回值：

```
{
  "size": xxxxx,               (numeric) Current tx count
  "bytes": xxxxx,              (numeric) Sum of all virtual transaction sizes as defined in BIP 141. Differs from actual serialized size because witness data is discounted
  "usage": xxxxx,              (numeric) Total memory usage for the mempool
  "maxmempool": xxxxx,         (numeric) Maximum memory usage for the mempool
  "mempoolminfee": xxxxx       (numeric) Minimum feerate (BTC per KB) for tx to be accepted
}
```

* 示例

```shell
> bitcoin-cli 
```

### getrawmempool ( verbose )

* 说明：获取内存池中的所有交易数据
* 参数：
    * verbose（boolean，可选，默认false）true-返回json对象，false-返回交易id数组
* 返回值：
    * verbose=false

    ```
    [                     (json array of string)
      "transactionid"     (string) The transaction id
      ,...
    ]
    ```

    * verbose=true

    ```
    {                           (json object)
      "transactionid" : {       (json object)
        "size" : n,             (numeric) virtual transaction size as defined in BIP 141. This is different from actual serialized size for witness transactions as witness data is discounted.
        "fee" : n,              (numeric) transaction fee in BTC
        "modifiedfee" : n,      (numeric) transaction fee with fee deltas used for mining priority
        "time" : n,             (numeric) local time transaction entered pool in seconds since 1 Jan 1970 GMT
        "height" : n,           (numeric) block height when transaction entered pool
        "descendantcount" : n,  (numeric) number of in-mempool descendant transactions (including this one)
        "descendantsize" : n,   (numeric) virtual transaction size of in-mempool descendants (including this one)
        "descendantfees" : n,   (numeric) modified fees (see above) of in-mempool descendants (including this one)
        "ancestorcount" : n,    (numeric) number of in-mempool ancestor transactions (including this one)
        "ancestorsize" : n,     (numeric) virtual transaction size of in-mempool ancestors (including this one)
        "ancestorfees" : n,     (numeric) modified fees (see above) of in-mempool ancestors (including this one)
        "depends" : [           (array) unconfirmed transactions used as inputs for this transaction
            "transactionid",    (string) parent transaction id
           ... ]
      }, ...
    }
    ```

* 示例

```shell
> bitcoin-cli getrawmempool true
```

### gettxout "txid" n ( include_mempool )

* 说明：获取UXTO数据
* 参数：
    * txid（string，必选）交易id
    * n（numeric，必选）vout值
    * include_mempool（boolean，可选）是否包含mempool
* 返回值：

```
{
  "bestblock" : "hash",    (string) the block hash
  "confirmations" : n,       (numeric) The number of confirmations
  "value" : x.xxx,           (numeric) The transaction value in BTC
  "scriptPubKey" : {         (json object)
     "asm" : "code",       (string)
     "hex" : "hex",        (string)
     "reqSigs" : n,          (numeric) Number of required signatures
     "type" : "pubkeyhash", (string) The type, eg pubkeyhash
     "addresses" : [          (array of string) array of bitcoin addresses
        "address"     (string) bitcoin address
        ,...
     ]
  },
  "coinbase" : true|false   (boolean) Coinbase or not
}
```

* 示例

```shell
> bitcoin-cli gettxout "txid" 1
```

### gettxoutproof ["txid",...] ( blockhash )

* 说明：获取txid在某个block中的hex编码证明
* 参数：
    * txids（string）txid的json数组
    * blcokhash（string，可选）在该区块中寻找交易
* 返回值：
    * data（string）hex编码数据
* 示例


### gettxoutsetinfo

* 说明：获取未花费交易输出集合的统计数据，可能比较耗时
* 参数：无
* 返回值：

```
{
  "height":n,     (numeric) The current block height (index)
  "bestblock": "hex",   (string) the best block hash hex
  "transactions": n,      (numeric) The number of transactions
  "txouts": n,            (numeric) The number of output transactions
  "bogosize": n,          (numeric) A meaningless metric for UTXO set size
  "hash_serialized_2": "hash", (string) The serialized hash
  "disk_size": n,         (numeric) The estimated size of the chainstate on disk
  "total_amount": x.xxx          (numeric) The total amount
}
```

* 示例

```shell
> bitcoin-cli gettxoutsetinfo
```

### invalidateblock "blockhash"

- 说明：标记某个block及其后代为无效
- 参数：
  - blockhash（stirng，必选）待标记的区块哈希
- 返回值：无
- 示例

```shell
> bitcoin-cli invalidateblock "blockhash" 
```

### reconsiderblock "blockhash"

- 说明：重新标记某个block及其后代为有效，用于取消`invalidateblock`命令
- 参数：
  - blockhash（stirng，必选）待标记的区块哈希
- 返回值：无
- 示例

```shell
> bitcoin-cli reconsiderblock "blockhash" 
```

### preciousblock "blockhash"

* 说明：标记某个block为precious
* 参数：
    * blockhash（stirng，必选）待标记的区块哈希
* 返回值：无
* 示例

```shell
> bitcoin-cli preciousblock "blockhash" 
```

### pruneblockchain

* 说明：修剪区块链
* 参数：
    * height（numeric，必选）修剪的上限高度
* 返回值：
    * n（numeric）最后修剪的区块高度
* 示例

```shell
> bitcoin-cli pruneblockchain 1000
```

### verifychain ( checklevel nblocks )

* 说明：校验区块链数据库
* 参数：
    * checklevel（numeric，可选，0-4，默认3）区块校验的详细程度
    * nblocks（numeric，可选，0-所有，默认6）校验的区块数
* 返回值：
    * true或者false（boolean）是否通过校验
* 示例

```shell
> bitcoin-cli verifychain
```

### verifytxoutproof "proof"

* 说明：Verifies that a proof points to a transaction in a block, returning the transaction it commits to and throwing an RPC error if the block is not in our best chain
* 参数：
    * proof（string，必选）The hex-encoded proof generated by gettxoutproof
* 返回值：
    * txid（array，stirngs）The txid(s) which the proof commits to, or empty array if the proof is invalid
* 示例

```shell
> bitcoin-cli 
```


## == Control ==
### getinfo

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getmemoryinfo ("mode")

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### help ( "command" )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### stop

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### uptime

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


## == Generating ==
### generate nblocks ( maxtries )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### generatetoaddress nblocks address (maxtries)

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


## == Mining ==
### getblocktemplate ( TemplateRequest )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getmininginfo

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getnetworkhashps ( nblocks height )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### prioritisetransaction <txid> <dummy value> <fee delta>

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### submitblock "hexdata"  ( "dummy" )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


## == Network ==
### addnode "node" "add|remove|onetry"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### clearbanned

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### disconnectnode "[address]" [nodeid]

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getaddednodeinfo ( "node" )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getconnectioncount

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getnettotals

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getnetworkinfo

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getpeerinfo

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### listbanned

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### ping

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### setban "subnet" "add|remove" (bantime) (absolute)

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### setnetworkactive true|false

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


## == Rawtransactions ==
### combinerawtransaction ["hexstring",...]

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### createrawtransaction [{"txid":"id","vout":n},...] {"address":amount,"data":"hex",...} ( locktime ) ( replaceable )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### decoderawtransaction "hexstring"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### decodescript "hexstring"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### fundrawtransaction "hexstring" ( options )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### getrawtransaction "txid" ( verbose )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### sendrawtransaction "hexstring" ( allowhighfees )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


## == Util ==
### createmultisig nrequired ["key",...]

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### estimatefee nblocks

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### estimatesmartfee conf_target ("estimate_mode")

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### signmessagewithprivkey "privkey" "message"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### validateaddress "address"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### verifymessage "address" "signature" "message"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


## == Wallet ==
### abandontransaction "txid"

* 说明：标记钱包中的交易及其后代交易为无效，仅对没有被包含在任何区块中或者内存池中的交易可用，对已经标记为无效或者冲突的交易没有效果
* 参数：
    * txid（string，必选）交易id
* 返回值：无
* 示例

```shell
> bitcoin-cli abandontransaction "1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d"
```

### abortrescan

* 说明：停止钱包当前正在进行的重扫描，比如由于导入私钥引起的
* 参数：无
* 返回值：无
* 示例

```shell
> bitcoin-cli abortrescan
```

### addmultisigaddress nrequired ["key",...] ( "account" )

* 说明：添加多签名地址，输入的签名数量为n，需要签名数量为nrequired
* 参数：
    * nrequired（numeric，必选）需要签名的数量
    * keys（string，必选）比特币地址或者公钥的json数组
    * account（string，可选）**已弃用** 地址所属账户
* 返回值：
    * address（string）公钥所对应的比特币地址
* 示例

```shell
> bitcoin-cli addmultisigaddress 2 "[\"16sSauSf5pF2UkUwvKGq4qjNRzBZYqgEL5\",\"171sgjn4YtPu27adkKGrdDwzRTxnRkBfKV\"]"
```

### addwitnessaddress "address"

* 说明：添加witness地址
* 参数：
    * address（string，必选）钱包中的某个地址
* 返回值：
    * witnessaddress（string）新地址，witness脚本的P2SH
* 示例

```shell
> bitcoin-cli addwitnessaddress “address”
```

### backupwallet "destination"

* 说明：备份钱包
* 参数：
    * destination（string）目的地址
* 返回值：无
* 示例

```shell
> bitcoin-cli backupwallet "backup.dat"
```

### bumpfee "txid" ( options )

* 说明：修改手续费并构造新的交易，如果该交易的输出已经被花费，则修改失败
* 参数：
    * txid（string，必选）待修改的txid
    * options（object，可选）

    ```
       {
     "confTarget"        (numeric, optional) Confirmation target (in blocks)
     "totalFee"          (numeric, optional) Total fee (NOT feerate) to pay, in satoshis.
                         In rare cases, the actual fee paid might be slightly higher than the specified
                         totalFee if the tx change output has to be removed because it is too close to
                         the dust threshold.
     "replaceable"       (boolean, optional, default true) Whether the new transaction should still be
                         marked bip-125 replaceable. If true, the sequence numbers in the transaction will
                         be left unchanged from the original. If false, any input sequence numbers in the
                         original transaction that were less than 0xfffffffe will be increased to 0xfffffffe
                         so the new transaction will not be explicitly bip-125 replaceable (though it may
                         still be replaceable in practice, for example if it has unconfirmed ancestors which
                         are replaceable).
     "estimate_mode"     (string, optional, default=UNSET) The fee estimate mode, must be one of:
         "UNSET"
         "ECONOMICAL"
         "CONSERVATIVE"
       }
    ```

* 返回值：

```
{
  "txid":    "value",   (string)  The id of the new transaction
  "origfee":  n,         (numeric) Fee of the replaced transaction
  "fee":      n,         (numeric) Fee of the new transaction
  "errors":  [ str... ] (json array of strings) Errors encountered during processing (may be empty)
}
```

* 示例

```shell
> bitcoin-cli bumpfee <txid>
```

### dumpprivkey "address"

* 说明：打印地址对应的私钥
* 参数：
    * address（string，必选）钱包中的比特币地址
* 返回值：
    * key（stirng）私钥
* 示例

```shell
> bitcoin-cli dumpprivkey "myaddress"
```

### dumpwallet "filename"

* 说明：导出钱包中所有私钥和地址
* 参数：
    * filename（string，必选）导出文件名
* 返回值：
    * json对象

    ```
    {                           (json object)
      "filename" : {        (string) The filename with full absolute path
    }
    ```

* 示例

```shell
> bitcoin-cli dumpwallet "test"
```

### encryptwallet "passphrase"

* 说明：对钱包进行加密，加密后所有需要使用私钥的操作，比如发送或者签名，需要在操作之前调用`walletpassphrase`，操作之后调用`walletlock`，可以调用`walletpassphrasechange`修改密码
* 参数：
    * passphrase（string）加密钱包密码
* 返回值：无
* 示例

```shell
  加密钱包
> bitcoin-cli encryptwallet "my pass phrase"
  使用之前使用密码解密
> bitcoin-cli walletpassphrase "my pass phrase"
  签名消息
> bitcoin-cli signmessage "address" "test message"
  锁定钱包
> bitcoin-cli walletlock
```

### getaccount "address"

* 说明：**已弃用** 获取地址关联的账户
* 参数：
    * address（string，必选）比特币地址
* 返回值：
    * accountname（string）账户名
* 示例

```shell
> bitcoin-cli getaccount "1D1ZrZNe3JUo7ZycKEYQQiQAWd9y54F4XX" 
```

### getaccountaddress "account"

* 说明：**已弃用** 获取账户对应的比特币地址
* 参数：
    * account（string，必选）账户名，""代表默认账户，如果账户名不存在，则新建账户和地址
* 返回值：
    * address（string）账户的比特币地址
* 示例

```shell
> bitcoin-cli getaccountaddress ""
```

### getaddressesbyaccount "account"

* 说明：**已弃用** 返回指定账户的地址列表
* 参数：
    * account（string，必选）账户名
* 返回值：

```
[                     (json array of string)
  "address"         (string) a bitcoin address associated with the given account
  ,...
]
```

* 示例

```shell
> bitcoin-cli getaddressesbyaccount "tabby"
```

### getbalance ( "account" minconf include_watchonly )

* 说明：获取余额，如果不指定account，则返回本机上所有余额，如果指定account（**已弃用**），返回该账户余额
* 参数：
    * account（string，可选，**已弃用**）账户名，""代表默认账户，"*"代表所有账户
    * minconf（numeric，可选，默认1）只包含确认数大于该值的交易
    * include_watchonly（bool，可选，默认false）是否包含只监控的地址
* 返回值：
    * amount（numeric）该账户接收到的BTC数量
* 示例

```shell
> bitcoin-cli getbalance "*" 6
```

### getnewaddress ( "account" )

* 说明：获取新的比特币地址，如果给定account（**已弃用**），则将该地址归属在account中
* 参数：
    * account（string，可选）账户名，如果没给定，则地址归属于默认账户""中，如果给定的account不存在，则新建相应的account
* 返回值：
    * address（string）比特币地址
* 示例

```shell
> bitcoin-cli getnewaddress
```

### getrawchangeaddress

* 说明：新建用于接收变化的比特币地址，与raw transaction配合使用
* 参数：无
* 返回值：
    * address（string）比特币地址
* 示例

```shell
> bitcoin-cli getrawchangeaddress
```

### getreceivedbyaccount "account" ( minconf )

* 说明：**已弃用** 获取账户中各个地址收到的总额，交易至少需要`minconf`个交易
* 参数：
    * account（string，必选）账户名，默认账户为""
    * minconf（numeric，可选，默认1）用于统计的交易所需最小确认数
* 返回值：
    * amount（numeric）账户收到的总额
* 示例

```shell
> bitcoin-cli getreceivedbyaccount "tabby" 6
```

### getreceivedbyaddress "address" ( minconf )

* 说明：获取比特币地址收到的比特币数量
* 参数：
    * address（string，必选）比特币地址
    * minconf（numeric，可选，默认1））用于统计的交易所需最小确认数
* 返回值：
    * amount（numeric）账户收到的总额
* 示例

```shell
> bitcoin-cli getreceivedbyaddress "1D1ZrZNe3JUo7ZycKEYQQiQAWd9y54F4XX" 6
```

### gettransaction "txid" ( include_watchonly )

* 说明：获取钱包内交易的详细信息
* 参数：
    * txid（string，必选）交易ID
    * include_watchonly（bool，可选，默认false）是否包括只监控地址的交易
* 返回值：

```
{
  "amount" : x.xxx,        (numeric) The transaction amount in BTC
  "fee": x.xxx,            (numeric) The amount of the fee in BTC. This is negative and only available for the
                              'send' category of transactions.
  "confirmations" : n,     (numeric) The number of confirmations
  "blockhash" : "hash",  (string) The block hash
  "blockindex" : xx,       (numeric) The index of the transaction in the block that includes it
  "blocktime" : ttt,       (numeric) The time in seconds since epoch (1 Jan 1970 GMT)
  "txid" : "transactionid",   (string) The transaction id.
  "time" : ttt,            (numeric) The transaction time in seconds since epoch (1 Jan 1970 GMT)
  "timereceived" : ttt,    (numeric) The time received in seconds since epoch (1 Jan 1970 GMT)
  "bip125-replaceable": "yes|no|unknown",  (string) Whether this transaction could be replaced due to BIP125 (replace-by-fee);
                                                   may be unknown for unconfirmed transactions not in the mempool
  "details" : [
    {
      "account" : "accountname",      (string) DEPRECATED. The account name involved in the transaction, can be "" for the default account.
      "address" : "address",          (string) The bitcoin address involved in the transaction
      "category" : "send|receive",    (string) The category, either 'send' or 'receive'
      "amount" : x.xxx,                 (numeric) The amount in BTC
      "label" : "label",              (string) A comment for the address/transaction, if any
      "vout" : n,                       (numeric) the vout value
      "fee": x.xxx,                     (numeric) The amount of the fee in BTC. This is negative and only available for the
                                           'send' category of transactions.
      "abandoned": xxx                  (bool) 'true' if the transaction has been abandoned (inputs are respendable). Only available for the
                                           'send' category of transactions.
    }
    ,...
  ],
  "hex" : "data"         (string) Raw data for transaction
}
```

* 示例

```shell
> bitcoin-cli gettransaction "1075db55d416d3ca199f55b6084e2115b9345e16c5cf302fc80e9d5fbf5d48d" true
```

### getunconfirmedbalance

* 说明：获取当前服务器上所有未确认的余额
* 参数：无
* 返回值：
    * balance（numeric）余额
* 示例

```shell
> bitcoin-cli getunconfirmedbalance
```

### getwalletinfo

* 说明：获取钱包当前信息
* 参数：无
* 返回值：

```
{
  "walletname": xxxxx,             (string) the wallet name
  "walletversion": xxxxx,          (numeric) the wallet version
  "balance": xxxxxxx,              (numeric) the total confirmed balance of the wallet in BTC
  "unconfirmed_balance": xxx,      (numeric) the total unconfirmed balance of the wallet in BTC
  "immature_balance": xxxxxx,      (numeric) the total immature balance of the wallet in BTC
  "txcount": xxxxxxx,              (numeric) the total number of transactions in the wallet
  "keypoololdest": xxxxxx,         (numeric) the timestamp (seconds since Unix epoch) of the oldest pre-generated key in the key pool
  "keypoolsize": xxxx,             (numeric) how many new keys are pre-generated (only counts external keys)
  "keypoolsize_hd_internal": xxxx, (numeric) how many new keys are pre-generated for internal use (used for change outputs, only appears if the wallet is using this feature, otherwise external keys are used)
  "unlocked_until": ttt,           (numeric) the timestamp in seconds since epoch (midnight Jan 1 1970 GMT) that the wallet is unlocked for transfers, or 0 if the wallet is locked
  "paytxfee": x.xxxx,              (numeric) the transaction fee configuration, set in BTC/kB
  "hdmasterkeyid": "<hash160>"     (string) the Hash160 of the HD master pubkey
}
```

* 示例

```shell
> bitcoin-cli getwalletinfo
```

### importaddress "address" ( "label" rescan p2sh )

* 说明：导入地址用于监控，如果需要重新扫描区块数据，可能需要比较长的时间
* 参数：
    * script（string，必选）hex编码脚本或者比特币地址
    * label（string，可选，默认""）标签
    * rescan（boolean，可选，默认true）是否重新扫描区块数据
    * p2sh（boolean，可选，默认false）是否添加相应的P2SH脚本
* 返回值：无
* 示例

```shell
> bitcoin-cli importaddress "myscript" "testing" false
```

### importmulti "requests" ( "options" )

* 说明：
* 参数：
    * requests（array，必选）导入的数据

    ```
      [     (array of json objects)
        {
          "scriptPubKey": "<script>" | { "address":"<address>" }, (string / json, required) Type of scriptPubKey (string for script, json for address)
          "timestamp": timestamp | "now"                        , (integer / string, required) Creation time of the key in seconds since epoch (Jan 1 1970 GMT),
                                                                  or the string "now" to substitute the current synced blockchain time. The timestamp of the oldest
                                                                  key will determine how far back blockchain rescans need to begin for missing wallet transactions.
                                                                  "now" can be specified to bypass scanning, for keys which are known to never have been used, and
                                                                  0 can be specified to scan the entire blockchain. Blocks up to 2 hours before the earliest key
                                                                  creation time of all keys being imported by the importmulti call will be scanned.
          "redeemscript": "<script>"                            , (string, optional) Allowed only if the scriptPubKey is a P2SH address or a P2SH scriptPubKey
          "pubkeys": ["<pubKey>", ... ]                         , (array, optional) Array of strings giving pubkeys that must occur in the output or redeemscript
          "keys": ["<key>", ... ]                               , (array, optional) Array of strings giving private keys whose corresponding public keys must occur in the output or redeemscript
          "internal": <true>                                    , (boolean, optional, default: false) Stating whether matching outputs should be treated as not incoming payments
          "watchonly": <true>                                   , (boolean, optional, default: false) Stating whether matching outputs should be considered watched even when they're not spendable, only allowed if keys are empty
          "label": <label>                                      , (string, optional, default: '') Label to assign to the address (aka account name, for now), only allowed with internal=false
        }
      ,...
      ]
    ```

    * options（json，可选）

    ```
      {
         "rescan": <false>,         (boolean, optional, default: true) Stating if should rescan the blockchain after all imports
      }
    ```

* 返回值：与输入大小相同的数组

```
  [{ "success": true } , { "success": false, "error": { "code": -1, "message": "Internal Server Error"} }, ... ]
```

* 示例

```shell
> bitcoin-cli '[{ "scriptPubKey": { "address": "<my address>" }, "timestamp":1455191478 }, { "scriptPubKey": { "address": "<my 2nd address>" }, "label": "example 2", "timestamp": 1455191480 }]'
```

### importprivkey "bitcoinprivkey" ( "label" ) ( rescan )

* 说明：导入私钥
* 参数：
    * bitcoinprivkey（string，必选）私钥，wif_compressed格式
    * label（string，可选，默认""）标签名
    * rescan（boolean，可选，默认true）是否重新扫描所有区块
* 返回值：无
* 示例

```shell
> bitcoin-cli importprivkey "mykey" "testing" false
```

### importprunedfunds

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### importpubkey "pubkey" ( "label" rescan )

* 说明：导入公钥进行监控
* 参数：
    * pubkey（string，必选）公钥
    * label（string，可选，默认""）标签名
    * rescan（boolean，可选，默认true）是否重新扫描所有区块
* 返回值：无
* 示例

```shell
> bitcoin-cli importpubkey "mypubkey" "testing" false
```

### importwallet "filename"

* 说明：从`dumpwallet`导出的文件导入钱包
* 参数：
    * filename（string，必选）钱包文件
* 返回值：无
* 示例

```shell
> bitcoin-cli importwallet "test"
```

### keypoolrefill ( newsize )

* 说明：填充keypool
* 参数：
    * newsize（numeric，可选，默认100）新的keypool大小
* 返回值：无
* 示例

```shell
> bitcoin-cli keypoolrefill
```

### listaccounts ( minconf include_watchonly)

* 说明：获取所有账户及其相应的余额
* 参数：
    * minconf（numeric，可选，默认1）交易所需最小确认数
    * include_watchonly（bool，可选，默认false）是否包含仅监控的地址
* 返回值：

```
{                      (json object where keys are account names, and values are numeric balances
  "account": x.xxx,  (numeric) The property name is the account name, and the value is the total balance for the account.
  ...
}
```

* 示例

```shell
> bitcoin-cli listaccounts 0
```

### listaddressgroupings

* 说明：Lists groups of addresses which have had their common ownership made public by common use as inputs or as the resulting change in past transactions
* 参数：无
* 返回值：

```
[
  [
    [
      "address",            (string) The bitcoin address
      amount,                 (numeric) The amount in BTC
      "account"             (string, optional) DEPRECATED. The account
    ]
    ,...
  ]
  ,...
]
```

* 示例

```shell
> bitcoin-cli listaddressgroupings
```

### listlockunspent

* 说明：获取暂时无法花费的输出
* 参数：无
* 返回值：

```
[
  {
    "txid" : "transactionid",     (string) The transaction id locked
    "vout" : n                      (numeric) The vout value
  }
  ,...
]
```

* 示例

```shell
> bitcoin-cli listlockunspent
```

### listreceivedbyaccount ( minconf include_empty include_watchonly)

* 说明：**已弃用** 获取账户余额
* 参数：
    * minconf（numeric，可选，默认1）交易所需最小确认数
    * include_empty（bool，可选，默认false）是否包含未收到任何比特币的账户
    * include_watchonly（bool，可选，默认false）是否包含仅监控的地址
* 返回值：

```
[
  {
    "involvesWatchonly" : true,   (bool) Only returned if imported addresses were involved in transaction
    "account" : "accountname",  (string) The account name of the receiving account
    "amount" : x.xxx,             (numeric) The total amount received by addresses with this account
    "confirmations" : n,          (numeric) The number of confirmations of the most recent transaction included
    "label" : "label"           (string) A comment for the address/transaction, if any
  }
  ,...
]
```

* 示例

```shell
> bitcoin-cli listreceivedbyaccount 6 true
```

### listreceivedbyaddress ( minconf include_empty include_watchonly)

* 说明：获取比特币地址余额
* 参数：
    * minconf（numeric，可选，默认1）交易所需最小的确认数
    * include_empty（bool，可选，默认false）是否包含未收到任何比特币的账户
    * include_watchonly（bool，可选，默认false）是否包含仅监控的地址
* 返回值：

```
[
  {
    "involvesWatchonly" : true,        (bool) Only returned if imported addresses were involved in transaction
    "address" : "receivingaddress",  (string) The receiving address
    "account" : "accountname",       (string) DEPRECATED. The account of the receiving address. The default account is "".
    "amount" : x.xxx,                  (numeric) The total amount in BTC received by the address
    "confirmations" : n,               (numeric) The number of confirmations of the most recent transaction included
    "label" : "label",               (string) A comment for the address/transaction, if any
    "txids": [
       n,                                (numeric) The ids of transactions received with the address
       ...
    ]
  }
  ,...
]
```

* 示例

```shell
> bitcoin-cli listreceivedbyaddress 6 true
```

### listsinceblock ( "blockhash" target_confirmations include_watchonly include_removed )

* 说明：获取从某个区块之后的所有交易
* 参数：
    * blockhash（string，可选）起始区块号
    * target_confirmations（numeric，可选，默认1）结束区块号距离区块订单的确认数
    * include_watchonly（bool，可选，默认false）是否包含仅监控的地址
    * include_removed（bool，可选，默认true）是否包含因为重排被移除的交易
* 返回值：

```
{
  "transactions": [
    "account":"accountname",       (string) DEPRECATED. The account name associated with the transaction. Will be "" for the default account.
    "address":"address",    (string) The bitcoin address of the transaction. Not present for move transactions (category = move).
    "category":"send|receive",     (string) The transaction category. 'send' has negative amounts, 'receive' has positive amounts.
    "amount": x.xxx,          (numeric) The amount in BTC. This is negative for the 'send' category, and for the 'move' category for moves
                                          outbound. It is positive for the 'receive' category, and for the 'move' category for inbound funds.
    "vout" : n,               (numeric) the vout value
    "fee": x.xxx,             (numeric) The amount of the fee in BTC. This is negative and only available for the 'send' category of transactions.
    "confirmations": n,       (numeric) The number of confirmations for the transaction. Available for 'send' and 'receive' category of transactions.
                                          When it's < 0, it means the transaction conflicted that many blocks ago.
    "blockhash": "hashvalue",     (string) The block hash containing the transaction. Available for 'send' and 'receive' category of transactions.
    "blockindex": n,          (numeric) The index of the transaction in the block that includes it. Available for 'send' and 'receive' category of transactions.
    "blocktime": xxx,         (numeric) The block time in seconds since epoch (1 Jan 1970 GMT).
    "txid": "transactionid",  (string) The transaction id. Available for 'send' and 'receive' category of transactions.
    "time": xxx,              (numeric) The transaction time in seconds since epoch (Jan 1 1970 GMT).
    "timereceived": xxx,      (numeric) The time received in seconds since epoch (Jan 1 1970 GMT). Available for 'send' and 'receive' category of transactions.
    "bip125-replaceable": "yes|no|unknown",  (string) Whether this transaction could be replaced due to BIP125 (replace-by-fee);
                                                   may be unknown for unconfirmed transactions not in the mempool
    "abandoned": xxx,         (bool) 'true' if the transaction has been abandoned (inputs are respendable). Only available for the 'send' category of transactions.
    "comment": "...",       (string) If a comment is associated with the transaction.
    "label" : "label"       (string) A comment for the address/transaction, if any
    "to": "...",            (string) If a comment to is associated with the transaction.
  ],
  "removed": [
    <structure is the same as "transactions" above, only present if include_removed=true>
    Note: transactions that were readded in the active chain will appear as-is in this array, and may thus have a positive confirmation count.
  ],
  "lastblock": "lastblockhash"     (string) The hash of the block (target_confirmations-1) from the best block on the main chain. This is typically used to feed back into listsinceblock the next time you call it. So you would generally use a target_confirmations of say 6, so you will be continually re-notified of transactions until they've reached 6 confirmations plus any new ones
}
```

* 示例

```shell
> bitcoin-cli listsinceblock "000000000000000bacf66f7497b7dc45ef753ee9a7d38571037cdb1a57f663ad" 6
```

### listtransactions ( "account" count skip include_watchonly)

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### listunspent ( minconf maxconf  ["addresses",...] [include_unsafe] [query_options])

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### listwallets

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### lockunspent unlock ([{"txid":"txid","vout":n},...])

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### move "fromaccount" "toaccount" amount ( minconf "comment" )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### removeprunedfunds "txid"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### sendfrom "fromaccount" "toaddress" amount ( minconf "comment" "comment_to" )

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### sendmany "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] replaceable conf_target "estimate_mode")

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### sendtoaddress "address" amount ( "comment" "comment_to" subtractfeefromamount replaceable conf_target "estimate_mode")

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### setaccount "address" "account"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### settxfee amount

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```

### signmessage "address" "message"

* 说明：
* 参数：
* 返回值：
* 示例

```shell
> bitcoin-cli 
```


