# Sample Queries with `docker-compose-3nodes.yml`

### Health check

* Cluster

```
$ curl -X GET "http://localhost:9200/_cat/health?v&pretty"
```

```
epoch      timestamp cluster                               status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1583303608 06:33:28  elasticsearch-practice-cluster-3nodes green           3         3      0   0    0    0        0             0                  -                100.0%
```

* Nodes

```
$ curl -X GET "http://localhost:9200/_cat/nodes?v&pretty"
```

```
ip            heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.160.4           30          77  34    1.55    0.98     0.48 dilm      -      elasticsearch_3
192.168.160.3           30          77  35    1.55    0.98     0.48 dilm      -      elasticsearch_2
192.168.160.2           49          77  35    1.55    0.98     0.48 dilm      *      elasticsearch_1
```

* Master Node

```
$ curl -X GET "http://localhost:9200/_cat/master?v&pretty"
```

```
id                     host          ip            node
RatiPhOHTHmFKSHnNW6INQ 192.168.160.2 192.168.160.2 elasticsearch_1
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
green  open   user  CojPNu9SRwqx6qfPCrNbOg   1   1          0            0       460b           230b
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
{"_index":"user","_type":"_doc","_id":"M1lDpHABKBUa_Fhbupkm","_version":1,"result":"created","_shards":{"total":2,"successful":2,"failed":0},"_seq_no":0,"_primary_term":1}
```

```
$ curl -X POST "http://localhost:9200/user/_doc" -H 'Content-Type: application/json' \
-d '{
  "name" : "Tanaka",
  "age" : 48
}'
```

```
{"_index":"user","_type":"_doc","_id":"NFlFpHABKBUa_FhbNJn-","_version":1,"result":"created","_shards":{"total":2,"successful":2,"failed":0},"_seq_no":1,"_primary_term":1}
```

### Show setting of `user`

```
$ curl -X GET "http://localhost:9200/user/_settings?pretty"
```

```
{
  "user" : {
    "settings" : {
      "index" : {
        "creation_date" : "1583303786490",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "CojPNu9SRwqx6qfPCrNbOg",
        "version" : {
          "created" : "7060099"
        },
        "provided_name" : "user"
      }
    }
  }
}
```

>"number_of_shards" : "1",
>"number_of_replicas" : "1",

### Show shards

```
$ curl -X GET "http://localhost:9200/_cat/shards?v&pretty"
```

```
index shard prirep state   docs store ip            node
user  0     p      STARTED    0  230b 192.168.160.4 elasticsearch_3
user  0     r      STARTED    0  230b 192.168.160.3 elasticsearch_2
```

### Show allocation

```
$ curl -X GET "http://localhost:9200/_cat/allocation?v&pretty"
```

```
shards disk.indices disk.used disk.avail disk.total disk.percent host          ip            node
     0           0b   169.7gb     63.7gb    233.4gb           72 192.168.160.2 192.168.160.2 elasticsearch_1
     1         230b   169.7gb     63.7gb    233.4gb           72 192.168.160.3 192.168.160.3 elasticsearch_2
     1         230b   169.7gb     63.7gb    233.4gb           72 192.168.160.4 192.168.160.4 elasticsearch_3
```

### Get all data

```
$ curl -X GET http://localhost:9200/user/_search
```

```
{"took":125,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":2,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"user","_type":"_doc","_id":"M1lDpHABKBUa_Fhbupkm","_score":1.0,"_source":{
  "name" : "Kota",
  "age" : 24
}},{"_index":"user","_type":"_doc","_id":"NFlFpHABKBUa_FhbNJn-","_score":1.0,"_source":{
  "name" : "Tanaka",
  "age" : 48
}}]}}
```

>"took":125

### Change number of primary shards

*Can't update non dynamic settings*

```
$ curl -X PUT 'localhost:9200/user/_settings' -H 'Content-Type: application/json' -d'
{
  "index" : {
    "number_of_shards" : 3
  }
}'
```

```
{"error":{"root_cause":[{"type":"illegal_argument_exception","reason":"Can't update non dynamic settings [[index.number_of_shards]] for open indices [[user/CojPNu9SRwqx6qfPCrNbOg]]"}],"type":"illegal_argument_exception","reason":"Can't update non dynamic settings [[index.number_of_shards]] for open indices [[user/CojPNu9SRwqx6qfPCrNbOg]]"},"status":400}
```

### Change number of replica shards

```
$ curl -X PUT 'localhost:9200/user/_settings' -H 'Content-Type: application/json' -d'
{
  "index" : {
    "number_of_replicas" : 2
  }
}'
```

```
{"acknowledged":true}
```

### Show added replica

```
$ curl -X GET "http://localhost:9200/_cat/shards?v&pretty"
```

```
index shard prirep state   docs store ip            node
user  0     p      STARTED    2 3.4kb 192.168.160.4 elasticsearch_3
user  0     r      STARTED    2 3.4kb 192.168.160.3 elasticsearch_2
user  0     r      STARTED    2 3.4kb 192.168.160.2 elasticsearch_1
```

### Get all data

```
$ curl -X GET http://localhost:9200/user/_search
```

```
{"took":5,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":2,"relation":"eq"},"max_score":1.0,"hits":[{"_index":"user","_type":"_doc","_id":"M1lDpHABKBUa_Fhbupkm","_score":1.0,"_source":{
  "name" : "Kota",
  "age" : 24
}},{"_index":"user","_type":"_doc","_id":"NFlFpHABKBUa_FhbNJn-","_score":1.0,"_source":{
  "name" : "Tanaka",
  "age" : 48
}}]}}
```

>"took":5

### Create index with custom setting (number of shards)

```
$ curl -X PUT 'localhost:9200/goods' -H 'Content-Type: application/json' -d'
{
  "settings" : {
    "index" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 2
    }
  }
}'
```

```
{"acknowledged":true,"shards_acknowledged":true,"index":"goods"}
```

### Show setting of `goods`

```
$ curl -X GET "http://localhost:9200/goods/_settings?pretty"
```

```
{
  "goods" : {
    "settings" : {
      "index" : {
        "creation_date" : "1583304913542",
        "number_of_shards" : "3",
        "number_of_replicas" : "2",
        "uuid" : "TFVxQDuZTJiwpIc07GgNLw",
        "version" : {
          "created" : "7060099"
        },
        "provided_name" : "goods"
      }
    }
  }
}
```

>"number_of_shards" : "3",
>"number_of_replicas" : "2",

### Show Shards

```
$ curl -X GET "http://localhost:9200/_cat/shards?v&pretty"
```

```
index shard prirep state   docs store ip            node
goods 1     r      STARTED    0  230b 192.168.160.4 elasticsearch_3
goods 1     p      STARTED    0  230b 192.168.160.3 elasticsearch_2
goods 1     r      STARTED    0  230b 192.168.160.2 elasticsearch_1
goods 2     r      STARTED    0  230b 192.168.160.4 elasticsearch_3
goods 2     r      STARTED    0  230b 192.168.160.3 elasticsearch_2
goods 2     p      STARTED    0  230b 192.168.160.2 elasticsearch_1
goods 0     p      STARTED    0  230b 192.168.160.4 elasticsearch_3
goods 0     r      STARTED    0  230b 192.168.160.3 elasticsearch_2
goods 0     r      STARTED    0  230b 192.168.160.2 elasticsearch_1
user  0     p      STARTED    2 3.4kb 192.168.160.4 elasticsearch_3
user  0     r      STARTED    2 3.4kb 192.168.160.3 elasticsearch_2
user  0     r      STARTED    2 3.4kb 192.168.160.2 elasticsearch_1
```
