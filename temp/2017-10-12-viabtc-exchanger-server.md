# 架构

HTTP Server

WebSocket Server


RPC Server: 
* matchengine
* marketprice
* readhistory

Alert Server

Monitor Server 用于监控服务器状态

Listener和Worker是做什么用的

存储：

Redis
MySQL
Kafka



底层是基于libev的事件驱动服务器


# 模块

## MatchEngine

### 数据结构

asset采用字典`dict_asset`存储，键值为币种名，存储内容为结构体`asset_type`

```C
struct asset_type {
    int prec_save;
    int prec_show;
};
```

balance采用字典`dict_balance`存储，键值为结构体`balance_key`，其中`type`分为`available`和`freeze`，存储内容为余额

```C
struct balance_key {
    uint32_t    user_id;
    uint32_t    type;
    char        asset[ASSET_NAME_MAX_LEN + 1];
};
```

market采用字典`dict_markets`存储，键值为交易对名称，存储内容为结构体`market_t`

```C
typedef struct market_t {
    char            *name;
    char            *stock;
    char            *money;

    int             stock_prec;
    int             money_prec;
    int             fee_prec;
    mpd_t           *min_amount;

    dict_t          *orders;
    dict_t          *users;

    skiplist_t      *asks;
    skiplist_t      *bids;
} market_t;
```

用户余额和未完成的订单（挂单）在内存（或者redis）中存储，每隔1分钟，将挂单和用户余额同步到数据库（MySQL）中，完成后的订单以及用户余额的变动情况需要写入数据库


### 流程图

* 初始化用户账户余额`init_balance`
* 初始化用户余额更新`init_update`
* 初始化交易对`init_trade`
* 从数据库中读取信息`init_from_db`
    * 获取上一次处理的订单和交易号
    * 每隔1s钟，将`dict_balance`和所有未成交买卖单数据进行持久化写入数据库


# 代码解析

