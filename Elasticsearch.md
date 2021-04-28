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

1. 1查询的时候通过analyzer指定分词器

   2.2通过index mapping设置search_analyzer

3.索引时分词是通过配置Index Mapping中每个字段的analyzer属性实现的，如下

```
  不指定分词时，使用默认standard
```

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

  > arrays：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/array.html

## dynamic mapping

**es可以自动识别文档字段类型**，从而降低用户使用成本，如下所示：

```shell
PUT /test_index/doc/1
{
    "username":"alfred";
    "age":1
}

GET /test_index/_mapping

{"test_index":{
    "mapping":{
        "doc":{
            "properties":{
                "age":{
                    "type":"long"
                },
                "username":{
                    "type":"text",
                    "fields":{
                        "keyword":{
                            "type":"keyword",
                            #es自动识别，age为long类型，username为text类型
                            "ignore_abover":256
                        }
                    }
                }
            }
        }
    }
}}
```

es是依靠JSON文档的字段类型来实现自动识别字段类型，支持的类型如下：

| JSON类型 | es类型                                                       |
| -------- | ------------------------------------------------------------ |
| null     | 忽略                                                         |
| boolean  | boolean                                                      |
| 浮点类型 | float                                                        |
| 整数     | long                                                         |
| object   | object                                                       |
| array    | 由第一个非null值得类型决定                                   |
| string   | 匹配为日期则设为date类型（默认开启）；                                                                       匹配为数字的话设为float或long类型（默认关闭）；                                                       设为text类型并附带keyword的子字段 |

**日期的自动识别可以自行配置日期格式**，以满足各种需求

