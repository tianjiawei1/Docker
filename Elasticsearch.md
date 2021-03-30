# 第一章 Elasticsearch入门

## ElaStic Stack

kinana    --      数据探索与可视化分析

ElasticSearch  --   数据存储、查询与分析

Beants，Logstash  -- 数据收集与处理

## ETL 

Extract Transform Load

## 数据源多样

> 数据文件，如日志，Excel等；数据库，如MySQL，oracle等；http服务；网络数据；支持自定义拓展，无线可能
>
> 完备的数据分析工具集合
>
> - 搜索引擎
> - 日志分析
> - 指标分析

## ELK

> Kinana  Elasticsearch  Logstash

## Document

用户存储在es中的数据文档

Json Object ,由字段（Field）组成，常见数据类型如下：

- 字符串：text，keyword
- 数值型：long,integer,shrot,byte,double,float,half_float,scaled_float
- 布尔：boolean
- 日期：date
- 二进制：binary
- 范围类型：integer_range,float_range,long_range,double_range,date_range

每个文档有唯一的id表示

- 自行制定
- es自动生成

### Document MetaData

元数据，用于标注文档的相关信息

- _index：文档所在的索引名
- _type：文档所在的类型名
- _id：文档唯一id
- _uid：组合id，由 _type 和 _id组成（6.x_type不在起作用，同 _id一样）
- _source：文档的原始Json数据，可以从这里获取每个字段的内容
- _all：整合所有字段内容到该字段，默认禁用

### 文档 Document API 

es有专门的Document API

创建文档

指定id创建文档API 如下：

```json
PUT /index/type/id 

PUT /test_index/doc/1
{
"username":"alfred",
"age":1
}
```

查询文档

更新文档

删除文档

## 索引 Index

由具有相同字段的文档列表组成

索引中存储具有相同结构的文档（document）

- 每个索引都有自己的mapping定义，用于定义字段名和类型

一个集群可以由多个索引，比如：

- Nginx日志存储的时候可以按照日期每天生成一个索引来存储

## 节点 Node

一个Eleasticsearch 的运行实例，是集群的构成单元

## 集群 Cluster

由一个或多个节点组成，对外提供服务

## Rest API

Elasticsearch集群对外提供RESTful API

- REST - REpresentational State Transfer
- URI指定资源，如Index，Document 等
- Http Method 指明资源操作类型，如GET，POST，DELETE，PUT等

常用两种交互方式

- Curl命令行
- Kibana DevTools

## 索引API

es有专门的Index API，用于创建，更新，删除索引配置等

- 创建索引api如下

  ```markdown
  PUT  /test_index
  ```

- 查看现有索引

  ```markdown
  GET  _cat/indices
  ```

- 删除索引api

  ```markdown
  DELETE  /test_index
  ```

## 创建文档API

### 创建文档

指定id创建文档API

```java
PUT /test_index/doc/1   // index/type/id  
{
    "username": "alfred",
    "age": 1
}
//创建文档时，如果索引不存在，es会自动创建对应的index和type
```

不指定id创建文档

```java
POST  /test_index/doc
{
    "username": "alfred",
    "age": 1
}
```

### 查询文档

指定要查询的文档

```json
GET /test_index/doc/1
//_source 存储了文档的完整原始数据
{
    "_index": "test_index",
    "_type": "doc",
    "_id": "1",
    "_version": 1,
    "found": true,
    "_source": {
        "username": "alfred",
        "age": 1
    }
}
```

搜索所有文档，用到 _search

```json
GET /test_index/doc/_search
//查询语句，json格式，放在http body中发送到es
{
    "query": {
        "term": {
            "_id": "1"
        }
    }
}

//响应
{
    "took": 0, //查询耗时，单位ms
    "time_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "age": 0
    },
    "hits": {
        "total": 1, //符合条件的总文档数
        "max_source": 1,
        "hits": [ //返回的文档详情数据数组，默认前10个文档
            {
                "_index": "test_index", //索引名
                "_type": "doc",  
                "_id": "1",  //文档的id
                "_score": 1,  //文档的得分
                "_source": {  //文档详情
                    "username": "alfred",
                    "age": 1
                }
            }
        ]
    }
}
```

