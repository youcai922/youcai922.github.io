```json
#查询所有索引信息
GET _search
{
  "query": {
    "match_all": {}
  }
}

#创建商品索引 product 指定mapping{id,title,price}
PUT /product
{
  "settings": {
    "number_of_replicas": 1
    , "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "id":{
        "type": "integer"
      },
      "title":{
        "type": "keyword"
      },
      "price":{
        "type": "double"
      },
      "create":{
        "type": "date"
      },
      "descrition":{
        "type": "text"
      }
    }
  }
}

#查看某个索引的映射信息
GET /product/_mapping

#删除索引
DELETE product

#在索引product下添加文档
POST /product/_doc/1
{
  "id":"1",
  "title":"iphone",
  "price":"9999",
  "created_at":"2018-6-6",
  "description":"华为P30"
}

#查询索引下的所有信息
GET /product/_search
{
  "query": {
    "match_all": {}
  }
}

#基于id文档查询
GET /product/_doc/1

#基于id删除文档
DELETE /product/_doc/1

#更新文档（删除原始文档，再重新添加）
PUT /product/_doc/1
{
  "title": "desktop"
}

#这种更新方式将原始数据内容保存，并在此基础上更新
POST /product/_doc/1/_update 
{
  "doc":{
    "title":"desktop"
  }
}
```



- 批量操作（如果id之前就存在，则会进行覆盖操作）

```
POST /product/_doc/_bulk
{"index":{"_id":2}}
  {"id":"2","title":"辣条","price": "0.5","created_at":"1999.9.9","description":"delicious"}
{"index":{"_id":5}}
  {"id":"3","title":"鱼豆腐","price": "1","created_at":"2020.10.10","descripion":"delicious"}
```

批量操作也可以是不同类型的操作

```
#更新文档的同时删除文档
POST /product/_doc/_bulk
{"update":{"_id":"1"}}
  {"doc":{"title":"keyboard","price":"30"}}
{"delete":{"_id":2}}
```

- 高级搜索

```
#term 基于关键字查询
#text 默认，标准分词器，单字搜索才能搜索的到
GET /product/_search
{
  "query": {
    "term": {
      "descripion": {
        "value": "delicious"
      }
    }
  }
}

#范围查询
GET /product/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 0.5,
        "lte": 3
      }
    }
  }
}

#前缀查找
GET /product/_search
{
  "query": {
    "prefix": {
      "descripion": {
        "value": "del"
      }
    }
  }
}

#通配符查询 wildcard（*表示多个，？表示一个）
GET /product/_search
{
  "query": {
    "wildcard": {
      "title": {
        "value": "鱼*"
      }
    }
  }
}

#查询一组符合条件的id
GET /product/_search
{
  "query": {
    "ids": {
      "values": [1,3,4]
    }
  }
}

#模糊查询
GET /product/_search
{
  "query": {
    "fuzzy": {
      "title": "吃辣条"
    }
  }
}
```