- 默认是["strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]
- strict_date_optional_time是ISO datetime的格式，完整格式类似下面：

```markdown
YYYY-MM-DDThh:mm:ssTZD(eg 1997-07-16T19:20:30+01:00)
```

- dynamic_date_formats可以自定义日期类型
- date_detection可以关闭日期自动识别的机制

```shell
PUT my_index
{
    "mapping":{
        "my_type":{
           "dynamic_date_formats": ["MM/dd/yyyy"]
        }
    } 
}

PUT my_index/my_type/1
{
    "create_date":"09/25/2015"
}

#关闭日期自动识别机制
PUT my_index
{
    "mapping":{
        "my_type":{
           "date_detection":false
        }
    } 
}
```

字符串是数字时，默认不会自动识别为整型，应为字符串中出现数字时完全合理的

- numeric_detection可以开启字符串中数字的自动识别，如下所示：

```powershell
PUT my_index
{
    "mapping":{
        "my_type":{
           "numeric_detection": true
        }
    } 
}

PUT my_index/my_type/1
{
    "my_float":"1.0",
    "my_integer":"1"
}

{"my_index1":{
    "mapping":{
        "my_type":{
            "numeric_detection":true,
            "properties":{
                "my_float":{
                    "type":"float"
                },
                "my_integer":{
                    "type":"long"              
                }
            }
        }
    }
}
}
```

## Dynamic Templates

允许根据es自动识别的数据类型、字段名等来动态设定字段类型，可以实现如下效果：

- 所有字符串类型都设定为keyword类型，即默认不分词
- 所有message开头的字段都设定为text类型，即分词
- 所有以long_ 开头的字段都设定为long类型
- 所有自动匹配为double类型的都设定为float类型，以节省空间

```shell
PUT test_index

{
    "mappings":{
        "doc":{
        #数组，可以指定多个匹配规则
            "dynamic_templates":[{
            #template 的名称
                "strings":{
                #匹配规则
                    "match_mapping_type":"string",
                    #设置mapping信息
                    "mapping":{
                        "type":"keyword"
                    }
                }
            }]
        }
    }
}
```

匹配规则一般有如下几个参数：

- match_mapping_type 匹配es自动识别的字段类型，如boolean，long，string等
- match，numatch 匹配字段名
- patch_match，patch_unmatch 匹配路径

**字符串默认使用keyword类型**

- es默认会为字符串设置为text类型，并增加一个keyword的子字段

**以message开头的字段都设置为text类型**

```shell
PUT test_index
{
    "mappings":{
        "doc":{
            "dynamic_templates":[{
                "message_as_text":{
                    "match_mapping_type":"string",
                    "match":"message",
                    "mapping":{
                        "type":"text"
                    }
                }
            }]
        }
    }
}
```

**double类型设定为float，节省空间**

```shell
PUT test_index
{
    "mappings":{
        "doc":{
            "dynamic_templates":[{
                "double_as_float":{
                    "match_mapping_type":"double",
                    "mapping":{
                        "type":"float"
                    }
                }
            }]
        }
    }
}
```

## 建议

自定义Mapping的操作步骤如下：

1. 写入一条文档到es的临时索引中，获取es自动生成的mapping
2. 修改步骤1得到的mapping，自定义相关配置
3. 使用步骤2的mapping创建实际所需索引

## 索引模板

索引模板（Index Template），主要用于在新建索引时自动应用预先设定的配置，简化索引创建的操作步骤

- 可以设定索引的配置和mapping
- 可以有多个模板，根据order设置，order大的覆盖小的配置

```shell
PUT _template/test_template   #template 名称
{
#索引名称
    "index_patterns":["te*","bar"],
    #order 顺序位置
    "order":0,
    #索引的配置
    "setting":{
        "number_of_shards":1
    },
    "mappings":{
        "doc":{
            "_source":{
                "enabled":false
            },
            "properties":{
                "name":{
                    "type":"keyword"
                }
            }
        }
    }
}

#创建索引并查看配置
PUT test_index
GET test_index

#获取与删除的API如下：
GET _template
GET _template/test_template
DELETE _template/test_template
```

# 第四章Search API介绍

实现对es中存储的数据进行查询分析，endpoint为 `_search`，如下所示

```markdown
GET /_search
GET /my_index/_search
GET /my_index1,my_index2/_search
GET /my_*/_search 指定索引查询，可以一次查询多个
```

查询主要有两种形式

- URI Search
  - 操作简便，方便通过命令行测试
  - 仅包含部分查询语法
- Request Body Search
  - es提供的完备查询语法Query DSL（Domain Specific Language）

```shell
如：
URI search
GET /my_index/_search?q=user:alfred

Request Body Search 
GET /my_index/_search
{
    "query":{
        "term":{"user":"alfred"}
    }
}
```

## URI Search

通过 URL query 参数来实现搜索，常用参数如下：

- q 指定查询的语句，语法为 Query String Syntax
- df q 中不指定字段是默认查询的字段，如果不指定，es会查询所有字段
- sort 排序
- timeout 指定超时时间，默认不超时
- from，size用于分页

```shell
GET /my_index/_search?q=alfred&df=user&sort=age:asc&from=4&size=10&timeout=1s
#查询user字段包含alfred的文档，结果展示age升序排列，返回地5~10个文档，如果超过1s没有结束，则以超时结束
```

### Query String Syntax

**term 与phrase**

- alfred way 等效于 alfred OR way
- "alfred way" 词语查询，要求先后查询

**泛查询**

- alfred 等效于在所有字段去匹配该term

**指定字段**

- name:alfred

**Group 分组设定，使用括号指定匹配的规则**

- (quick OR brown) AND fox   (先判断前面条件在判断后面条件)
- status:(active OR pending) title:(full text search)

**布尔操作符**

- AND (&&)，OR (||)，NOT (!)
  - name：(tom NOT)
  - 注意大写，不能小写
- `+ -`分别对应must 和must not
  - name : (tom +lee -alfred)
  - name : ((lee && !alfred) || (tom && lee && !alfred))
  - +在URL中会被解析为空格，要是用encode后的结果才可以，为`%2B`

**范围查询，支持数值和日期**

- 区间写法，闭区间用`[]`，开区间用`{}`

```markdown
age:[1 TO 10] 意为 1<= age <= 10
age:[1 TO 10} 意为 1<= age < 10
age:[1 TO ] 意为 1<= age 
age:[* TO 10] 意为 age <= 10
```

- 算数符号写法

```markdown
age : >=1
age : (>=1 && <=10) 或者 age(+>=1 +<=10)
```

**通配符**

- `?`代表1个字符，`*`代表0或多个字符

```markdown
name : t?m
name : tom*
name : t*m
```

- 通配符匹配执行效率低，且占用较多内存，不建议使用
- 如无特殊需求，不要将`？/*` 放在最前面

**正则表达式匹配**

```markdown
name : /[mb]oat/
```

**模糊匹配 fuzzy query**

```markdown
name : roam ~1
匹配与roam差1个character的词，比如foam roams 等
```

**近似度查询 proximity search**

```markdown
"fox quick" ~5
以term为单位进行差异比较，比如"quick fox" "quick brown fox"都会被匹配
```

### Request Body Search

将查询语句通过http request body 发送到es，主要包含如下参数

- query 符合Query DSL 语法的查询语句
- from，size
- timeout
- sort

```yaml
GET /my_index/_search
{
    "query":{
        "term":{"user":"alfred"}
    }
}
```

基于JSON定义的查询语言，主要包含如下两种类型：

- 字段类查询

> 如term，match，range等，只针对某一个字段进行查询

- 复合查询

> 如bool查询等，包含一个或多个字段类查询或者复合查询语句

字段类查询主要包括以下两类：

- 全文匹配

> 针对text类型的字段进行全文检索，会对查询语句先进行分词处理，如match，match_phrase等query类型

- 单词匹配

> 不会对查询语句做分词处理，直接去匹配字段的倒排索引，如term，terms，range等query类型

**Match Query**

对字段做全文检索，最基本和常用的查询类型，API示例如下：

```json
GET test_search_index/_search
{"query":{
    #关键词
    "match":{
      #字段名  ；待查询的语句Query Clause
      "username":"alfred way"  
    }
}
}
```

流程：

![](D:\Typora\笔记\image\图片36.png)

## 相关性算分

相关性算分是指文档与查询语句间的相关度，英文为relevance

- 通过倒排索引可以获取与查询语句相匹配的文档列表，那么如何将最符合用户查询需求的文档放到前列呢？
- 本质是一个排序问题，排序的依据是相关性算分

| 单词   | 文档ID列表 |
| ------ | ---------- |
| alfred | 1,2        |
| way    | 1          |

相关性算分的几个重要概念如下：

- Term Frequency(TF) 词频，即单词在该文档中出现的次数。词频越高，相关度越高
- Document Frequency(DF) 文档频率，即单词出现的文档数
- Inverse Document Frequency(IDF) 逆向文档频率，与文档频率相反，简单理解为1/DF。即单词出现的文档数越少，相关度越高
- Field-length Norm 文档越短，相关性越高

ES目前主要有两个相关性算分模型如下：

- TF/IDF 模型
- BM25 模型 5.x之后的默认模型

### TF/IDF 模型

是Lucene的经典模型，其计算公式如下：

![](D:\Typora\笔记\image\图片37.png)

可以通过**explain**参数来查询看具体的计算方法，但要注意：

- es的算分是按照`shard`进行的，即shard的分数计算是相互独立的，所以在使用explain的时候注意分片数
- 可以通过设置索引的分片数为1来避免这个问题

```shell
GET test_search_index/_search
{
    "explain":true,
    "query":{
        "match":{
            "username":"alfred way"
        }
    }
}

PUT test_search_index
{
    "settings":{
        "index":{
            "number_of_shards":"1"
        }
    }
}
```

BM25 模型

BM25模型中BM指Best Match，25指迭代了25次才计算方法，是针对TF/IDF的一个优化，其计算公式如下：

![](D:\Typora\笔记\image\图片38.PNG)

BM25相比TF/IDF的一大优化是降低了tf在过大时的权重

![](D:\Typora\笔记\image\图片39.PNG)



通过minimum_should_match 参数可以控制需要匹配的单词数

```json
GET test_search_index/_search
{
    "query":{
        "match":{
            "username":{
                "query":"alfred way",
                "minimum_should_match":2
            }
        }
    }
}
```

## Query API

### Match Phrase Query

对字段作检索，有顺序要求，API示例如下：

```json
GET test_search_index/_search
{
    "query":{
        "match_phrase":{
            "job":"java engineer"
        }
    }
}
```

通过slop参数可以控制单词间的间隔

```json
GET test_search_index/_search
{
    "query":{
        "match_phrase":{
            "job":"java engineer",
            "slop":"1"
        }
    }
}
```

### Query String Query

类似于URI Search中q参数查询

```json
GET test_search_index/_search
{
    "query":{
        "query_string":{
            "default_field":"username",
            "query":"alfred AND way"
        }
    }
}

{
    "query":{
        "query_string":{
            #指明默认查询的字段名
            "field":["username","job"],
            "query":"alfred OR (java AND ruby)"
        }
    }
}

等同于
{
    "query":{
        "simple_query_string":{
            #指明默认查询的字段名
            "query":"(job:alfred | username:alfred)(job:java |username:java) +(job:ruby | username:ruby)"
        }
    }
}
```

### Simple Query String query

类似Query String，但是会忽略错误的查询语法，并且仅支持部分查询语法

其常用的逻辑符号如下，不能使用AND，OR，NOT等关键词：

```markdown
+ 代指AND
| 代指OR
- 代指NOT
```

```json
GET test_search_index/_search
{
    "query":{
        "simple_query_string":{          
            "query":"alfred + way",
            "fields":["username"]
        }
    }
}
```

### Terms Query

一次传入多个单词进行查询，如下所示：

```json
GET test_search_index/_search
{
    "query":{
        "terms":{          
            "username":["alfred","way"]
        }
    }
}
```

Range Query

范围查询主要针对数值和日期类型，如下所示：

```json
GET test_search_index/_search
{
    "query":{
        "range":{          
            "age":{
                "gte":10,
                "lte":20            
            }
        }
    }
}

#比较关键词
#gt - greater than
#gte - greater than or equal to
#lt - less than
#lte - less than or equal to

#针对日期做查询如下所示：
GET test_search_index/_search
{
    "query":{
        "range":{          
            "birth":{
                "gte":"1990-01-01"
            }
        }
    }
}

{
    "query":{
        "range":{          
            "birth":{
                #Date Math
                "gte":"now-20y"
            }
        }
    }
}
```

针对日期提供一种更友好的计算方式，格式如下：

```markdown
now - 1d
基准日期，也可以是具体的日期，比如2018-01-01，使用具体日期的时候要用||隔离

计算公式，主要有如下3种：
+1h - 加1个小时
-1d - 减1天
/d  - 将时间舍入到天

单位主要有如下几种：
- y - years
- M - months
- w - weeks
- d - days
- h - hours
- m - minutes
- s - seconds
```

假如now为2018-01-02 12:00:00

| 计算公式            | 实际结果            |
| ------------------- | ------------------- |
| now + 1h            | 2018-01-02 13:00:00 |
| now - 1h            | 2018-01-02 11:00:00 |
| now - 1h/d          | 2018-01-02 00:00:00 |
| 2016-01-01\|\|+1M/d | 2016-02-01 00:00:00 |

## Query DSL - 复合查询

复合查询是指包含字段类查询或复合查询的类型，主要包括以下几类：

- constant_score query

该查询将内部的查询结果文档得分都设定为1或者boost的值

> 多用于结合bool查询实现自定义得分

```json
GET test_search_index/_search
{
    "query":{
        #关键词
        "constant_score":{     
            #只能有一个
            "filter":{
                "match":{
                    "username":"alfred"
                }
            }
        }
    }
}
```

- bool query

布尔查询由一个或多个布尔字句组成，主要包含如下4个：

| filter   | 只过滤符合条件的文档，不计算相关性得分         |
| -------- | ---------------------------------------------- |
| must     | 文档必须符合must中所有条件，会影响相关性得分   |
| must_not | 文档必须不符合 must_not 中的所有条件           |
| should   | 文档可以符合 should 中的条件，会影响相关性得分 |

查询的API如下所示：

```markdown
GET test_search_index/_search
{
    "query":{
        #关键词
        "bool":{
        #支持数组
           "must":[{}],
           "must_not":[{}],
           "should":[{}],
           "filter":[{}]
        }
    }
}
```

> **Filter** 查询只过滤符合条件的文啊个，不会进行相关性算分
>
> - es针对filter会有只能缓存，因此其执行效率很高
> - 做简单匹配查询且不考虑算分时，推荐使用filter替代query
>
> ```json
> GET test_search_index/_search
> {
> 	"query": {
> 		"bool": {
> 			"filter": [{
> 				"term": {
> 					"username": "alfred"
> 				}
> 			}]
> 		}
> 	}
> }
> ```
>
> **must**
>
> ```json
> GET test_search_index/_search
> #查询username包含alfred并且job包含specialist关键词的文档列表
> {
> 	"query": {
> 		"bool": {
> 			"must": [
>                 #两个 match query文档最终的得分为这两个查询的得分加和
>                 {
> 				"match": {
> 					"username": "alfred"
> 				}
>             },{
>                 "match":{
>                     "job":"specialist"
>                 }
>             }]
> 		}
> 	}
> }
> ```
>
> **must_not**
>
> ```json
> GET test_search_index/_search
> #查询job中包含Java关键词但不包含ruby关键词的文档列表
> {
> 	"query": {
> 		"bool": {
> 			"must": [
>                 {
> 				"match": {
> 					"job": "java"
> 				}
>             }],
>             "must_not": [
>                 {
> 				"match": {
> 					"job": "ruby"
> 				}
>             }]
> 		}
> 	}
> }
> ```
>
> **should**
>
> - bool查询中只包含should，不包含must查询
>
>   只包含should时，文档必须满足至少一个条件：`minimum_should_match`可以控制瞒住条件的个数或者百分比
>
> ```json
> GET test_search_index/_search
> {
> 	"query": {
> 		"bool": {
> 			"should": [
>                 {"term": {"job": "java"}},
>                 {"term": {"job": "ruby"}},
>                 {"term": {"job": "specialist"}},
>             }],
>             #至少满足2个条件
>             "minimum_should_match":2
> 		}
> 	}
> }
> ```
>
> - bool查询中同时包含should和must查询
>
>   同时满足should和must时，文档不必满足should中的条件，但是如果满足条件，会增加相关性得分
>
> ```json
> GET test_search_index/_search
> {
> 	"query": {
> 		"bool": {
>             #查询username包含alfred的文档，同时将job宝含ruby的文档排在前面
>             "must":[{"term":{"username":"alfred"}}],
> 			"should":[{"term":{"job": "ruby"}}],         
> 		}
> 	}
> }
> ```

- dis_max query
- function_score query
- boosting query

## **Query Context VS Filter Context**

当一个查询语句位于Query或者Filter上下文时，es执行的结果会不同，对比如下：

| 上下文类型 | 执行类型                                                   | 使用方式                                           |
| ---------- | ---------------------------------------------------------- | -------------------------------------------------- |
| Query      | 查找与查询语句最匹配的文档，对所有文档进行相关性算分并排序 | query；bool中的must和should                        |
| Filter     | 查找与查询语句相匹配的文档                                 | bool中的filter与must_not；constant_score中的filter |

如：

```json
GET test_search_index/_search
{
	"query": {
		"bool": {
            #must下的时query上下文，会进行相关性算分
            "must":[
            {"match":{"title":"Search"}},
            {"match":{"content":"Elasticsearch"}}
            ],
#filter下的是filter上下文，不会影响算分，只会过滤符合条件的文档
			"filter":[
                {"term":{"status": "published"}},
                {"term":{"publish_date": {"gte":"2015-01-01"}}}
                ],         
		}
	}
}
```

## Count API

获取符合条件的文档数，endpoint为 `_count`

```json
GET test_search_index/_count
{
	"query": {
		"match": {
			"username": "alfred"
		}
	}
}

{
	"count": 3,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	}
}
```

## Source Filtering

过滤返回结果中`_source`中的字段，主要有如下几种方式：

```json
#url参数
GET test_search_index/_search?_source=username
#不返回_source
GET test_search_index/_search
{
  "_source":false
}
#返回部分字段
GET test_search_index/_search
{
  "_source":["useaname","age"]
}
#返回部分字段
GET test_search_index/_search
{
    "_source":{
        "includes":"*i*",
        "excludes":"birth"
    }
}
```

# 第五章 分布式特性介绍

- es支持集群模式，是一个分布式系统，其好处主要有两个：

> 增大系统容量，如内存、磁盘、使得es集群可以支持PB级的数据
>
> 提高系统可用性，即使部分机电停止服务，整个集群依然可以正常服务

- es集群由多个es实例组成

> 不容集群通过集群名字来区分，可通过`cluster.name`进行修改，默认为`elasticsearch`
>
> 每个es实例本质上是一个jvm进程，且有自己的名字，通过`node.name`进行修改

## cerebro安装与运行

地址如下：https://github.com/lmenezes/cerebro

**启动一个节点**

运行如下命令可以启动一个es节点实例

> bin/elasticsearch -E clueter.name=my_cluster -E node.name=node1 -E http:port=5200 -d

**Cluster State**

es集群相关的数据成为`cluster state`，主要记录如下信息：

- 节点信息，比如节点名称、链接地址等
- 索引信息，比如索引名称，配置等

**Mater Node**

- 可以修改`cluster state`的节点成为master节点，一个集群**只能有一个**
- cluster state 存储在每个节点上，master维护最新版本并同步给其他节点
- master节点是通过集群总所有节点选举产生的，可以被选举的节点成为`master-eligible节点`，相关配置如下：

> node.master:true

![](D:\Typora\笔记\image\图片40.jpg)

**创建一个索引**

通过如下API创建一个索引：

> PUT test_index

![](D:\Typora\笔记\image\图片41.jpg)

**Coordinating Node**

处理请求的节点即为`coordinating 节点`，该节点为所有节点的默认角色，不能取消

- 路由请求到正确的节点处理，比如创建索引的请求到master节点

![](D:\Typora\笔记\image\图片42.jpg)

**Data Node**

存储数据的节点即为`data节点`，默认节点都是data类型，相关配置如下：

> node.data:true

![](D:\Typora\笔记\image\图片43.jpg)

### 单点问题

如果node1停止服务，集群停止服务，这种情况下课新增节点解决

**新增一个节点**

运行如下命令可以启动一个es节点实例

> bin/elasticsearch -E clueter.name=my_cluster -E node.name=node2 -E http:port=5300 -d

### 提高系统可用性

**服务可用性**

- 2个节点的情况下，允许其中1个节点停止服务

**数据可用性**

- 引入副本（Replication）解决
- 每个节点上都有完备的数据

------

**副本**

如下图所示，node2上是test_index的副本

![](D:\Typora\笔记\image\图片44.jpg)

------

#### 增大系统容量

如何将数据分布于所有节点上？

- 引入分片（Shard）解决问题

分片是es支持PB级数据的基石

- 分片存储了部分数据，可以分布于任意节点上
- 分片数在索引创建时指定且后续不允许在更改，默认为5个
- 分片有主分片和副本分片之分，以实现数据的高可用
- 副本分片的数据由主分片同步，可以有多个，从而提高读取的吞吐量

#### **分片**

下图演示的是3个节点的集群中test_index的分片分布情况，创建时我们指定了3个分片和1个副本，api如下所示

```json
PUT test_index
{
    "settings":{
        "number_of_shards":3,
        "number_of_replicas":1
    }
}
```

![](D:\Typora\笔记\image\图片45.jpg)

此时增加节点是否能提高test_index的数据容量？

> 不能，因为只有3个分片，已经分布在3台节点上，新增的节点无法利用。

此时增加副本数是否能提高test_index的读取吞吐量？

> 不能，因为新增的副本也是在这3个节点上，还是利用了同样得到资源，如果要增加吞吐量，还需要新增节点。

![](D:\Typora\笔记\image\图片46.jpg)

分片数的设定很重要，需要提前规划好

- 过小会导致后续无法通过增加节点实现水品扩容
- 过大会导致一个节点上分布过多分片，造成资源浪费，同时会影响查询性能

### **Cluster Health**

通过如下Api可以查看集群健康状况，包括以下三种：

- green 健康状态，指所有主副分片都正常分配
- yellow 指所有主分片都正常，但是有副本分片未正常分配
- red 有主分片未分配

```json
GET _cluster/health
```

#### 故障转移

集群由3个节点组成，如下所示，此时集群状态是green

![](D:\Typora\笔记\image\图片47.jpg)

node1所在机器宕机导致服务 终止，此时集群会如何处理？

1. node2和node3发现node1无法响应一段时间后会发起master选举，必粗这里选择node2为master节点。此时由于主分片P0下线，集群状态变为Red。

![](D:\Typora\笔记\image\图片48.jpg)

1. node2发现主分片P0未分配，将P0提升为主分片。此时由于所有主分片都正常分配，集群状态变为Yellow。

![](D:\Typora\笔记\image\图片49.png)

1. node2为P0和P1生成新的副本，集群状态变为Green

![](D:\Typora\笔记\image\图片50.jpg)

### 文档分布式存储

文档最终会存储在分片上，如下图所示：

- Document1 最终存储在分片P1上

![](D:\Typora\笔记\image\图片51.jpg)

Document1是如何存储到分片P1的，选择P1的依据是什么？

- 需要文档到分片的映射算法

目的

- 使得文档均匀分布在所有分片上，以充分利用资源

算法

- 随机选择或者round-robin算法？

> 不可取，因为需要维护文档到分片的映射关系成本巨大

- 根据文档值实时计算对应的分片

#### 文档到分片的映射算法

es通过如下的工时计算文档对应的分片

> shard=hash(routing)%number_of_primary_shards
>
> hash算法保证可以将数据均匀的分散在分片中
>
> routing是一个关键参数，默认是文档ID，也可以自行指定

**文档创建的流程**

![](D:\Typora\笔记\image\图片52.jpg)

**文档读取流程**

![](D:\Typora\笔记\image\图片53.jpg)

**文档批量创建的流程**

![](D:\Typora\笔记\image\图片54.jpg)

**文档批量读取的流程**

![](D:\Typora\笔记\image\图片55.jpg)

### 脑裂问题

脑裂问题，英文为split-brain，是分布式系统中得到经典网络问题，如下图所示：

- 3个节点组成的集群，突然node1的网络和其他两个节点中断
  - node2和node3会重新选举master，比如node2成为了新master，此时会更新cluster state
  - node1自己组成集群后，也会更新cluster state

![](D:\Typora\笔记\image\图片56.jpg)

- 同一个集群有两个master，而且维护不同的cluster state，网络恢复后无法选择正确的master

![](D:\Typora\笔记\image\图片57.jpg)

解决方案：

仅在可选举`master-eligible`节点数大于等于`quorum`时才可以进行master选举

- quorum = master-eligible 节点数/2 + 1，例如3个master-eligible节点时，quorum为2。
- 设定`discovery.zen.minimum_master_nodes`为quorum即可避免脑裂

![](D:\Typora\笔记\image\图片58.jpg)

### 倒排索引的不可变更

倒排索引一旦生成，不能更改

其好处如下：

1. 不用考虑并发写文件的问题，杜绝了锁机制带来的性能问题
2. 由于文件不再更改，可以充分利用文件系统缓存，秩序载入一次，只要内存足够，对该文件的读取都会从内存读取，性能高
3. 利于生成缓存数据
4. 利于对文件进行压缩存储，节省磁盘和内存存储空间

坏处：

1. 需要写入新文档时，必须重新构建倒排索引文件，然后替换老文件后，新文档才能被检索，导致文档实时性差

![](D:\Typora\笔记\image\图片59.jpg)

解决方案：

新文档直接生成新的倒排索引文件，查询的时候同时查询所有的倒排文件，然后做结果的汇总计算即可

![](D:\Typora\笔记\image\图片59.jpg)

### 文档搜索实时性

`Lucene`便是采用了这种方案，它构建的单个倒排索引成为segment，合在一起成为`Index`，与ES中的Index概念不同。ES中的一个`Shard`对应一个`Lucene Index`。

Lucene会有一个专门的文件来记录所有的segment信息，称为`commit point`

![](D:\Typora\笔记\image\图片60.jpg)

#### refresh

segment写入磁盘的过程依然很耗时，可以借助文件系统缓存的特性，现将segment在缓存中创建并开放查询来进一步提升实时性，该过程在es中被称为`refresh`。

在refresh之前文档会先存储在一个buffer中，refresh时将buffer中的所有文档清空，并生成segment

![](D:\Typora\笔记\image\图片61.jpg)

es默认每1秒执行一次refresh，因此文档的实时性被提高到1秒，这也是es被称为近实时（Near Real Time）的原因

![](D:\Typora\笔记\image\图片62.jpg)

refresh发生的时机主要有如下几种情况：

- 间隔时间达到时，通过`index.settings.refresh_interval`来设定，默认是1秒
- index.buffer占满时，其大小通过`indices.memory.index_buffer_size`设置，默认为jvm heap的10%，所有shard共享
- flush发生时也会发生refresh

#### translog

如果在内存中的segment还没有写入磁盘前发生了宕机，那么其中的文档就无法恢复了，如何解决这个问题？

> es引入`translog`机制。写入文档到`buffer`时，同时将该操作写入translog。
>
> translog文件会及时写入磁盘（fsync），6.x默认每个请求都会落盘，可以修改为每5秒写一次，这样风险辨识丢失5秒内的数据，相关配置为`index.translog.*`
>
> es启动时会检查translog文件，并从中恢复数据

![](D:\Typora\笔记\image\图片63.jpg)

#### flush

flush负责将内存中的segment写入磁盘，主要做如下工作：

- 将translog写入磁盘
- 将index buffer清空，其中的文档生成一个新的segment，相当于一个refresh操作
- 更新commit point 并写入磁盘
- 执行fsync操作，将内存中的segment写入磁盘
- 删除旧的translog文件

![](D:\Typora\笔记\image\图片64.jpg)

发生的时机主要有如下几种情况：

- 间隔时间达到时，默认是30分钟，5.x之前可以修改`index.translog.flush_threshold_period`修改，之后无法修改
- translog占满时，其大小可以通过`index.translog.flush_threshold_size`控制，默认是512mb，每个index有自己的translog

#### 删除与更新文档

segment一旦生成就不能更改，那么如果你要删除文档该如何操作？

- Lucene专门维护一个`.del`的文件，记录所有已经删除的文档，注意`.del`上记录的是文档在Lucene内部的ID
- 在查询结果返回前会过滤掉`.del`中的所有文档

更新文档如何进行呢？

- 首先删除文档，然后在创建新文档

#### 整体视角

ES Index 与Lucene Index的术语对照如下所示：

![](D:\Typora\笔记\image\图片65.jpg)

#### Segment Merging

- 随着segment的增多，由于一次查询的segment数增多，查询速度会变慢
- es 会定时在后台进行`segment merge`的操作，减少segment的数量
- 通过`force_merge api`可以手动强制做`segment merge`的操作

# 第六章 深入了解Search 的运行机制

Search执行的时候时机分两个步骤运作的

## Query 阶段

node3在接收到用户的search其你去后，会先进行Query阶段（此时是Coordinating Node角色）

> 2.node3在6个主副分片中随机选择3个分片，发送search request
>
> 3.被选中的3个分片会分别执行查询并排序，返回from+size个文档ID和排序值
>
> 4.node3整合3个分片返回的from+size个文档ID，根据排序值排序后选取from到from+size的文档ID

![](D:\Typora\笔记\image\图片66.jpg)

## Fetch 阶段

node3根据Query阶段获取的文档ID列表去对应的shard上获取文档详情数据

> 1.node3向相关的分片发送multi_get请求
>
> 2.3个分片返回文档详细数据
>
> 3.node3拼接返回的结果并返回给客户

![](D:\Typora\笔记\image\图片67.jpg)

**相关性算分问题**

- 相关性算分在shard与shard间是相互独立的，也就意味着同一个Term的IDF等值在不同shard上是不同的。文档的相关性算分和它所处的shard相关。
- 在文档数量不多时，会导致相关性算分严重不准的情况发生

解决思路有两个：

一、设置分片数为1个，从根本上排除问题，在文档数量不多的时候可以考虑该方案，比如百万到千万级别的文档数量

二、使用DFS Query-then-Fetch查询方式

## Query-Then-Fetch

DFS Query-then-Fetch 是在拿到所有文档后再重新完整的计算一次相关性算分，耗费更多的CPU和内存，执行性能也比较地下，一般不建议使用。使用方式如下：

```json
GET test_search_relevance/_search?search_type=dfs_query_then_fetch
{
 "query":{
    "match":{
        "name":"hello"
    }
 }
}
```

## 排序

es默认会采用相关性算分排序，用户可以通过设定sorting参数来自行设定排序规则

```json
GET test_search_index/_search
{
    #关键词
    "sort":{
        "birth":"desc"
    }
}

#按照多个字段排序分别按照出生日期倒排，得分倒排，文档内部ID倒排
{
    #关键词
    "sort":[{
        "birth":"desc"
    },{
        #指相关性得分
        "_score":"desc"
    },{
        #指文档内部ID，和索引的顺序相关（分片内部唯一）
        "_doc":"desc"
    }]
}
```

 按照字符串排序比较特殊，应为es有text和keyword两种类型，如下所示：

```json
GET test_search_index/_search
{
    "sort":{
        "username":"desc"
    }
}

{
    "error":{
        "root_cause":[
            {
                "type":"illegal_argument_exception",
                "reason":"Fielddata is disabled on text fields by deafult"
            }
        ]
    }
}
```

针对keyword类型排序，可以返回预期结果

```json
GET test_search_index/_search
{
    "sort":{
        "username.keyword":"desc"
    }
}
```

 排序的过程实质是对字段原始内容排序的过程，这个过程中**倒排索引无法发挥作用**，需要用到**正排索引**，也就是通过文档ID和字段可以快速得到字段原始内容。

es对此提供了2中实现方式：

> fielddata默认禁用
>
> doc values 默认启动，除了text类型

| 文档ID | 字段值 |
| ------ | ------ |
| 1      | 100    |
| 2      | 89     |
| 3      | 129    |

**FieldData vs DocValues**

| 对比     | FieldData                                          | DocValues                          |
| -------- | -------------------------------------------------- | ---------------------------------- |
| 创建时机 | 搜索时即时创建                                     | 索引时创建，与倒排索引创建时机一直 |
| 创建位置 | JVM Heap                                           | 磁盘                               |
| 优点     | 不会占用额外得到磁盘资源                           | 不会占用Heap内存                   |
| 缺点     | 文档过多时，即时创建会花过多时间，占用过多Heap内存 | 减慢索引的速度，占用二外的磁盘资源 |

**FieldData**

Fielddata默认是关闭，可以通过如API开启：

- 此时字符串是按照分词后的term排序，往往结果很难符合预期
- 一般是在对分词作聚合分析的时候开启

```json
PUT test_search_index/_mapping/doc
{
    "properties":{
        "username":{
            "type":"text",
            #可以随时开启和关闭
            "fielddata":true
        }
    }
}
```

**Doc Values**

Doc Values 默认是启动的，可以在创建索引的时候关闭：

- 如果后面要在开启doc values，需要做reindex操作

```json
PUT test_doc_values1/
{
    "mapping":{
        "properties":{
            "username":{
                "type":"keyword",
                #设置为false即可
                "doc_values":false
            }
        }
    }
}
```

**docvalue_fields**

可以通过该字段获取fielddata或者doc values中存储的内容

```json
GET test_search_index/_search
{
    #指明需要的字段
    "docvalue_fields":[
        "username",
        "username.keyword",
        "age"
    ]
}
```

## 分页与遍历

es提供了3中方式来解决分页与遍历的问题

### from/size

最常用的分页方案

- from 指明开始位置
- size 指明获取总数

```json
GEt test_search_index/_search
{
    "from":1,
    "size":2
}
```

深度分页时一个经典的问题：在数据分片存储的情况下如何获取前1000个文档？

1. 获取从990~1000的文档时，会在每个分片上都想获取1000个文档，然后再由Coordinating Node 去和所有分片的结果后再排序选取前1000个文档
2. 页数越深，处理文档越多，占用内存越多，耗时越长，精良避免深度分页，es通过`index.max_result_window`限定对多到`10000`条数据

![](D:\Typora\笔记\image\图片68.jpg)

### scroll

遍历文档集的API，以快照的方式来避免深度分页的问题

- 不能用来做实时搜索，因为数据不是实时的
- 尽量不要使用复杂的sort条件，使用_doc最搞笑
- 使用稍显复杂

第一步：需要发起1个scroll search，如下所示：

- es在收到该请求后会根据查询条件创建文档ID合集的快照

```json
GEt test_search_index/_search?scroll=5m #该scroll快照有效时间
{
  #指明每次scroll返回的文档数
  "size":1
}

{
    "_scroll_id":"DXF1Z==",
    ...
}   
```

第二步：调用scroll search的API，获取文档 集合，如下所示：

- 不断迭代调用知道返回htis.hits 数组为空时停止

```json
POST _search/scroll
{
    "sroll":"5m", #指明有效时间
    "scroll_id":"..." #上一步返回的ID
}

{
    "_scroll_id":"DXF1Z...",#下一次调用使用
    "took":35,
    ...
    "hits":{
    "total":6,
    "max_score":1,
    "hits":[...]
   }
}
```

过多的scroll调用会占用大量内存，可以通过clear api 删除过多的scroll快照：

```json
DELETE /_search/scroll
{
    "scroll_id":[
        "DXF1ZXJ5QW...",
        "DnF1ZXJ5VGhlbkZ..."
    ]
}

DElETE /_search/scroll/_all
```

### search_alfter

避免深度分页的性能问题，提供实时的下一页文档获取功能

- 缺点是不能使用from参数，即不能指定页数
- 只能下一页，不能上一页
- 使用简单

第一步：正常的搜索，弹药指定sort值，并保证唯一

第二步：使用上一步最后一个文档的sort值进行查询

```json
GET test_search_index/_search
{
    "size":1,
    "sort":{
        #要保障sort值唯一
        "age":"desc",
        "_id","desc"
    }
}

GET test_search_index/_search
{
    "size":1,
    "search_after":[28,"2"],
    "sort":{
        "age":"desc",
        "_id","desc"
    }
}

#response
{
    ...
	"hits": {
		"total": 6,
		"hits": [
         ...
          {
			"_index": "test_search_index",
			"_type": "doc",
			"_id": "4",
			"sort": ["28", "2"]
		  }
        ]
	}
}

```

如何避免深度分页问题？

- 通过唯一排序值定位将每次要处理的文档数都控制在size内

![](D:\Typora\笔记\image\图片69.png)

**应用场景**

| 类型         | 场景                                       |
| ------------ | ------------------------------------------ |
| From/Size    | 需要实时获取顶部的部分文档，且需要自由翻页 |
| Scroll       | 需要全部问答个，如导出所有数据的功能       |
| Search_After | 需要全部文旦个，不需要自由翻页             |

# 第七章 聚合分析入门

聚合分析，英文为Aggregation，是es除搜索功能外提供的针对es数据做统计分析的功能

- 功能丰富，提供Bucket、Metric。Pipeline等多种分析方式，可以满足大分部的分析需求
- 实时性高，所有的计算结果都说是即时返回的，而Hadoop等大数据系统一般都是T+1级别的

聚合分析作为search的一部分，api如下所示：

```json
GEt test_search_index/_search
{
    "size":0,
    #关键词，与query统计
    "aggs":{
    #自定义聚合名称
        "<aggregation_name>":{
            "<aggregation_type>":{
                <aggregation_body>
            }
          [,"aggs":{[<sub_aggregation>]+ }]? #子查询  
        }
        [,"<aggregation_name_2>":{...}]* #可以包含多个聚合分析
    }
}
```

为了便于理解，es将聚合分析主要分为如下4类：

- Bucket  分桶类型，类似SQL中的Group by语法
- Metric  指标分析类型，如计算最大值、最小值、平均值等等
- Pipeline 管道分析类型，基于上一级的聚合分析结果进行再分析
- Matrix 聚成分析类型

## Metric 聚合分析

主要分如下两类：

- 单值分析，只输出一个分析结果
  - min、max、avg、sum
  - cardinality
- 多指分析，输出多个分析结果
  - stats、extended stats
  - percentile、percentile rank
  - top hits

### min

返回数值类字段的最小值

```json
GEt test_search_index/_search
{
    #不需要返回文档列表
    "size":0,
    "aggs":{
        "min_age":{
            #关键词
            "min":{
               "field":"age" 
            }        
        }
    }
}
```

### max

返回数值类字段的最大值

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "max_age":{
            "max":{
               "field":"age" 
            }        
        }
    }
}
```

### Avg

返回数值类字段的平均值

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "avg_age":{
            "avg":{
               "field":"age" 
            }        
        }
    }
}
```