### 批量创建文档API

es允许一次创建多个文档，从而减少网络传输开销，提升写入sulv

endpoint 为 _bull如下

```json
POST _bull

{"index":{"_index":"test_index","_type":"doc","id":"3"}} //action_type:
{"username":"alfred","age":10}
{"delete":{"_index":"test_index","_type":"doc","id":"1"}}
{"update":{"_index":"test_index","_type":"doc","id":"2"}}
{"doc":{"age":20}}
```

### 批量查询文档API

es允许一次查询多个文档

- endpoint 为 _mget 

```json
GEt /_mget

{
    "docs": [
        {
            "_index": "test_index",
            "_type": "doc",
            "_id": "1" //指明要查询的文档id
        },
        {
            "_index": "test_index",
            "_type": "doc",
            "_id": "2"
        }
    ]
}
```

# 第二章 倒排索引与分词

## 正排索引

- 文档Id到文档内容、单词的关联关系

| 文档ID | 文档内容                         |
| ------ | -------------------------------- |
| 1      | eleasticsearch是最流行的搜索引擎 |
| 2      | PHP是世界最好的语言              |

## 倒排索引

- 单词到文档Id的关联关系

| 单词           | 文档ID列表 |
| -------------- | ---------- |
| eleasticsearch | 1          |
| 流行           | 1          |
| 搜索引擎       | 1,3        |

查询包含“搜索引擎”的文档

- 通过倒排索引获得“搜索引擎”对应的文档ID有1和3
- 通过正排索引查询1和3的完整内容
- 返回用户最终结果

### 组成

倒排索引是搜索引擎的核心，主要包含两部分

#### 单词词典（Term DIctionary）

记录所有文档的单词，一般都比较大

记录单词到倒排列表的关联信息

- 单词字典的实现一般是用 B+Tree方式（查询性能比较高）

#### 倒排列表（Posting List）

记录了单词对应的文档集合，有倒排索引项（Postint）组成

倒排索引项（Posting）主要包含如下信息：

- 文档Id，用于获取原始信息
- 单词频率（TF，Term Frequency），记录该单词在该文档中的出现次数，用于后续相关性分析
- 位置（Postion），记录单词在文档中的分词位置（多个），用于做词语搜索（Phrase Query）
- 偏移（Offset），记录单词在文档的开始和结束位置，用于做高亮显示

![](D:\软件\Typora\笔记\image\QQ图片20200601145708.png)

**es存储的是一个json格式的文档，其中包含多个字段，每个字段会有自己的倒排索引**。

## 分词

将文本转换成一系列单词（term or token）的过程，也可以叫做文本分析，在es里面称为 Analysis

例：

文本：elasticsearch是最流行的搜索引擎

分词结果：elastic search  ；流行；搜索引擎

### 分词器

分词器是es中专门处理分词的组件，英文为Analyzer，它的组成如下：

**Character Filters**

> 针对原始文本进行处理，比如去除html特殊标记符

- 在Tokenizer之前对原始文本进行处理，比如增加、删除或替换字符等

- 自带的如下：

  - HTML Strip去除html标签和转换html实体

  - Mapping进行字符替换操作

  - Pattern Replace 进行正则匹配替换

- 会影响后续tokenizer解析的position和offset信息

Character Filters测试时可以采用如下api：

```json
POST _analyze
{
    "tokenizer":"keyword", //keyword类型的tokenizer可以直接看到输出结果
    "char_filter":["html_strip"], //指明要使用的char_filter
    "text":"<p>I&apos;m so<b>happy</b>!</p>", //测试文本
}
```

**Tokenizer**

> 将原始文本按照一定规则切分为单词(term or token)

自带的如下：

- standard按照单词进行分割
- letter按照非字符类进行分割
- whitespace按照空格进行分割
- UAX URL Email 按照standard分割，但不会分割邮箱和url
- NGram和Edge NGram连词分割
- Path Hierarchy按照文件路径进行切割

Tokenizer测试时可以采用如下api：

