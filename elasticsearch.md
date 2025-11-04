# ElasticSearch- - -分布式/ RESTful 搜索引擎
## 介绍
- 基于 Apache Lucene 构建
利用了**倒排索引**，即 key 是分词，value 是该词语所在的文档 id。正排索引则是 key 是 id ，value 是文档
- 支持 java、python 等语言，其实是通过 RESTful 接口可被任何模型调用
- 面向 **文档(Document)** 的，文档以 json 形式存储；一个 Doc 相当于数据库中的一行数据；一个 **字段(Field)** 则是数据库中的列； **索引(index)** 就是 json 列表，相当于数据库的表； **映射(Mapping)** 是字段约束，相当于表字段约束
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
可视化界面
## java 使用
```
package com.nowcoder.ut.david.web.service.search;

import com.nowcoder.avatar.common.consts.Constants;
import com.nowcoder.ut.david.entity.BiSchema;
import com.nowcoder.ut.david.web.config.GlobalSetting;
import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.sort.SortOrder;
import org.enthusa.avatar.face.type.PageModel;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.io.IOException;

@Service
// 针对 BiSchema 实体的搜索服务
// 抽象父类封装了通用的 ES 操作逻辑（如搜索、索引创建）
public class BiSchemaSearchService extends org.enthusa.avatar.search.AbstractSearchService<BiSchema> {
    // 注入 GlobalSetting 实例，用于获取环境配置（如当前是生产环境还是开发环境
    @Resource
    private GlobalSetting globalSetting;

    @Autowired
    public BiSchemaSearchService(@Qualifier("utEsClient") RestHighLevelClient client) {
        super(client);
    }

    @Override
    public String getIndexName() {
        if (StringUtils.equalsAny(globalSetting.getEnv(), Constants.ENV_PROD, Constants.ENV_PRE)) {
            return String.format("%s_prod%d", Constants.ES_SCHEMA, 1);
        }
        return String.format("%s_dev%d", Constants.ES_SCHEMA, 1);
    }

    @Override
    public IndexRequest buildIndex(BiSchema schema) {
        return new IndexRequest(getIndexName()).id(schema.getId().toString()).source("loc", schema.getLoc(), "level", schema.getLevel(), "requestType", schema.getRequestType(), "eventName", schema.getEventName(), "title", schema.getTitle(), "description", schema.getDescription(), "status", schema.getStatus());
    }

    @Override
    public QueryBuilder buildQuery(String query) {
        BoolQueryBuilder queryBuilder = QueryBuilders.boolQuery();
        queryBuilder.should(QueryBuilders.termQuery("loc", query).boost(4.0f));
        queryBuilder.should(QueryBuilders.wildcardQuery("eventName", "*" + query + "*").boost(4.0f));
        queryBuilder.should(QueryBuilders.matchQuery("title", query).boost(2.0f));
        queryBuilder.should(QueryBuilders.matchQuery("description", query));
        return queryBuilder;
    }

    @Override
    public PageModel<SearchHit> search(QueryBuilder query, int page, int pageSize) throws IOException {
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.sort("status", SortOrder.ASC);
        sourceBuilder.query(query);
        return this.search(sourceBuilder, page, pageSize);
    }
}
```