### sum

返回数值类字段的总和

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "sum_age":{
            "sum":{
               "field":"age" 
            }        
        }
    }
}
```

### **聚合分析**

一次返回多个聚合结果

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "min_age":{
            "min":{
               "field":"age" 
            }        
        },
         "max_age":{
            "max":{
               "field":"age" 
            }        
        },
         "avg_age":{
            "avg":{
               "field":"age" 
            }        
        },
         "sum_age":{
            "sum":{
               "field":"age" 
            }        
        }
    }
}
```

### Cardinality

Cardinality，意味集合的势，或者技术，是指不同数值的个数，类似SQL中的distinct count概念

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "count_of_job":{
            "cardinality":{
               "field":"job.keyword" 
            }        
        }
    }
}
```

### Extended Stats

对stats的扩展，包含了更多的统计数据，如方差、标准差等

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "stats_age":{
            "extended_stats":{
               "field":"age" 
            }        
        }
    }
}
```

### Percentile

百分位数统计

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "per_age":{
            "percentiles":{
               "field":"salary" 
            }        
        }
    }
}
```

### Top Hits

一般用于分桶后获取该桶内最匹配的顶部文档列表，即详情数据

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "jobs":{
            "terms":{
               "field":"job.keyword",
                "size":10
            },
            "aggs":{
                "top_employee":{
                    "top_hits":{
                        "size":10,
                        "sort":[
                            {
                                "age":{
                                    "order":"desc"
                                }
                            }
                        ]
                    }
                }
            }
        }
    }
}
```

