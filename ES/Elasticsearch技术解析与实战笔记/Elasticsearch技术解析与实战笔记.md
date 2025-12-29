# 书籍介绍

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251226115540819.png" alt="image-20251226115540819" style="zoom:25%;" />

> 17年的老书，主要用来过一些基础。

本书主要内容包括:

第1章“Elasticsearch 入门”,介绍 Elasticsearch 是什么、Apache Lucene 的基础知识、 Elasticsearch 的术语、JSON 介绍、Elasticsearch 的安装运行、Elasticsearch 的 HTTP 接口和 Elasticsearch 的 Java API 接口。 



第2章“索引”,介绍和 Elasticsearch 索引相关的接口,包括索引管理、索引映射管理、 索引别名、索引设置、索引监控、索引其他重要接口以及文档管理 



第3章“映射”,介绍 Elasticsearch 文档的内部结构,Elasticsearch 支持的字段类型,除 此之外,本章还将展示 Elasticsearch 内置的元字段,映射的参数和动态映射功能。

 

第4章“搜索”,详细介绍和搜索相关的知识,包括搜索的详细参数,搜索的评分机制、 滚动查询、系统内部隐藏内容的查询、搜索模板等;接着介绍 Elasticsearch 的**领域查询语言 DSL (Domain-specific Language)**相关的知识点;最后介绍 Elasticsearch 的精简查询接口。 



第5章“聚合”,聚合可以对文档中的数据进行统计汇总、分组等,通过聚合可以完成 很多的统计功能,该章介绍聚合相关的知识,包括度量聚合、分组聚合和管道聚合。 



第6章“集群管理”,详细介绍和集群相关的内容,包括集群的监控、集群分片迁移、 集群的节点配置、集群发现、集群平衡的原理和配置。



第7章“索引分词器”,介绍 Elasticsearch 的分词器和分词的原理,以及如何添加新的 分词器等;还介绍 Elasticsearch 的插件相关知识,包括插件安装等。 



第8章“高级配置”,介绍 Elasticsearch 的高级配置,包括网络配置、脚本配置、快照 和恢复配置、线程池配置和索引配置。



第9章“告警、监控和权限管理”,介绍 Elasticsearch 官方支持的几个比较好的插件: Watcher、Marvel、Shield,它们可以对 Elasticsearch 进行告警、监控和权限管理。 



第10章“ELK 应用”,介绍 Elasticsearch 与另外两个产品 Logstash 和 Kibana 如何组合 使用, Logstash 是对日志进行收集和处理,Kibana 是对存储在 Elasticsearch 中的索引进行展 示和报表分析;最后通过一个简单的示例来介绍 ELK 几个产品是如何关联的。



# 第1章 Elasticsearch 入门

## 1.1 es是什么

Elasticsearch (ES)是一个基于Lucene 构建的开源、分布式、RESTful 接口**全文搜索引擎**。

Elasticsearch 还是一个**分布式文档数据库**,其中每个字段均是被索引的数据且可被搜索, 它能够扩展至数以百计的服务器存储以及处理 PB级的数据。

优点：

- **横向可扩展性**:只需要增加一台服务器,做一点儿配置,启动一下 Elasticsearch 进 程就可以并入集群。 
- **分片机制提供更好的分布性**:同一个索引分成多个分片(sharding),这点类似于 HDFS 的块机制;分而治之的方式可提升处理效率。
- **高可用**:提供复制(replica)机制,一个分片可以设置多个复制,使得某台服务器在 岩机的情况下,集群仍旧可以照常运行,并会把服务器宕机丢失的数据信息复制恢 复到其他可用节点上。 
- **使用简单**:只需一条命令就可以下载文件,然后很快就能搭建一个站内搜索引擎。

相关产品：

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251226120900737.png" alt="image-20251226120900737" style="zoom:50%;" />



## 1.2 全文搜索

> **全文搜索**是指计算机搜索程序通过扫描文章中的每一个词,对每一个词建立一个索引, 指明该词在文章中出现的次数和位置,当用户查询时,搜索程序就根据事先建立的索引进 行查找,并将查找的结果反馈给用户。