```json
POST _analyze
{
    "tokenizer":"path_hierarchy", //指定要测试的tokenizer
    "text":"/one/two/thress" //测试文本
}
```

**Token Filters**

> 针对tokenizer处理的单词进行在加工，比如转小写、删除或新增等处理

自带的如下：

- lowercase将所有term转换为小写
- stop删除stop words
- NGram 和Edge NGram连词分割
- Synonym添加近义词的term

Filter测试时可以采用如下api：

```json
POST _analyze
{
    "tokenizer":"standard",
    "text":"a Hello,world!",
    "filter":[//指定要测试的filter
        "stop",
        "lowercase",
        {
            "type":"ngram",
            "min_gram":4,
            "max_gram":4
        }]
}
```

#### 分词器-调用顺序

![](D:\软件\Typora\笔记\image\QQ图片20200601145524.png)

#### Analyze API

es提供了一个测试分词的api接口，方便验证分词效果，endpoint是_analyze

- 直接制定analyzer进行测试

  ```json
  POST _analyze
  {
      "analyzer":"standard", //分词器(es默认分词器)
      "text":"hello world!" //测试文本
  }
  响应：
  {
      "tokens":[
          {
              "token":"hello",//分词结果
              "start_offset":0,//起始偏移
              "end_offset":5,//结束偏移
              "type":"<ALPHANUM>",
              "position":0  //分词位置        
          },
          {
              "token":"world",
              "start_offset":6,
              "end_offset":11,
              "type":"<ALPHANUM>",
              "position":1          
          }
      ]
  }
  ```

- 直接指定索引中的字段进行测试

  ```json
  POSt test_index/_analyze
  {
      "field":"username", //测试字段
      "text":"hello world!" //测试文本
  }
  ```

- 自定义分词器进行测试

  ```json
  POST _analyze
  {
      "tokenizer":"standard", 
      "filter":["lowercase"], //自定义analyzer
      "text":"hello world!"  //测试文本
  }
  ```

#### 自定义的分词器

es自带如下的分词器

> **Standard**

- 默认分词器
- 其组成如图，特性为：
  - 按词切分，支持多语言
  - 小写处理

![](D:\软件\Typora\笔记\image\QQ图片20200601152217.png)

> **Simple**

其组成如图，特性为：

- 按照非字母切分
- 小写处理

![](D:\软件\Typora\笔记\image\QQ图片20200601153322.png)

> **Whitespace**

其组成如图：特性为：

- 按照空格切分

![](D:\软件\Typora\笔记\image\QQ图片20200601153849.png)

> **Stop**

Stop Word 指语气助词等修饰性的词语，比如 the、an、的、这等等

其组成如图，特性为：

- 相比Simple Analyzer 多了stop word处理

![](D:\软件\Typora\笔记\image\QQ图片20200601154527.png)

> **Keyword**

其组成如图，特性为：

- 不分词，直接将输入作为一个单词输出

![](D:\软件\Typora\笔记\image\QQ图片20200601161636.png)

> **Pattern**

其组成如图，特性为：

- 通过正则表达式自定义分隔符
- 默认是\W+，即非字词的符号作为分隔符

![](D:\软件\Typora\笔记\image\QQ图片20200601161918.png)

> **Language**

- 提供了30+常见语言的分词器
- arabic，armenian，basque，bengali，brazilian，bulgarian，catalan，cjk，czech，danish，dutch，english

### 中文分词

难点

- 中文分词指的是将一个汉字序列切分为一个一个单独的词。在英文中，单词之间以空格作为自然分节符，汉语中词没有一个形式上的分界符。

- 上下文不同，分词结果迥异，比如交叉歧义问题，比如下面两种分词都合理

  > 乒乓球拍/卖/完了
  >
  > 乒乓球/拍卖/完了

#### 常见分词系统

> **IK**

- 实现中英文单词的切分，支持ik_smart、ik_maxword等模式

- 可自定义词库，支持热更新分词词典

  > https://github.com/medcl/elasticsearch-analysis-ik

> **jieba**

- python 中最流行的分词系统，支持分词和♀标注