## Bucker 聚合分析

Bucket ，意为桶，即按照一定的规则将文档分配到不同的桶中，达到分类分析的目的

![](D:\Typora\笔记\image\图片70.png)

按照Bucket的分桶侧策略，常见的Bucket聚合分析如下：

> Terms
>
> Range
>
> Date Range
>
> Histogram
>
> Date Histogram

### Terms

该分桶策略最简单，直接按照term来分桶，如果是text类型，则按照分词后的结果分桶

```json
GEt test_search_index/_search
{
    "size":0,
    "aggs":{
        "jobs":{
            #关键词
            "terms":{
                #指明term字段
               "field":"job.keyword",
                #指定返回数目
                "size":5
            }        
        }
    }
}
```

### Range

通过指定数值的方位来设定分桶规则

```json
GEt test_search_index/_search
{
	"size": 0,
	"aggs": {
		"salary_range": {
            #关键词
			"range": {
				"field": "salary",
            #指定每个range的范围
				"ranges": [{
					"to": 10000
				}, {
					"from": 10000,
					"to": 20000
				}, {
					"from": 20000
				}]
			}
		}
	}
}
```

### Date Range

通过指定日期的范围来设定分桶规则

```json
GEt test_search_index/_search
{
	"size": 0,
	"aggs": {
		"date_range": {
            #关键词
			"range": {
				"field": "birth",
            #指定返回结果的日期格式
                "format":"yyyy",
				"ranges": [{
            #指定日期，可以使用date math
                    "from":"1980",   
					"to": "1990"
				}, {
					"from": "1990",
					"to": "20000"
				}, {
					"from": "2000"
				}]
			}
		}
	}
}
```

