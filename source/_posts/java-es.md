---
title: StringBoot + ElasticSearch
date: 2021-02-25 00:17:18
tags:
    - java
categories:
    - java
---

### [StringBoot（2.4.3）+ ElasticSearch（7.9.3）](https://github.com/ilubov/elasticsearch-demo)

#### ElasticsearchRepository
[文档地址](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.3/reference/html/#repositories.core-concepts)
##### 实体类

* @Document（必写）

|属性名	| 说明 |
|  ----  | ----  |
|indexName	| 索引名，支持SpEl |
|shards	| 分片 |
|replicas	| 每个分区备份数 |
|refreshInterval	| 刷新间隔，默认1s |
|indexStoreType	| 索引文件存储类型，默认fs |
|versionType	| 配置版本管理，默认VersionType.EXTERNAL |

* @Id（必写）

* @Field

|属性名	| 说明 |
|  ----  | ----  |
| name	| 将在Elasticsearch文档中表示的字段名称，如果未设置，则使用Java字段名称 |
| type	| 属性类型 |
| store	| 标记是否原始字段值应存储在Elasticsearch中，默认值为false。 |
| analyzer | 分词 |

* demo

```
@Data
@Document(indexName = "test")
public class EsTestVo {

    @Id
    private Long id;

    @Field(type = FieldType.Text)
    private String name;

    @Field(type = FieldType.Date)
    private Date time;

    @Field(type = FieldType.Keyword)
    private List<String> tags;

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String desc;
}
```

##### Repository接口

* 继承ElasticsearchRepository<T, ID>

* ElasticsearchRepository接口中的方式都是@Deprecated

* 两种方式：关键字拼接查询条件、@Query注解查询

|Keyword	| Sample | Elasticsearch Query String |
|  ----  | ----  | ----  |
|And |	findByNameAndPrice |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } }, { "query_string" : { "query" : "?", "fields" : [ "price" ] } } ] } }} |
|Or |	findByNameOrPrice |	{ "query" : { "bool" : { "should" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } }, { "query_string" : { "query" : "?", "fields" : [ "price" ] } } ] } }} |
|Is |	findByName |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } } ] } }} |
|Not |	findByNameNot |	{ "query" : { "bool" : { "must_not" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } } ] } }} |
|Between |	findByPriceBetween |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : ?, "to" : ?, "include_lower" : true, "include_upper" : true } } } ] } }} |
|LessThan |	findByPriceLessThan |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : null, "to" : ?, "include_lower" : true, "include_upper" : false } } } ] } }} |
|LessThanEqual |	findByPriceLessThanEqual |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : null, "to" : ?, "include_lower" : true, "include_upper" : true } } } ] } }} |
|GreaterThan |	findByPriceGreaterThan |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : ?, "to" : null, "include_lower" : false, "include_upper" : true } } } ] } }} |
|GreaterThanEqual |	findByPriceGreaterThan |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : ?, "to" : null, "include_lower" : true, "include_upper" : true } } } ] } }} |
|Before |	findByPriceBefore |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : null, "to" : ?, "include_lower" : true, "include_upper" : true } } } ] } }} |
|After |	findByPriceAfter |	{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : ?, "to" : null, "include_lower" : true, "include_upper" : true } } } ] } }} |
|Like |	findByNameLike |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "?*", "fields" : [ "name" ] }, "analyze_wildcard": true } ] } }} |
|StartingWith |	findByNameStartingWith |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "?*", "fields" : [ "name" ] }, "analyze_wildcard": true } ] } }} |
|EndingWith |	findByNameEndingWith |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "*?", "fields" : [ "name" ] }, "analyze_wildcard": true } ] } }} |
|Contains/Containing |	findByNameContaining |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "*?*", "fields" : [ "name" ] }, "analyze_wildcard": true } ] } }} |
|In  (when annotated as FieldType.Keyword) | 	findByNameIn(Collection<String>names) |	{ "query" : { "bool" : { "must" : [ {"bool" : {"must" : [ {"terms" : {"name" : ["?","?"]}} ] } } ] } }} |
|In	 | findByNameIn(Collection<String>names) |	{ "query": {"bool": {"must": [{"query_string":{"query": "\"?\" \"?\"", "fields": ["name"]}}]}}} |
|NotIn  (when annotated as FieldType.Keyword) |	findByNameNotIn(Collection<String>names) |	{ "query" : { "bool" : { "must" : [ {"bool" : {"must_not" : [ {"terms" : {"name" : ["?","?"]}} ] } } ] } }} |
|NotIn |	findByNameNotIn(Collection<String>names) |	{"query": {"bool": {"must": [{"query_string": {"query": "NOT(\"?\" \"?\")", "fields": ["name"]}}]}}} |
|Near |	findByStoreNear |	Not Supported Yet ! |
|True |	findByAvailableTrue |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "true", "fields" : [ "available" ] } } ] } }} |
|False |	findByAvailableFalse |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "false", "fields" : [ "available" ] } } ] } }} |
|OrderBy |	findByAvailableTrueOrderByNameDesc |	{ "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "true", "fields" : [ "available" ] } } ] } }, "sort":[{"name":{"order":"desc"}}] } |

* demo

```
public interface EsTestRepository extends ElasticsearchRepository<EsTestVo, Long> {

    @Query("{\"match\": {\"name\": {\"query\": \"?0\"}}}")
    List<EsTestVo> findByName(String name);

    List<EsTestVo> findByDesc(String desc);

    Page<EsTestVo> findByDesc(String desc, Pageable pageable);

    List<EsTestVo> findByNameOrDesc(String name, String desc);
}
```

#### ElasticsearchRestTemplate
[文档地址](https://docs.spring.io/spring-data/elasticsearch/docs/4.1.3/reference/html/#elasticsearch.operations)

主要有以下操作

* IndexOperations定义索引级别的操作，如创建或删除索引。

* DocumentOperations定义基于实体 ID 存储、更新和检索实体的操作。

* SearchOperations定义使用查询搜索多个实体的操作

* demo

```
@PostMapping("/update")
@ApiOperation("修改")
public Object update(@RequestBody EsTestVo entity) {
    String id = entity.getId().toString();
    Document document = Document.parse(JSON.toJSONString(entity));
    UpdateQuery build = UpdateQuery.builder(id)
            .withDocument(document) .withScriptedUpsert(true) .build();
    return elasticsearchRestTemplate.update(build, IndexCoordinates.of("test"));
}
```

#### sql