- 支持繁体分词、自定义词典、并行分词等

  > https://github.com/singlee/elasticsearch-jieba-plugin

#### 基于自然语言处理的分词系统

> hanlp

- 由一系列模型与算法组成的java工具包，目标是普及自然语言处理在生产环境的应用
- http://github.com/hankcs/HanLP

> THULAC

- THU Lexical Analyzer for chinese，由清华大学自然语言处理与社会人文计算实验室研制推出的一套中文词法分析工具包，具有中文分词和词性标注功能
- https://github.com/microbun/elasticsearch-thulac-plugin

### 自定义分词

自定义分词需要在索引的配置中设定，如下图所示：

![](D:\软件\Typora\笔记\image\1602224726.jpg)

![](D:\软件\Typora\笔记\image\1602228013.jpg)

![](D:\软件\Typora\笔记\image\1602228155.jpg)

### 分词使用说明

1.创建或更新文档时（index Time），会对相应的文档进行分词处理

2.查询时（Search Time），会对查询语句进行分词

2. 1查询的时候通过analyzer指定分词器

   2.2通过index mapping设置search_analyzer

3.索引时分词是通过配置Index Mapping中每个字段的analyzer属性实现的，如下

      不指定分词时，使用默认standard

一般不需要特别指定查询时分词器，直接使用索引时分词器即可，否则会出现无法匹配的情况

#### 分词的使用建议

1，明确字段是否需要分词，不需要分词的字段就将type设置为keyword，可以节省空间和提供写性能

2，善用+annlyze API，查看文档的具体分词结果

3，动手测试

# 第三章 Mapping设置

类似数据库中的表结构定义，主要作用如下：

- 定义Index下的字段名（Field Name）
- 定义字段的类型，比如数值型，字符串型，布尔型
- 定义倒排索引相关的配置，比如是否索引，记录position等



## 自定义Mapping

Mapping中的字段类型一旦设定后，禁止直接修改，原因如下：

- Lucene实现的倒排索引生成后不允许修改

重新建立新的索引，然后做reindex操作

允许新增字段，通过dynamic参数来控制字段的新增

- true（默认）允许自动新增字段
- false 不允许自动新增字段，但是文档可以正常写入，但无法对字段进行查询等操作
- strict 文档不能写入，报错

## Mapping示例

> ##### copy_to

- 将该字段的值复制到目标字段，实现类似_all的作用
- 不会出现在 _source中，只用来搜索

> index

- 控制当前字段是否索引，默认为true，即记录索引，false不记录，即不可搜索

> index_options 

用于控制倒排索引记录的内容，有如下4中配置

- docs只记录doc id
- freqs记录doc id 和term frequencies
- positions记录doc id 、term frequencies 和term position
- offsets记录doc id 、term frequencies 、term position 和character offsets

text类型默认配置为positions，其他默认为docs

记录内容越多，占用空间越大

> null_value

- 当字段遇到null值时的处理策略，默认为null，即空值，此时es会忽略该值。可以通过该特性设定字段的默认值

![](D:\软件\Typora\笔记\image\1602236433.jpg)

## 数据类型

- 字符串型 text （可分词）、keyword（不可分词）
- 数值型 long、integer、short、byte、double、float、half_float、scaled_float
- 日期类型 date
- 布尔类型 boolean
- 二进制类型 binary
- 范围类型 integer_range、float_range、long_range、double_range、date_range

### 负责数据类型

- 数组类型 array
- 对象类型 object
- 嵌套类型 nested object

### 专用类型

- 记录ip地址 ip
- 实现自动补全 completion
- 记录分词数 token_count
- 记录字符串hash值 murmur3
- percolator
- join

### 多字段特性 multi-fields

- 允许对同一个字段采用不同的配置，比如分词，常见例子如对人名实现拼音搜索，只需要在人名中新增一个子字段为pinyin即可

# 第四章Search API介绍



# 第五章 分布式特性介绍



# 第六章 深入了解Search 的运行机制



# 第七章 聚合分析入门



# 第八章 数据建模



# 第九章 集群调优建议

