# elasticsearch 常用命令
## 基本命令
1.1 获取所有_cat命令
命令：curl -XGET localhost:9200/_cat

[elasticsearch@test-es7-master-0 ~]$ curl -XGET localhost:9200/_cat
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health

以上的命令中，你也可以后面加一个v，让输出内容表格显示表头

1.2 获取es集群服务健康状态
命令：curl -X GET “localhost:9200/_cat/health?v”

1.3 查看es节点信息
命令：curl -XGET ‘localhost:9200/_cat/nodes?v’

1.4 查看es指定节点信息
命令：curl -XGET ‘localhost:9200/_nodes/nodeName?pretty=true’

## 索引操作
1. 查看ES中所有的索引
命令：curl -X GET “ip地址:9200/_cat/indices?v”
示例：curl -X GET localhost:9200/_cat/indices?v


2. 新建索引
命令：curl -X PUT ‘localhost:9200/test’
示例：新建一个名字为test的 Index。创建后返回下面的json对象。“acknowledged”:true表示创建成功

curl -X PUT localhost:9200/test         
{
 "acknowledged":true,
 "shards_acknowledged":true,
 "index":"test-zp"
 }

3. 删除索引
命令：curl -X DELETE ‘localhost:9200/test’
示例：删除名为test的Index。“acknowledged”:true表示删除成功

curl -X DELETE localhost:9200/test         
{
 "acknowledged":true
 }

4. 查看指定索引信息
命令：curl -XGET “http://localhost:9200/test?pretty” 注意：test是索引名

4. 查看索引的统计信息
命令：curl -XGET “http://localhost:9200/test/_stats?pretty” 注意：test是索引名

## 文档操作 *
3.1 查询索引中的全部文档
命令：curl -X GET localhost:9200/index_name/_search?pretty
示例：curl -XGET localhost:9200/1021car_10061v1/_search?pretty 注意： ?pertty 表示让数据格式化，更好的展示
如图：显示指定索引下文档的信息


3.2 根据条件查询索引中的文档
单一条件搜索：
1、搜索品牌是大众的汽车
命令：curl -H “Content-Type: application/json” -XPOST ‘http://localhost:9200/1021car_10061v1/_search?pretty’ -d ‘{“query”: { “match”: { “brand”: “大众” } }}’
多条件搜索：
1、搜索品牌是大众，并且车型SUV的汽车（&&使用 must ）
命令：curl -H “Content-Type: application/json” -XPOST ‘http://localhost:9200/1021car_10061v1/_search?pretty’ -d ‘{“query”: {“bool”: {“must”: [{ “match”: { “brand”: “大众” } },{ “match”: { “body”: “SUV”} }]}}}’
2、搜索品牌是大众或者奥迪的汽车（|| 使用 should ）
命令：curl -H “Content-Type: application/json” -XPOST ‘http://localhost:9200/1021car_10061v1/_search?pretty’ -d ‘{“query”: {“bool”: {“should”: [{ “match”: { “brand”: “大众” } },{ “match”: { “brand”: “奥迪”} }]}}}’
3、搜索品牌是大众但车型不是SUV的汽车
命令：curl -H “Content-Type: application/json” -XPOST ‘http://localhost:9200/1021car_10061v1/_search?pretty’ -d ‘{“query”: {“bool”: { “must”: [{ “match”: { “brand”: “大众” } }],“must_not”: [{ “match”: { “body”: “SUV” } }]}}}’
4、统计品牌是大众的汽车数量有多少种
命令：curl -H “Content-Type: application/json” -XPOST ‘http://localhost:9200/1021car_10061v1/_count?pretty’ -d ‘{“query”: { “match”: { “brand”: “大众” } }}’