### Historgram

直方图，以固定间隔的策略来分割数据

```json
GEt test_search_index/_search
{
	"size": 0,
	"aggs": {
		"salary_hist": {
            #关键词
			"histogram": {
				"field": "salary",
            #指定间隔大小
                "interval": 5000,
				"extended_bounds": {
            #指定数据范围
                    "min": 0   
					"max": 40000
				}
			}
		}
	}
}
```

### Date Historgram

针对日期的直方图或者柱状图，是时序数据分析中常用的聚合分析类型

```json
GEt test_search_index/_search
{
	"size": 0,
	"aggs": {
		"by_year": {
            #关键词
			"date_historgram": {
				"field": "birth",
            #指定间隔大小
                "interval": "year",
            #指定日期格式化
				"format": "yyyy"
			}
		}
	}
}
```

## Bucket + Mertric 聚合分析

Bucket 聚合分析允许通过添加子分析来进一步进行分析，该子分析可以使Bucket也可以是Metric。这也是的es的聚合分析能力变得异常强大

- 分桶之后再分桶

```json
GEt test_search_index/_search
{
	"size": 0,
	"aggs": {
        #第一层聚合
		"jobs": {
			"terms": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                #第二层聚合
                "age_range":{
                    "range":{
                        "field":"age",
                        "ranges":[
                            {"to":20},
                            {"from":20,"to":30},
                            {"from":30}
                        ]
                    }
                }
            }
		}
	}
}
```