### Lucene 倒排索引（inverted index）

> 不是由记录来确定属性值,而是由属性值 来确定记录的位置,因而称为倒排索引。
>
> 倒排索引：包含属性值x的文档有 文档1、文档2
>
> ”正排索引“：文档1包含属性值x、y



例子：

```
假设有两篇文章1和文章2
文章1的内容为:Tom lives in Guangzhou, I live in Guangzhou too.
文章2的内容为:He once lived in Shanghai.
```

1、分词并建立索引

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251226121742028.png" alt="image-20251226121742028" style="zoom:50%;" />

> 以live 这行为例,我们说明一下该结构:live 在文章1中出现了2次,文章2中出现 了一次,它的出现位置为“2,5,2”这表示什么呢?我们需要结合文章号和出现频率来分析, 文章1中出现了2次,那么“2,5”就表示 live在文章1中出现的两个位置,文章2中出现 了一次,剩下的“2”就表示 live 是文章 2中的第2个关键字。



2、数据结构实现

实现时,Lucene 将上面三列分别作为**词典文件(Term Dictionary)、频率文件(frequencies)、 位置文件(positions)** 保存。其中词典文件不仅保存了每个关键词,还保留了指向频率文件和 位置文件的指针,通过指针可以找到该关键字的频率信息和位置信息。

Lucene 中使用了 **felid** 的概念,用于表达信息所在位置(如标题中、文章中、URL 中)。（即字段名）



3、压缩算法

为了减小索引文件的大小,Lucene 对索引还使用了压缩技术。 

首先,对词典文件中的关键词（文本）进行了压缩,关键词压缩为<前缀长度,后缀>,例如: 当前词为“阿拉伯语”,上一个词为“阿拉伯”,那么“阿拉伯语”压缩为<3,语 > 

其次大量用到的是对数字的压缩,数字只保存与上一个值的差值(这样可以减少数字的 长度,进而减少保存该数字需要的字节数)。例如当前文章号是16389(不压缩要用3个字 节保存),上一文章号是16382,压缩后保存7(只用一个字节)。



## 1.3 Elasticsearch 术语及概念

1、索引词(term) 

在 Elasticsearch 中索引词(term)是一个能够被索引的精确值。foo、Foo、FOO几个单 词是不同的索引词。索引词(term)是可以通过 term 查询进行准确的搜索。



2、文本 (text) 

文本是一段普通的非结构化文字。通常,文本会被分析成一个个的索引词,存储在 Elasticsearch 的索引库中。



3、分析 (analysis) 

> 其实就是分词

分析是将文本转换为索引词的过程,分析的结果依赖于分词器。比



4、集群(cluster) 

集群由一个或多个节点组成,对外提供服务,对外提供索引和搜索功能。在所有节点, 一个集群有一个唯一的名称默认为“Elasticsearch”。此名称是很重要的,因为每个节点只 能是集群的一部分,当该节点被设置为相同的集群名称时,就会自动加入集群。请注意,一个节点只能加入一个集群。此外,你还可以拥有多个独立的集群,每个集群都有其不同的集群名称。

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251226123052150.png" alt="image-20251226123052150" style="zoom:50%;" />



5、节点(node) 

一个节点是一个逻辑上独立的服务,它是集群的一部分,可以存储数据,并参与集群的索引和搜索功能。

在网络中 Elasticsearch 集群通过节点名称进行管理和通信。一个节点可以被配置加入一个特定的集 群。默认情况下,每个节点会加入名为 Elasticsearch。



6、路由(routing) 

当存储一个文档的时候,它会存储在唯一的主分片中,具体哪个分片是通过散列值进行 选择。默认情况下,这个值是由文档的ID 生成。如果文档有一个指定的父文档,则从父文 档ID 中生成,该值可以在存储文档的时候进行修改。



7、分片(shard) 

分片是单个 Lucene 实例,这是 Elasticsearch 管理的比较底层的功能。索引是指向主分片和副本分片的逻辑空间。

分片主要有两个很重要的原因是:

- 允许水平分割扩展数据。
- 允许分配和并行操作(可能在多个节点上)从而提高性能和吞吐量。

> 有文档存储数量限制,你可以在一个 单一的Lucene 索引中存储的最大值为 lucene-5843,极限是2147483519 (= integer. max_value - 128, 2^31-128)个文档。
>
> _cat/shards 监控分片的大小



8、主分片(primary shard) 

每个文档都存储在一个分片中,当你存储一个文档的时候,系统会首先存储在主分片 中,然后会复制到不同的副本中。默认情况下,一个索引有5个主分片。你可以事先制定分 片的数量,当分片一旦建立,则分片的数量不能修改。



9、副本分片(replica shard) 每一个分片有零个或多个副本。

副本主要是主分片的复制,其中有两个目的: 

- 增加高可用性:当主分片失败的时候,可以从副本分片中选择一个作为主分片。 
- 提高性能:当查询的时候可以到主分片或者副本分片中进行查询。默认情况下,一 个主分片配有一个副本,但副本的数量可以在后面动态地配置增加。副本分片必须部署在不同的节点上,不能部署在和主分片相同的节点上。



10、复制(replica) 

复制是一个非常有用的功能,不然会有**单点问题**（单服务不可用造成的问题）。当网络中的某个节点出现问题的时候,复制可以对故障进行转移,保证系统的高可用。因此,Elasticsearch 允许你创建一个或 多个拷贝,你的索引分片就形成了所谓的副本或副本分片。



11、索引 (index) 

索引是具有相同结构的文档集合。



12、类型(type) 

在索引中,可以定义一个或多个类型,**类型是索引的逻辑分区**。

在一般情况下,一种类 型被定义为具有一组公共字段的文档。例如,让我们假设你运行一个博客平台,并把所有的 数据存储在一个索引中。在这个索引中,你可以定义一种类型为用户数据,一种类型为博客 数据,另一种类型为评论数据。==》es7.0后去掉了type配置，固定为_doc



13、文档 (document) 

文档是存储在 Elasticsearch 中的一个 JSON 格式的字符串。

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251226142142319.png" alt="image-20251226142142319" style="zoom: 67%;" />



14、映射(mapping) 

映射像关系数据库中的表结构,每一个索引都有一个映射,它定义了索引中的每一个字段类型（properitis）,以及一个索引范围内的设置。

一个映射可以事先被定义,或者在第一次存储文档的 时候自动识别



15、字段(field) 

文档中包含零个或者多个字段,字段可以是一个简单的值(例如字符串、整数、日期), 也可以是一个数组或对象的嵌套结构。



16、来源字段(source field) 

默认情况下,你的原文档将被存储在_source 这个字段中,当你查询的时候也是返回这 个字段。



17、主键(ID) 

ID 是一个文件的唯一标识,如果在存库的时候没有提供ID,系统会自动生成一个ID, 文档的 index/type/id 必须是唯一的



## 1.4  API对外接口

### API约定

#### 多索引参数

```bash
# 查询多个指定索引
GET /index1,index2,index3/_search
{
  "query": { "match_all": {} }
}

# 使用通配符
GET /test*/_search                    # 所有test开头的索引
GET /log-2024-*/_search              # 2024年所有日志索引

# 排除特定索引
GET /test*,-test3/_search            # 所有test开头，排除test3
GET /2024-*, -2024-12*/_search       # 2024年数据，排除12月

# _all 查询所有索引（慎用，性能影响大）
GET /_all/_search
{
  "query": { "match_all": {} },
  "size": 10  # 限制结果数
}

# 查询指定多个索引模式
GET /prod*,dev*,test*/_search
```

多索引查询还支持以下参数: 

- ignore unavailable :当索引不存在或者关闭的时候,是否忽略这些索引,值为true 和false 
- allow_no_indices :当使用通配符查询所有索引的时候,当有索引不存在的时候是否 返回查询失败。值为true 和 false。 
- expand wildcards :控制通配符索引表达式匹配具体哪一类的索引,值为 open. close, none, all open 表示只支持开启状态的索引, close 表示只支持关闭状态的索 引, none 表示不可用, all 表示同时支持 open 和 close 索引。

