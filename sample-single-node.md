# Sample Queries with `docker-compose.yml`

### Health check

```
$ curl -X GET "http://localhost:9200/_cat/health?v&pretty"
```

```
epoch      timestamp cluster                        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1583303099 06:24:59  elasticsearch_practice_cluster green           1         1      0   0    0    0        0             0                  -                100.0%
```

### Empty index

```
$ curl -X GET "http://localhost:9200/_cat/indices?v&pretty"
```

```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

### Create index and mapping

```
$ curl -X PUT "http://localhost:9200/user" -H 'Content-Type: application/json' \
-d '{
  "mappings" : {
    "properties" : {
      "name" : { "type" : "text" },
      "age" : { "type" : "integer" }
    }
  }
}'
```

```
{"acknowledged":true,"shards_acknowledged":true,"index":"user"}
```

### Show index

```
$ curl -X GET "http://localhost:9200/_cat/indices?v&pretty"
```

```
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   user  TpURqIE_Sl23NcojGzMIaw   1   1          0            0       230b           230b
```

### Show mapping

```
$ curl -X GET "http://localhost:9200/user/_mapping?pretty"
```

```
{
  "user" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "name" : {
          "type" : "text"
        }
      }
    }
  }
}
```

### Create document

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Kota",
  "age" : 24
}'
```

```
{"_index":"user","_type":"_doc","_id":"ySA5pHABgo6kz09YytGe","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Tanaka",
  "age" : 48
}'
```

```
{"_index":"user","_type":"_doc","_id":"yiA6pHABgo6kz09YGdFj","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term":1}
```

### Type error

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Kota",
  "age" : "Kota"
}'
```

```
{"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":"failed to parse field [age] of type [integer] in document with id 'yyA6pHABgo6kz09YktFI'. Preview of field's value: 'Kota'"}],"type":"mapper_parsing_exception","reason":"failed to parse field [age] of type [integer] in document with id 'yyA6pHABgo6kz09YktFI'. Preview of field's value: 'Kota'","caused_by":{"type":"number_format_exception","reason":"For input string: \"Kota\""}},"status":400}
```

### Get all data

```
$ curl -X GET http://localhost:9200/user/_search
```

```
{"took":959,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":2,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"user","_type":"_doc","_id":"ySA5pHABgo6kz09YytGe","_score":1.0,"_source":{
  "name" : "Kota",
  "age" : 24
}},{"_index":"user","_type":"_doc","_id":"yiA6pHABgo6kz09YGdFj","_score":1.0,"_source":{
  "name" : "Tanaka",
  "age" : 48
}}]}}
```

>"took":959

### Search

```
$ curl -X GET "http://localhost:9200/user/_search" -H 'Content-Type: application/json' \
-d '{
  "query": {
    "match": {
      "name": "Kota"
    }
  }
}'
```

```
{"took":21,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":0.6931472,"hits":[{"_index":"user","_type":"_doc","_id":"ySA5pHABgo6kz09YytGe","_score":0.6931472,"_source":{
  "name" : "Kota",
  "age" : 24
}}]}}
```

```
$ curl -X GET "http://localhost:9200/user/_search" -H 'Content-Type: application/json' \
-d '{
  "query" : {
    "bool":{
      "must":[],
      "filter":{
        "range" : {
          "age": {
            "gte": 40, "lte" : 50
          }
        }
      }
    }
  }
}'
```

```
{"took":32,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":0.0,"hits":[{"_index":"user","_type":"_doc","_id":"yiA6pHABgo6kz09YGdFj","_score":0.0,"_source":{
  "name" : "Tanaka",
  "age" : 48
}}]}}
```

### Delete document

```
$ curl -X DELETE "http://localhost:9200/user/_doc/yiA6pHABgo6kz09YGdFj"
```

```
{"_index":"user","_type":"_doc","_id":"yiA6pHABgo6kz09YGdFj","_version":2,"result":"deleted","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":2,"_primary_term":1}
```

```
$ curl -X GET "http://localhost:9200/user/_search" -H 'Content-Type: application/json' \
-d '{
  "query": {
    "match": {
      "name": "Tanaka"
    }
  }
}'
```

```
{"took":279,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":0,"relation":"eq"},"max_score":null,"hits":[]}}
```