- 分桶后进行数据分析

```json
GEt test_search_index/_search
{
	"size": 0,
	"aggs": {
        #第一层聚合
		"jobs": {
			"terms": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                #第二层聚合
                "salary":{
                    "stats":{
                        "field":"salary"
                    }
                }
            }
		}
	}
}
```

## Pipeline 聚合分析

针对聚合分析的结果再次进行聚合分析，而且支持链式调用

```json
POST order/_search
{
	"size": 0,
	"aggs": {
		"sales_per_month": {
			"date_historgram": {
				"field": "date",
                "interval": "month"
			},
            "aggs":{
                "sales":{
                    "sum":{
                        "field":"price"
                    }
                }
            }
		},
        "avg_monthly_sales":{
            "avg_bucket":{
                "buckets_path":"sales_per_month>sales"
            }
        }
	}
}
```

Pipeline 的分析结果会输出到原结果中，根据输出位置的不同，分为以下两类

- Parent 结果内嵌到现有的聚合分析结果中
  - Derivative
  - Moving Average
  - Cumulative Sum
- Sibling 结果与现有聚合分析结果同级
  - Max/Min/Avg/Sum Bucket
  - Stats/Extended Stats Bucket
  - Percentiles Bucket

### Min Bucket

找出所有Bucket中值最小的Bucket名称和值

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                "avg_salary":{
                    "avg":{
                        "field":"salary"
                    }
                }
            }
		},
        "min_salary_by_job":{
            #关键词
            "min_bucket":{
                "buckets_path":"jobs>avg_salary"
            }
        }
	}
}
```

### Max Bucket

找出所有Bucket中值最大的bucket名称和值

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                "avg_salary":{
                    "avg":{
                        "field":"salary"
                    }
                }
            }
		},
        "max_salary_by_job":{
            #关键词
            "max_bucket":{
                "buckets_path":"jobs>avg_salary"
            }
        }
	}
}
```

