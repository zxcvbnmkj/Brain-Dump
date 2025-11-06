# ElasticSearch- - -分布式/ RESTful 搜索引擎
## 介绍
- 基于 Apache Lucene 构建
利用了**倒排索引**，即 key 是分词，value 是该词语所在的文档 id。正排索引则是 key 是 id ，value 是文档
- 支持 java、python 等语言，其实是通过 RESTful 接口可被任何模型调用
- 面向 **文档(Document)** 的，文档以 json 形式存储；一个 Doc 相当于数据库中的一行数据；一个 **字段(Field)** 则是数据库中的列； **索引(index)** 就是 json 列表，相当于数据库的表； **映射(Mapping)** 是字段约束，相当于表字段约束
| 数据库 | ES| ES中的类型说明 |
|:-:|:-:|:-:|
| 表 | 索引 | Index => json 的列表 |
| 一行数据 | 文档 | Document => 一条 json 数据 |
| 列 | 字段 | Field |
| 约束 | 映射 | Mapping |
- `DSL`是 elasticsearch 提供的 JSON 风格的请求语句，用来操作 elasticsearch ，实现CRUD，相当于 SQL
- ES 可与数据库共同使用
  - 用户插入数据到数据库
  - 数据库数据同步到 ES
  - 用户从 ES 中高效查询
### 推荐阅读
- [官方文档](https://www.elastic.co/)
- [宝藏教程](https://www.cnblogs.com/sexintercourse/p/18646560)
### 相较于数据库，ES的优点
- 全文搜索
- 内置相关性排序
- 分布式扩展，可自动分片
- 向量搜索
### 适用于
- 推荐系统
- 网站的搜索框
- 日志分析
## 安装部署
### ElasticSearch
相当于数据库，存储海量数据
### IK 分词器
ES 默认只认识英文单词（按空格分词）。IK 分词器将一句中文，智能地切分成一个个有意义的词语，当用户搜索“xx”时，才能准确地找到包含这个词的文档。
### Kibana
- 可视化界面，地址为：IP + 5601 端口
- Dev Tool 中输入 `GET /_cat/indices?v` 可以查询 ES 中有哪些索引（表格），返回信息的各个字段意思是：
  - health：索引健康状态（green=健康，yellow=警告，red=异常）
  - index：索引名称
  - docs.count：文档数量（相当于表的行数）
  - store.size：存储大小
- 查询索引的映射（表的字段）可通过 `GET /schemas_dev1（索引名字）/_mapping` ,得到的数据如下所示，可看到有 7 个字段，从 `description` 到 `title`
```
{
  "schemas_dev1" : {
    "mappings" : {
      "properties" : {
        "description" : {
          "type" : "text",
          "analyzer" : "ik_max_word",
          "search_analyzer" : "ik_smart"
        },
        "eventName" : {
          "type" : "text",
          "analyzer" : "ik_max_word",
          "search_analyzer" : "ik_smart"
        },
        "level" : {
          "type" : "integer"
        },
        "loc" : {
          "type" : "keyword"
        },
        "requestType" : {
          "type" : "keyword"
        },
        "status" : {
          "type" : "integer"
        },
        "title" : {
          "type" : "text",
          "analyzer" : "ik_max_word",
          "search_analyzer" : "ik_smart"
        }
      }
    }
  }
}
```
