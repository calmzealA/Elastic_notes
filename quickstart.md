# 快速开始


---

健康状态命令

```
curl 'localhost:9200/_cat/health?v'
```

查看节点

```
curl 'localhost:9200/_cat/nodes?v'
```

列出所有索引

```
curl 'localhost:9200/_cat/indices?v'
```

创建索引

```
curl -XPUT 'localhost:9200/customer?pretty'
```

创建类型和文档

```
curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "John Doe"
}'
```

查看文档
```
curl -XGET 'localhost:9200/customer/external/1?pretty'
```
删除索引
```
curl -XDELETE 'localhost:9200/customer?pretty'
```
修改更新索引

```
curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
{
  "name": "John Doe"
}'
```
删除文档
```
curl -XDELETE 'localhost:9200/customer/external/2?pretty'
```

### 批量操作


创建
```
curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'```

更新和删除
```
curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
'```

从文件导入数据
```
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary "@accounts.json"
```
搜索
```
curl 'localhost:9200/bank/_search?q=*&pretty'
```
```
took – 执行的搜索耗时，单位毫秒
timed_out – 搜索是否超时
_shards – 搜索的碎片数已经成功和失败的数量
hits – 搜索的结果
hits.total – 匹配数
hits.hits – 结果数组，默认显示前10个
_score and max_score - ignore these fields for now
```

分页
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}'
```

排序
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}'
```
只返回结果里指定的字段内容
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}'
```
匹配 mill 或 lane
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match": { "address": "mill lane" } }
}'
```

匹配 mill lane
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": { "match_phrase": { "address": "mill lane" } }
}'```

匹配 mill 和 lane
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'```

匹配 mill 或 lane
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```

不能匹配 mill 和 lane
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}'
```
bool可以同时有多个条件
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}'
```

bool下可以通过filter来对指定字段在指定范围内进行过滤
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}'
```
聚合
```
curl -XPOST 'localhost:9200/bank/_search?pretty' -d '
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      }
    }
  }
}'
```
相当于下面的SQL语句
```
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```
上面的size=0是让结果不要返回 hits 结果