### Avg Bucket

计算所有Bucket的平均值

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                "avg_salary":{
                    "avg":{
                        "field":"salary"
                    }
                }
            }
		},
        "avg_salary_by_job":{
            #关键词
            "avg_bucket":{
                "buckets_path":"jobs>avg_salary"
            }
        }
	}
}
```

### Sum Bucket

计算所有Bucket值的总和

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                "avg_salary":{
                    "avg":{
                        "field":"salary"
                    }
                }
            }
		},
        "sum_salary_by_job":{
            #关键词
            "sum_bucket":{
                "buckets_path":"jobs>avg_salary"
            }
        }
	}
}
```

### Stats Bucket

计算所有Bucket值的stats分析

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                "avg_salary":{
                    "avg":{
                        "field":"salary"
                    }
                }
            }
		},
        "stats_salary_by_job":{
            #关键词
            "stats_bucket":{
                "buckets_path":"jobs>avg_salary"
            }
        }
	}
}
```

### Percentiles Bucket

计算所有Bucket值得百分位数

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword",
                "size": 10
			},
            "aggs":{
                "avg_salary":{
                    "avg":{
                        "field":"salary"
                    }
                }
            }
		},
        "percentile_salary_by_job":{
            #关键词
            "percentile_bucket":{
                "buckets_path":"jobs>avg_salary"
            }
        }
	}
}
```

### Parent - Derivative

计算Bucket值得导数

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"birth": {
			"date_historgram": {
				"field": "birth",
				"interval": "year",
				"min_doc_count": 0
			},
			"aggs": {
				"avg_salary": {
					"avg": {
						"field": "salary"
					}
				},
				"derivative_avg_salary": {
                    #关键词
					"derivative": {
						"buckets_path": "avg_salary"
					}
				}
			}
		}
	}
}
```

### Moving Average

计算Bucket值得移动平均值

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"birth": {
			"date_historgram": {
				"field": "birth",
				"interval": "year",
				"min_doc_count": 0
			},
			"aggs": {
				"avg_salary": {
					"avg": {
						"field": "salary"
					}
				},
				"mavg_salary": {
                    #关键词
					"moving_avg": {
						"buckets_path": "avg_salary"
					}
				}
			}
		}
	}
}
```

### Cumulative Sum

计算Bucket值得累积加和

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"birth": {
			"date_historgram": {
				"field": "birth",
				"interval": "year",
				"min_doc_count": 0
			},
			"aggs": {
				"avg_salary": {
					"avg": {
						"field": "salary"
					}
				},
				"cumulative_salary": {
                    #关键词
					"cumulative_sum": {
						"buckets_path": "avg_salary"
					}
				}
			}
		}
	}
}
```

## 作用范围

es聚合分析默认作用范围是`query`的结果集，可以通过如下的方式改变其作用范围：

- filter
- post_filter
- global

```json
GET test_search_index/_search
{
	"size": 0,
	"query": {
        #aggs作用于该query的结果集
		"match": {
			"username": "alfred"
		}
	},
	"aggs": {
		"jobs": {
			"terms": {
				"field": "job.keyword",
				"size": 10
			}
		}
	}
}
```

### filter

为某个聚合分析设定过滤条件，从而在不更改整体query语句的情况下修改了作用范围

```json
GET test_search_index/_search
{
	"aggs": {
		"jobs_salary_small": {
            #过滤条件
			"filter": {
				"range": {
					"salary": {
						"to": 10000
					}
				}
			},
			"aggs": {
				"jobs": {
					"term": {
						"field": "key.word"
					}
				}
			}
		},
		"jobs": {
			"terms": {
				"field": "job.keyword"
			}
		}
	}
}
```

### post-filter

作用于文档过滤，但在聚合分析后生效

```json
GET test_search_index/_search
{
	"aggs": {
		"jobs": {
			"term": {
				"field": "job.keyword"
			}
		}
	},
    #过滤条件
	"post_filter": {
		"match": {
			"job.keyword": "java.engineer"
		}
	}
}
```

### global

无视query过滤条件，基于全部文档进行分析

```json
GET test_search_index/_search
{
	"query": {
		"match": {
			"job.keyword": "java.engineer"
		}
	},
	"aggs": {
		"java_avg_salary": {
			"avg": {
				"field": "salary"
			}
		},
		"all": {
			"global": {},
			"aggs": {
                #过滤条件
				"avg_salary": {
					"avg": {
						"field": "salary"
					}
				}
			}
		}
	}
}
```

## 排序 

可以使用自带的关键数据进行排序，比如：

- _count 文档数
- _key 按照key值排序

```json
GET test_search_index/_search
{
	"size": 0,
	"aggs": {
		"jobs": {
			"terms": {
				"field": "job.keyword",
				"size": 10,
				"order": [{
					"_count": "asc"
				}, {
					"_key": "desc"
				}]
			}
		}
	}
}


{
	"size": 0,
	"aggs": {
		"salary_hist": {
			"histogram": {
				"field": "salary",
				"interval": 5000,
				"order": {
                    #更深层次的嵌套
					"avg>avg_age": "desc"
				}
			},
			"aggs": {
				"age": {
					"filter": {
						"range": {
							"age": {
								"gte": 10
							}
						}
					},
					"aggs": {
						"avg_age": {
							"avg": {
								"field": "age"
							}
						}
					}
				}
			}
		}
	}
}


```

## 计算精准度问题

### Min 聚合的执行流程

```json
GET test_search_index/_search
{
    "size":0,
    "aggs":{
        "min_age":{
            "min":{
                "field":"age"
            }
        }
    }
}
```

![](D:\Typora\笔记\image\图片71.png)

### Terms 聚合的执行流程

```json
GET test_search_index/_search
{
    "size":0,
    "aggs":{
        "jobs":{
            "terms":{
                "field":"job.keyword",
                "size":5
            }
        }
    }
}
```

![](D:\Typora\笔记\image\图片72.jpg)

**Terms 并不永远准确**

原因：

> 数据分散在多Shard上，Coordinating Node无法得悉数据全貌

解决：

> 设置Shard数为1，消除数据分散的问题，但无法承载大数据量
>
> 合力设置Shard_Size 大小，即每次从Shard上额外多获取数据，以提升准确度

```json
GET test_search_index/_search
{
    "size":0,
    "aggs":{
        "jobs":{
            "terms":{
                "field":"job.keyword",
                "size":1,
                "shard_size":10
            }
        }
    }
}
```

**Shard_Size 大小的设定方法**

terms聚合返回结果中有如下两个统计值：

- `doc_count_error_upper_bound`被遗漏的term可能的最大值
- `sum_other_doc_count`返回结果bucket的term外其他term的文档总数

```json
{
    "took":19,
    ...
    "aggregations":{
        "states":{
        "doc_count_error_upper_bound":20,
        "sum_other_doc_count":51269,
        "buckts":[]
       }
   }
}
```

设定show_term_doc_count_errorr 可以查看每个bucket误算的最大值

```json
GET test_search_index/_search
{
    "size":0,
    "aggs":{
        "jobs":{
            "terms":{
                "field":"job.keyword",
                "size":2,
                "show_term_doc_count_error":true
            }
        }
    }
}


