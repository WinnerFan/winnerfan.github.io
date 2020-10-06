---
layout: post
title: ElasticSearch
tags: ElasticSearch
---
## configuration

- path.data
- path.logs
- network.host
- http.port

## basic
关系数据库（ES）-》数据库（index）-》表（type）-》行（doc）-》列（field）
- create index
    ```
    curl -XPOST 'localhost:9200/first?pretty' -d @first.json
    
    first.json在同级目录，否则绝对路径
    {
    "settings":{
           "number_of_shards":3,
           "number_of_replicas":0
        },
    "mappings":{
          "typeName":{
               "dynamic":"strict",
               "_all":{"enabled":false},
               "properties":{
                   "user":{
                       "type":"keyword"
                   },
                   "name":{
                   "type":"string","index":"analyzed","analyzer":"ik_max_word","search_analyzer": "ik_max_word"},
                   "pic":{
                       "type":"string","index":"not_analyzed"}
                   },
                   "balance":{
                       "type":"long"
                   },
                   "updateTime":{
                       "type":"date","format":"yyyy-MM-dd HH:mm:ss.SSS"
                   }
               }
            }
        }
    }
    ```
- insert/create index

	```
	curl -XPUT 'localhost:9200/first?pretty'
	```
- delete index
	
	```
	curl -XDELETE 'localhost:9200/first?pretty'
	```
- query all index

    ```
    curl -XGET 'localhost:9200/_cat/indices?v'
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
	# 
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
	
  curl -X GET 'localhost:9200/indexName/typeName/_search?pretty'
  ```
## cluster

```
localhost:9200/_cluster/health?pretty
localhost:9200/_cat
localhost:9200/_cat/nodes?pretty
localhost:9200/_cat/indices?v
```

## Inverted Index

根据Age或Name等所有Field即**term**找ID List，即**Posting List**

- Term Index在内存中保存term的前缀，FST压缩，A
- Term dictionary在磁盘保存在block中，公共前缀压缩，Asia
- Posting List在磁盘保存，[1,2,3]

联合索引查找

- skip list

  [1,3,13,101,105,108,255,256,257]分成三个block，[1,3,13] [101,105,108] [255,256,257]，形成skip list第二层[1,101,255]。对三个block增量编码压缩（Frame Of Reference），即2,5,100保存为2,3,95。skip list跳过了解压缩过程

- bitset按位与

  [1,3,4,7,10]的bitset[1,0,1,1,0,0,1,0,0,1]，Bitmap存储空间随着文档个数线性增长，故数据结构Roaring Bitmap。Posting List除以65536，按商分组，block中保存余数，大于4096个使用bitset保存block，否则使用short（2byte保存）

## Mapping

mapping定义索引中字段，定义字段类型，字段倒排索引相关配置。例如description=This is a apple分词时不包含a，包含apple。query data时"description": "a"找不到对应文档，"description": "apple" 却可以。