ignore unavailable 

```bash
# 忽略不存在的索引（默认false）
GET /existing_index,non_existent_index/_search?ignore_unavailable=true
{
  "query": { "match_all": {} }
}

# 响应：只返回existing_index的结果，不报错

# 如果设为false（默认）
GET /existing_index,non_existent_index/_search?ignore_unavailable=false
# 响应：404错误，索引不存在
```

**allow_no_indices**

```bash
# 通配符匹配不到索引时的行为（默认true）
GET /non_existent_*/_search?allow_no_indices=true
# 响应：返回空结果，不报错

GET /non_existent_*/_search?allow_no_indices=false  
# 响应：404错误，没有匹配的索引
```

**expand_wildcards**

```bash
# 控制通配符匹配的索引状态
GET /test*/_search?expand_wildcards=open          # 只匹配开启的索引
GET /test*/_search?expand_wildcards=closed       # 只匹配关闭的索引  
GET /test*/_search?expand_wildcards=all          # 匹配所有状态索引
GET /test*/_search?expand_wildcards=none         # 禁用通配符
```



#### 日期筛选

> 日期筛选可以限定时间序列索引的搜索范围
>
> ==》索引名称必须包含日期模式（如 `test-2024.01.15`）

```
日期筛选的语法为:
<static_name{date_math_expr{date_format|time_zone}}>

语法解释:
static_name:索引的名称:
date_math_expr:动态日期计算表达式;
date format:日期格式:
time_zone:时区,默认为 UTC。
```

示例：

```bash
curl -XGET '127.0.0.1:9200/<logstash-{now%2Fd-2d}>/_search' {
"query" : {...}
}
```



#### 通用参数

> pretty 参数

?pretty=true时,请求的返回值是经过格 式化后的JSON 数据，format=yaml  --》 YAML格式

> human 参数

human=true 的时候输出更适合人类阅读的数据, 但这会消耗更多的资源,默认是 false

> 日期表达式

大多数参数接受格式化日期表达式,如范围查询 gt(大于) 和lt(小于), 或在日期聚合中用 from to 来表达时间范围。表达式设定的日期为 now 或者日期字符串加 +1h   -1h   /h分别表示 增加一小时。 减少一个小时。 上一个小时。 支持的时间单位为:y(年)、M(月)、w(周)、d(日)、h(小时)、m(分钟)、s(秒)。

> 响应过滤(filter_path)

所有的返回值可以通过 filter_path 来减少返回值的内容,多 个值可以用逗号分开。例：

```bash
GET /test/_search?filter_path=took,hits.hits._id,hits.hits._score,,hits.hits._source.name
==》
{
  "took" : 0,
  "hits" : {
    "hits" : [
      {
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "abw"
        }
      },
      {
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "狂神说java"
        }
      }
    ]
  }
}

# 它也支持通配符*匹配任何部分字段的名称,我们可以用两个通配符** 来匹配不确定名称的字段
GET /test/_search?filter_path=hits.hits._source.**
==》
{
  "hits" : {
    "hits" : [
      {
        "_source" : {
          "name" : "abw",
          "age" : 10,
          "desc" : "还行",
          "tags" : [
            "技术宅",
            "帅哥",
            "渣男"
          ]
        }
      },
      {
        "_source" : {
          "name" : "狂神说java",
          "age" : 10,
          "desc" : "还行",
          "tags" : [
            "技术宅",
            "帅哥",
            "渣男"
          ],
          "title" : "love China"
        }
      }
    ]
  }
}

```



> flat_settings参数

```bash
flat_settings参数不能用在 _searchAPI 中。flat_settings是用于集群设置和索引设置相关的API，用于使返回紧凑

1. 获取集群设置（正确用法）
GET /_cluster/settings?flat_settings=true
2. 获取索引设置（正确用法）
GET /test/_settings?flat_settings=true
3. 获取索引映射（正确用法）
GET /test/_mapping?flat_settings=true
```





# 第2章 索引

## 2.1 索引管理

### 2.1.1 CURD索引

