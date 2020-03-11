# Analyzer を試す

* 全文検索可能なインデックスの作成

```
$ curl -X PUT "http://localhost:9200/member" -H 'Content-Type: application/json' -d @schemas/member.json
```

* マッピング確認

```
$ curl -X GET "http://localhost:9200/member/_mapping?pretty"
```

* アナライザーの設定確認

```
$ curl -X GET "http://localhost:9200/member/_settings?pretty"
```

* アナライザーの機能確認

```
$ curl -X POST 'http://localhost:9200/member/_analyze?pretty' -H 'Content-Type: application/json' \
-d '{
  "analyzer": "ja_text_analyzer",
  "text": "ワチキはﾊﾟﾘﾋﾟ"
}'
```

```
{
  "tokens" : [
    {
      "token" : "ワチキ",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "パリピ",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "word",
      "position" : 2
    }
  ]
}
```

* 検索

```
$ curl -X GET "http://localhost:9200/member/_search?pretty" -H 'Content-Type: application/json' \
-d '{
  "query": {
    "match": {
      "name": "田中"
    }
  }
}'
```

```
$ curl -X GET "http://localhost:9200/member/_search?pretty" -H 'Content-Type: application/json' \
-d '{
  "query": {
    "multi_match": {
      "query": "こうた",
      "fields": [
        "group",
        "catch_phraze"
      ],
      "type": "most_fields"
    }
  }
}'
```

## アナライザー解説

* 文字列を解析する仕組み
* 文字列を解析する各機能 ①分割前フィルター・②トークナイザー・③分割後フィルター を一括りにした概念

```
"analysis": {
  "tokenizer": {
    "ja_text_tokenizer": {
      "type": "kuromoji_tokenizer",     // 形態素解析(kuromoji)を使う
      "mode": "search"                  // 検索のための設定
    }
  },
  "analyzer": {
    "ja_text_analyzer": {
      "tokenizer": "ja_text_tokenizer", // ②コア処理:「ワチキはパリピ」->「ワチキ」「は」「パリピ」
      "type": "custom",
      "char_filter": [                  // ①前処理:「ワチキはﾊﾟﾘﾋﾟ」->「ワチキはパリピ」
        "icu_normalizer"                // 記号・数字・特殊文字などを正規化する
      ],
      "filter": [                       // ③後処理:「ワチキ」「は」「パリピ」->「ワチキ」「パリピ」
        "kuromoji_part_of_speech"       // 形態素解析後に助詞を省く
      ]
    }
  }
}
```

***（１）分割前フィルター**（Char Filter）*

* 生のテキスト（分割を行う前の文字列）に対しての処理
* オプション・複数可
* 例 `全角半角の正規化` `正規表現の適用` など

***（２）トークナイザー**（Tokenizer）*

* 文字列分割に用いる手法
* **必須**
* 例 `kuromoji（形態素解析）` `N-gram（文字数分割）` など
* 上記設定では [analysis-kuromoji-tokenizer](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-kuromoji-tokenizer.html) を使用

***（３）分割後フィルター**（Token Filter）*

* 分割された文字列たちに対しての処理
* オプション・複数可
* 例 `表記揺れの展開` `特定の品詞削除` など