response
{
	"took": 4,
    ...
	"aggregations": {
		"jobs": {
			"doc_count_error_upper_bound": 20,
			"sum_other_doc_count": 51269,
			"buckts": [{
				"key": "ruby engineer",
				"doc_count": 2,
				"doc_count_error_upper_bound": 0 #0表明计算准确
			}]
		}
	}
}
```

Shard_Size默认大小如下：

- shard_size=(size * 1.5) + 10

通过调整Shard_Size的大小降低`doc_count_error_upper_bound`来提升准确度

- 增大了整体的计算量，从而降低了响应时间

### 近似统计算法

![](D:\Typora\笔记\image\图片73.jpg)

在ES的聚合分析中，Cardinality和Percentile分析使用的是近似统计算法

- 结果是近似准确的，但不一定精准
- 可以通过参数的调整使其结果精准，但同时也意味着更多的计算时间和更大的性能消耗

# 第八章 数据建模 

英文为Data Modeling，为创建数据模型的过程

数据模型（Data Model）

- 对现实世界进行抽象描述的一种工具和方法
- 通过抽象的实体和实体之间的联系的形式去描述业务规则，从而实现对现实世界的映射

概念模型

- 确定系统的核心需求和范围边界，涉及实体和实体间的关系

逻辑模型

- 进一步梳理业务需求，确定每个实体的属性、关系和约束等

物理模型

- 结合具体的数据产品，在满足业务读写性能等需求的前提下确定最终的定义
- Mysql、MongoDB、elasticsearch等
- 第三范式

ES是基于Lucene以倒排索引为基础实现的存储体系，不遵循关系型数据库中的范式约定

![](D:\Typora\笔记\image\图片74.jpg)

## **关联关系处理**

ES不擅长处理关系型数据库中的关联关心，比如文章表blog与评论表comment之间通过`blog_id`关联，在ES中可以通过如下两种手段变相解决

- Nested Object
- Parent/Child

### Nested Object

Comments默认是Object Array，存储结构类似下面的形式：

```json
{
    "title":"Blog Number One",
    "author":"alfred",
    "comments.username":["lee","fax"],
    "comments.date":["2017-01-02","2017-04-02"],
    "comments.content":["awesome article!","thanks!"]
}
```

Nested Object Array的存储结构类似下面的形式：

```json
{
    "comments.username":"lee",
    "comments.date":"2017-01-02",
    "comments.content":"awesome article!"
}
```

### Parent/Child

ES还提供饿了类似关心数据库中join的实现方式，使用`join`数据类型实现

```json
PUT bolg_index_parent_child
{
    "mapping":{
        "doc":{
            "properties":{
                "join":{
                    #指明类型
                    "type":"join",
                    #指明父子关系
                    "relations":{
                        "bolg":"comment" #父类型名称 ；子类型名称
                    }
                }
            }
        }
    }
}


#创建父文档
PUT blog_index_parent_child/doc/1
{
    "title":"blog",
    #指明父类型
    "join":"blog"
}

#创建子文档
PUT blog_index_parent_child/doc/comment-1?routing=1 #指明routing值，确保父子文档在一个分片上，一般使用父文档ID
{
    "comment":"comment world",
    "join":{
        #指明子类型
        "name":"comment",
        #指明父文档ID
        "parent":1
    }
}
```

常见query语法包括如下几种：

- parent_id 返回某父文档的子文档

```json
GET blog_index_parent_child/_search
{
    "query":{
        #关键词
        "parent_id":{
        #指定子文档类型
            "type":"commnet",
            "id":"2" #指明父文档ID
        }
    }   
}
```

- has_child 返回包含某子文档的父文档

```json
GET blog_index_parent_child/_search
{
	"query": {
		"has_child": {
			"type": "commnet",
			"query": {
				"match": {
					"comment": "world"
				}
			}
		}
	}
}
```

- has_parent 返回包含某父文档额子文档

```json
GET blog_index_parent_child/_search
{
	"query": {
		"has_parent": {
			"parent_type": "blog",
			"query": {
				"match": {
					"title": "world"
				}
			}
		}
	}
}
```

### Nested Object Vs Parent/Child

| 对比 | Nested Object                    | Parent/Child                                     |
| ---- | -------------------------------- | ------------------------------------------------ |
| 优点 | 文档存储在一起，因此读性能高     | 父子文档可以独立更新，互补影响                   |
| 缺点 | 更新父或子文档时需要更新整个文档 | 为了维护join的关系，需要占用部分内存读取性能较差 |
| 场景 | 子文档偶尔更新，查询频繁         | 子文档更新频繁                                   |

**建议尽量选择Nested Object来解决问题**

## Reindex

指重建所有数据的过程，一般发生在如下情况：

- mapping设置变更，比如字段类型变化，分词器字段更新等
- index设置变更，比如分片数更改等
- 迁移数据

ES提供了县城的API用于完成该工作

### _update_by_query 

在现有索引上重建

```json
#将blog_index的所有文档重建一遍
POST blog_index/_update_by_query?conficts=proceed #如果遇到版本冲突，覆盖并继续执行

POST blog_index/_update_by_query
{
   #更新文档的字段值
	"script": {
		"source": "ctx._source.likes++",
		"lang": "painless"
	},
#可以更新部分文档
	"query": {
		"term": {
			"user": "tom"
		}
	}
}
```

### _reindex

在其他索引上重建

```json
#将source的数据重建到dest中
POST _reindex
{
    "source":{
        "indedx":"blog_index"
    },
    "dest":{
        "index":"blog_new_index"
    }
}
```

#### **Task**

数据重建的时间受源索引文档规模的影响，当规模越大时，所需时间越多，此时需要通过设定url参数`wait_for_completion为`false`来异步执行，ES以task来描述此类执行任务。

ES提供了Task API来查看任务的执行进度和相关数据

```json
POST blog_index/_update_by_query?conflicts=proceed&wait_for_completion=false

GET _task/_qKI6E8_TD..
```

## 数据模型办版本管理

对Mapping进行版本管理

- 包含在代码或者以专门的文件进行管理，添加好注释，并加入Git等版本管理仓库中，方便回顾
- 为每个增加一个metadata字段，在其中维护一些文档相关的元数据，方便对数据进行管理

```json
#mapping 版本，可以自行指定，比如每次更新mapping设置后，该version加1
{
    "matadata":{
        "version":1
    },
    "username":"alfred",
    "job":"engineer"
}
```

**防止字段过多**

字段过多主要有如下的坏处：

- 难于维护，当字段成百上千时，基本很难有人能明确知道每个字段的含义
- mapping的信息存储在cluster state里面，过多的字段会导致mapping多大，最终导致更新变慢

通过设置`index.mapping.total_fields.limit`可以限定索引中最大字段数，默认是1000

可以通过key/value的方式解决字段过多的问题，但并不完美

> 虽然通过这种方式可以极大地减少Field数目，但也有一些明显的坏处
>
> - query语句复杂度飙升，且有一些可能无法实现，比如聚合分析相关度的
> - 不理在Kibana中做可视化分析

一般字段过多的原因是由于没有高质量的数据建模导致的，比如`dynamic`设置为`true`

考虑拆分多个索引来解决问题

# 第九章 集群调优建议

动态设定的参数有transient和persistent两种设置，前者在集群重启后会丢失、后者不会，但两种设定都会覆盖elasticsearch.yml中的配置

```json
PUT /_cluster/settings
{
    "persistent":{
        "discovery.zen.minimum_master_nodes":2
    },
    "transient":{
        "indices.store.throttle.max_bytes_per_sec":"50mb"
    }
}
```

**关于jvm内存设定**

- 不要超过31GB
- 预留一半内存给操作系统，用来做文件缓存

**写性能优化**

目标：增大写吞吐量-EPS（Events Per Second）越高越好

优化方案：

- 客户端：多线程写，批量写
- ES：在`高质量数据建模`的前提下，主要是在refresh、translog和flush之间做文章

## X-Park Monitoring

官方推出的免费集群监控功能