创建索引的时候可以通过修改 number_of_shards 和 number_of_replicas 参数的数量来修 改分片和副本的数量。在默认的情况下分片的数量是5个,副本的数量是1个。

```bash
# 例如,创建三个主分片,两个副本分片的索引。
PUT /secisland/
{
  "settings": {
    "index": {
      "number_of_shards": 3,
      "number_of_replicas": 2
    }
  }
}

可简写为：
{"settings": {"number_of_shards": 3, "number_of_replicas": 2}
}


# 修改副本数量
PUT /secisland/_settings/
{
  "number_of_replicas": 2
}
```

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251229194341993.png" alt="image-20251229194341993" style="zoom:33%;" />



mapping设置：

```bash
POST /secisland/_mapping/
{
  "properties": {
    "logType": {
      "type": "keyword"
    }
  }
}

# es 2.0 版本：
POST /secisland/_mapping/
{
  "secilog": {
    "properties": {
      "logType": {
        "type": "text",
        "index": "not_analyzed"
      }
    }
  }
}

```

注：**Elasticsearch 5.x+ 的重要变化**

1. **移除了类型（Type）**
   - 旧版本：`index/type`结构
   - 新版本：每个索引只有一个隐式的 `_doc`类型
2. **字段类型变化**
   - `string`类型拆分为 `text`和 `keyword`
   - `index: not_analyzed`被 `type: keyword`替代
   - `index: analyzed`被 `type: text`替代
3. **映射API变化**
   - 不再需要在mapping中指定type名称



```bash
DELETE /secisland/
GET /secisland/

# 判断类型是否存在
HEAD /secisland
==>200 - OK
==>{"statusCode":404,"error":"Not Found","message":"404 - Not Found"}
```



### 2.1.2 打开/关闭索引

打开关闭索引接口允许关闭一个打开的索引或者打开一个已经关闭的索引。关闭的索 引只能显示索引元数据信息,不能够进行读写操作。

```bash
# 打开/关闭索引的方式是{ 索引名 }/_close 或者/{ 索引名 }/_open
 
POST /secisland/_close
 
# 关闭后不能：搜索、索引文档、更新文档
PUT /secisland/_doc/1
{
  "logType": 1
}
 {
  "error" : {
    "root_cause" : [
      {
        "type" : "index_closed_exception",
        "reason" : "closed",
        "index_uuid" : "Qc6-xbtPSw6vKysolY04sw",
        "index" : "secisland"
      }
    ],
    "type" : "index_closed_exception",
    "reason" : "closed",
    "index_uuid" : "Qc6-xbtPSw6vKysolY04sw",
    "index" : "secisland"
  },
  "status" : 400
}

```



#### **索引关闭的影响**

**1. 资源占用大幅减少**

- **内存释放**：字段数据、索引缓存被清除
- **磁盘IO**：不再有索引刷新、合并操作
- **文件句柄**：索引文件句柄被释放
- **集群负载**：分片分配、恢复等操作停止

**2. 操作限制**

- **❌ 不能**：搜索、索引文档、更新文档
- **❌ 不能**：执行聚合分析
- **❌ 不能**：修改映射和设置（需先打开）
- **✅ 可以**：查看索引元数据、设置、映射
- **✅ 可以**：删除关闭的索引
- **✅ 可以**：打开索引恢复使用



#### **如何判断索引状态**

**方法1：使用_cat API（最直观）**

```bash
GET /_cat/indices/secisland?v&h=index,status,health,pri,rep,docs.count,store.size

# 返回示例：
# index       status  health pri rep docs.count store.size
# test_index  close   -      3   2   0          0b
# open_index  open    green  3   2   1000000    1.2gb
```

**状态说明：**

- `status=open`：索引已打开
- `status=close`：索引已关闭
- `health=-`：关闭的索引没有健康状态

**方法2：查看索引详情**

```json
GET /secisland/_stats
==》
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_closed_exception",
        "reason" : "closed",
        "index_uuid" : "Qc6-xbtPSw6vKysolY04sw",
        "index" : "secisland"
      }
    ],
    "type" : "index_closed_exception",
    "reason" : "closed",
    "index_uuid" : "Qc6-xbtPSw6vKysolY04sw",
    "index" : "secisland"
  },
  "status" : 400
}
```

