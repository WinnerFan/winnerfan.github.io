### ElasticSearch

#### configuration

- path.data
- path.logs
- network.host
- http.port

#### basic
- insert index

```
curl -XPUT 'localhost:9200/first?pretty'
```
- delete index

```
curl -XDELETE 'localhost:9200/first?pretty'
```

- insert/change data

id可以省略(PUT不支持，POST系统自动分配ID)，id一样会覆盖，pretty为返回值美化
```
curl -X PUT 'localhost:9200/indexName/typeName/1?pretty' -H 'Content-Type: application/json' -d '
{
	"user": "aa"
}'

curl -X POST 'localhost:9200/indexName/typeName' -H 'Content-Type: application/json' -d '
{
	"user": "aa"
}'
```

- delete data

```
curl -X DELETE 'localhost:9200/indexName/typeName/1'

curl -X POST 'localhost:9200//indexName/typeName/_delete_by_query?pretty' -H 'Content-Type: application/json' -d '{
    "query": {
        "match": {
            "user": "aa"
        }
    }
}'

curl -X POST 'localhost:9200/indexName/typeName/_delete_by_query?pretty' -H 'Content-Type: application/json' -d '{
    "query": {
        "match_all": {}
    }
}'
```

- query data

```
curl -X GET 'localhost:9200/indexName/typeName/1?pretty'

```

#### cluster

```
localhost:9200/_cluster/health?pretty
localhost:9200/_cat
localhost:9200/_cat/nodes?pretty
```