# Sample Queries

### Health Check
```
$ curl -X GET "http://localhost:9200/_cat/health?v&pretty"
```

```
epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1580891446 08:30:46  docker-cluster green           1         1      0   0    0    0        0             0                  -                100.0%
```

### Empty Index

```
$ curl -X GET "http://localhost:9200/_cat/indices?v&pretty"
```

```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

### Create Index and Mapping

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

### Show Index

```
$ curl -X GET "http://localhost:9200/_cat/indices?v&pretty"
```

```
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   user  fS6Db0-rRuqgFSAH6GSuOw   1   1          0            0       230b           230b
```

### Show Mapping

```
$ curl -X GET "http://localhost:9200/user/_mapping?pretty&pretty"
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

### Create Document

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Kota",
  "age" : 24
}'
```

```
{"_index":"user","_type":"_doc","_id":"nW96FHAByJp-ps0eunEk","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Tanaka",
  "age" : 48
}'
```

```
{"_index":"user","_type":"_doc","_id":"n29-FHAByJp-ps0e73Ga","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":1,"_primary_term":1}
```

### Type Error

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Kota",
  "age" : "Kota"
}'
```

```
{"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":"failed to parse field [age] of type [integer] in document with id 'nm99FHAByJp-ps0elXFA'. Preview of field's value: 'Kota'"}],"type":"mapper_parsing_exception","reason":"failed to parse field [age] of type [integer] in document with id 'nm99FHAByJp-ps0elXFA'. Preview of field's value: 'Kota'","caused_by":{"type":"number_format_exception","reason":"For input string: \"Kota\""}},"status":400}
```

### Get all data

```
$ curl -X GET http://localhost:9200/user/_search
```

```
{"took":834,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":2,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"user","_type":"_doc","_id":"nW96FHAByJp-ps0eunEk","_score":1.0,"_source":{
  "name" : "Kota",
  "age" : 24
}},{"_index":"user","_type":"_doc","_id":"n29-FHAByJp-ps0e73Ga","_score":1.0,"_source":{
  "name" : "Tanaka",
  "age" : "48"
}}]}}
```

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
{"took":30,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":0.6931472,"hits":[{"_index":"user","_type":"_doc","_id":"nW96FHAByJp-ps0eunEk","_score":0.6931472,"_source":{
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
{"took":27,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":0.0,"hits":[{"_index":"user","_type":"_doc","_id":"n29-FHAByJp-ps0e73Ga","_score":0.0,"_source":{
  "name" : "Tanaka",
  "age" : "48"
}}]}}
```