- **如果索引打开**：返回详细的统计信息
- **如果索引关闭**：返回错误 `index_closed_exception`

**方法3：使用集群状态API**

```json
GET /_cluster/state?filter_path=metadata.indices.secisland.state

# 返回示例：
{
  "metadata": {
    "indices": {
      "test_index": {
        "state": "close"  # 或 "open"
      }
    }
  }
}
```



### 2.1.3 kibana中类似head的索引管理

#### s**tack Management 模块**

Kibana 的图形化管理界面（**推荐用于日常监控**）：

**访问路径：**

```markdown
Kibana左侧菜单 → Stack Management → Index Management
```

**功能包括：**

- **索引列表**：查看所有索引的状态、文档数、大小等
- **索引详情**：点击索引名查看详细设置、映射、统计信息
- **索引生命周期管理**：配置 ILM 策略
- **模板管理**：查看和创建索引模板
- **数据流管理**：管理数据流（Elasticsearch 7.9+）

<img src="Elasticsearch技术解析与实战笔记.assets/image-20251229201059207.png" alt="image-20251229201059207" style="zoom:67%;" />



## 2.2 索引映射管理

```bash
# 更新索引映射
POST /secisland/_mapping/
{
  "properties": {
    "logType": {
      "type": "keyword"
    }
  }
}

#获取映射
GET /secisland/_mapping/
==>
{
  "secisland" : {
    "mappings" : {
      "dynamic_templates" : [],
      "properties" : {
        "logType" : {
          "type" : "text"
        }
      }
    }
  }
}


# 获取字段映射 获取文档字段接口允许你搜索一个或多个字段 host:port/{index}/{type}/_mapping/field/{field}
GET /secisland/_mapping/field/logType
==>
{
  "secisland" : {
    "mappings" : {
      "logType" : {
        "full_name" : "logType",
        "mapping" : {
          "logType" : {
            "type" : "text"
          }
        }
      }
    }
  }
}


```



**为什么字段映射API返回复杂结构？**

**原因1：支持嵌套字段和对象字段**

对于复杂结构，字段映射API能更好地展示层次关系：

```json
# 假设有这样的映射
{
  "user": {
    "type": "object",
    "properties": {
      "name": { "type": "text" },
      "address": {
        "type": "object", 
        "properties": {
          "city": { "type": "keyword" }
        }
      }
    }
  }
}

# 字段映射API返回
GET /index/_mapping/field/user.name
{
  "index": {
    "mappings": {
      "user.name": {
        "full_name": "user.name",     # 点分表示法
        "mapping": {
          "name": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

**原因2：保持API一致性**

字段映射API需要处理各种复杂场景，统一的结构更易于解析。



## 2.3 索引别名

Elasticsearch 的别名机制有点像数据库中的视 图。例如:为索引 test1 增加一个别名 alias1。

基本使用：

```bash
# 1. 为索引增加别名 为单个索引增加别名
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "test1",
        "alias": "alias1"
      }
    }
  ]
}

# 2. 删除别名
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "test1",
        "alias": "alias1"
      }
    }
  ]
}

# 3. 修改别名（先删后加）
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "test1",
        "alias": "alias1"
      }
    },
    {
      "add": {
        "index": "test1", 
        "alias": "alias2"
      }
    }
  ]
}
# 4. 一个别名关联多个索引
## 方式1：分别添加
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "test1",
        "alias": "alias1"
      }
    },
    {
      "add": {
        "index": "test2", 
        "alias": "alias1"
      }
    }
  ]
}
## 方式2：使用 indices 数组（推荐）
POST /_aliases
{
  "actions": [
    {
      "add": {
        "indices": ["test1", "test2"],
        "alias": "alias1"
      }
    }
  ]
}
## 方式3：使用通配符
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "test*",
        "alias": "all_test_indices"
      }
    }
  ]
}
```



## 2.4 索引配置

## 2.5 索引监控

## 2.6 状态管理

## 2.7 文档管理