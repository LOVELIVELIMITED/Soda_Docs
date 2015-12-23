# LeanCache 使用指南

## 介绍

LeanCache 使用 [Redis](http://redis.io/) 来提供高性能、高可用的 Key-Value 内存存储。主要用作缓存数据的存储，也可以用作持久化数据的存储。

对于以下场景非常合适：

* 某些数据量少，但是读写比例很高，比如某些应用的菜单可以通过后台调整，所有用户会频繁读取该信息。
* 需要同步锁或者队列处理，比如秒杀、抢红包等场景。
* 多个云引擎节点的协同和通信。

下图为 LeanCache 和云引擎配合使用的架构：

<div style="max-width:620px" data
-fix><img src="images/leancache_arch.png" class="img-responsive" alt=""></div>

如果使用得当，LeanCache 不仅可以极大地提高性能，还能**降低成本**，因为某些高频率的查询不需要走存储服务（存储服务按调用次数收费）。

## 主要特性

* **高性能**：接近 7 万的 QPS
* **高可用**：基于 [AOF 持久化](http://www.redis.cn/topics/persistence.html) 的 Master-Slave 主从热备份。
* **在线扩容**：在线调整容量，数据平滑迁移。
* **多实例**：满足更大容量或更高性能的需求。

## 创建实例

进入 [控制台 /<span class="text-muted">（选择应用）</span>/ 存储 / 云引擎 / LeanCache](/cloud.html?appid={{appid}}#/cache)，点击 **创建实例**，如下图所示：

<div style="max-width: 620px;" data-fix><img src="images/leancache_controller.png" class="img-responsive" alt=""></div>

<div class="callout callout-info">LeanCache 实例一旦生成，就开始计费，因此请认真对待该操作。</div>

创建实例时可设置的参数有：

* **实例名称**：最大长度不超过 32 个字符，限英文、数字、下划线，且不能以数字开头。每个开发者账户下 LeanCache 实例名称**必须唯一**，不填则为随机字符串。
* **最大容量**：可选 128 MB、256 MB、512 MB、1 GB、2 GB、4 GB、8 GB。
* **删除策略**：内存满时对 key 的删除策略，默认为 `volatile-lru`，更多选择请参考 [数据删除策略](#数据删除策略)。

### 数据删除策略

目前我们支持如下几种策略：

策略|说明
---|---
`noeviction`|不删除，当内存满时，直接返回错误。
`allkeys-lru`|优先删除最近最少使用的 key，以释放内存。
`volatile-lru`|优先删除设定了过期时间的 key 中最近最少使用的 key，以释放内存。
`allkeys-random`|随机删除一个 key，以释放内存。
<code class="text-nowrap">volatile-random</code>|从设定了过期时间的 key 中随机删除一个，以释放内存。
`volatile-ttl`|从设定了过期时间的 key 中删除最老的 key，以释放内存。
  
请注意，如果所有的 key 都不设置过期时间，那么 `volatile-lru`、`volatile-random`、`volatile-ttl` 这三种策略会等同于 `noeviction`（不删除）。更详细的内容请参考 [Using Redis as an LRU cache](http://redis.io/topics/lru-cache)。

<div class="callout callout-info">LeanCache 实例一旦生成后，该属性不可修改。</div>

## 删除实例

进入 [控制台/<span class="text-muted">（选择应用）</span>/ 存储 / 云引擎 / LeanCache](/cloud.html?appid={{appid}}#/cache)，在「当前应用的实例」下，点击每个实例右上角的齿轮图标（<i class="icon icon-gear"></i>），在出现的窗口底部，点击「删除」按钮。

删除「其他应用的可用实例」下的实例，有两种方式：点击每个实例的「隶属于」链接，切换到相应的应用下；或者从页面顶部的导航条，点击应用图标（<i class="icon icon-blank-app"></i>） 来切换应用。然后再按上一段的提示，进行删除操作。

## 使用

LeanCache 目前支持通过云引擎访问。实例创建完毕后，云引擎应用就可以从环境变量中获取 `REDIS_URL_<实例名称>` 的 Redis 连接字符串，通过该信息连接并使用 Redis。

LeanCache 不提供外网直接访问。如果需要进行简单的数据操作或者查看状态，可以查看控制台：

![image](images/leancache_status.png)

或者使用命令行工具。

### 在命令行工具中使用

**提示**：[命令行工具](cloud_code_commandline.html) 在 v0.8.0 增加了 redis 命令来支持 LeanCache 的操作。

可以通过下列命令查询当前应用有哪些 LeanCache 实例：

``` 
avoscloud redis list
```

可以通过下列命令创建一个交互式的 client：

``` 
avoscloud redis <实例名称>
```


### 在云引擎中使用（Node.js 环境）

首先添加相关依赖到云引擎应用中：

``` 
"dependencies": {
  ...
  "redis": "2.2.x",
  ...
}
```

然后可以使用下列代码获取 Redis 连接：

``` 
var client = require('redis').createClient(process.env['REDIS_URL_<实例名称>']);
```

### 在云引擎中使用（Python 环境）

首先添加相关依赖到云引擎应用的 `requirements.txt` 中：

``` 
Flask>=0.10.1
leancloud-sdk>=1.0.9
...
redis
```

然后可以使用下列代码获取 Redis 连接：

``` 
import os
import redis

r = redis.from_url(os.environ.get("REDIS_URL_<实例名称>"))
```

### 在云引擎中使用（旧版云代码环境）

旧版云代码环境不支持 LeanCache，建议升级到云引擎 3.0 Node.js 环境，升级文档详见 [云引擎 2.0 升级 3.0 指南](leanengine_upgrade_3.html)。

### 多应用间共享使用

LeanCache 实例在开发者账户内全局可见，并不与某个应用固定绑定。所以在某个应用内创建的 LeanCache 实例，其他应用也一样可以使用，其调用方法和上述例子一样。

对于某些使用场景，譬如 O2O 行业的用户端和管理端，或者网络租约车平台的乘客端和司机端，需要多个应用共享同一个 LeanCache 数据，这一点将会非常有用。

## 性能

下面是使用 redis-benchmark 测试一个典型的容量为 2 GB 的 LeanCache 实例的性能表现：

``` 
$ redis-benchmark -n 100000 -q
PING_INLINE: 69783.67 requests per second
PING_BULK: 68306.01 requests per second
SET: 68634.18 requests per second
GET: 67659.00 requests per second
INCR: 67294.75 requests per second
LPUSH: 61236.99 requests per second
LPOP: 62460.96 requests per second
SADD: 63451.78 requests per second
SPOP: 64724.92 requests per second
LPUSH (needed to benchmark LRANGE): 64808.82 requests per second
LRANGE_100 (first 100 elements): 62189.05 requests per second
LRANGE_300 (first 300 elements): 64267.35 requests per second
LRANGE_500 (first 450 elements): 66934.41 requests per second
LRANGE_600 (first 600 elements): 61462.82 requests per second
MSET (10 keys): 60096.15 requests per second
```

## 可靠性

每个 LeanCache 实例使用 Redis Master-Slave 主从热备，其下的多个观察节点每隔 1 秒钟观察一次主节点的状态。如果「主节点」最后一次有效响应在 5 秒之前，则该观察节点认为主节点失效。如果超过总数一半的观察节点发现主节点失效，则自动将「从节点」切换为主节点，并会有新的从节点启动重新组成主从热备。这个过程对应用完全透明，不需要修改连接字符串或者重启，整个切换过程应用只有几秒钟会出现访问中断。

与此同时，从节点还会以 [AOF 方式](http://www.redis.cn/topics/persistence.html) 将数据持久化存储到可靠的中央文件中，每秒刷新一次。如果很不巧主从节点同时失效，则马上会有新的 Redis 节点启动，并从 AOF 文件恢复，完成后即可再次提供服务，并且会有新的从节点与之构成主从热备。

### 极端情况下的数据丢失

当一个实例中的主节点失效，而最新的数据没有同步到对应的从节点时，主从切换会造成这部分数据丢失。

当主、从节点同时失效，未同步到从节点和从节点未刷新到磁盘 AOF 文件中的数据将会丢失。

<!-- TODO: 2015-11-19: 我们会提供迁移的日志文件，从中找出服务器因何原因节点迁移。-->

## 在线扩容

你可以在线扩大（或者缩小） LeanCache 实例的最大内存容量。整个过程可能会持续一段时间，在此期间 LeanCache 会中断几秒钟进行切换，其他时间都正常提供服务。

<div class="callout callout-danger">缩小容量之前，请务必确认现有数据体积小于目标容量，否则可能造成意料之外的结果。</div>

## 多实例

有些时候，你可能希望在一个应用里创建多个 LeanCache 实例：

* **需要存储的数据大于 8 GB**：目前我们提供的实例最大容量为 8 GB。如果有大于此容量的数据，建议你创建多个实例，然后根据功能来划分，比如一个用来做持久化，另一个用来做缓存。
* **需要更高的性能**：如果单实例的性能已经成为应用的瓶颈，你可以创建多个实例，然后在云引擎中同时连接，并自己决定 key 的分片策略，使请求分散到不同的实例来获得更高的性能。

添加实例的方式请参考 [创建实例](#创建实例)。

## 价格

因为用户可能需要随时调整 LeanCache 实例的容量，所以为了方便计算，我们按照每个实例当天所使用的「最大容量」来结算，而不是「实际使用容量」。不同容量的 LeanCache 实例的价格如下：

容量 | 每日
---:|---:
128 MB|1 元
256 MB|1.5 元
512 MB|3 元
1 GB|5 元
2 GB|10 元
4 GB|20 元
8 GB|50 元

### 费用计算

LeanCache 采取按天扣费，使用时间不足一天按一天收费，次日凌晨系统从账户余额中扣费。付费范围包括当前账户下
隶属于每个应用的所有 LeanCache 实例，取每个实例当天使用的最大容量的价格，累计相加计算出总的使用费用。

如果在系统扣费之时，账户没有充足余额，那么在扣费当天的上午 10 点，账户内所有应用使用的**全部实例会停止服务**，但数据仍会保留，期限为 1 个月。

已停止服务的实例状态显示为 <span class="label label-warning">未运行</span>。要恢复服务，需要向账户充值。在账户余额补足后的 5 分钟内，已停止服务的所有实例将会自动恢复运行。

<div class="callout callout-info">账户充值请到 [财务概况](/bill.html#/bill/general) 中进行。建议使用「支付宝」来实时充值，避免对公账户付款（银行转账）产生的到款延迟而影响服务开通的时间。</div>

### 删除无用实例

为了避免发生不必要的使用费，请及时删除不再使用的实例，步骤请参考 [删除实例](#删除实例)。