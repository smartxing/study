### 聚合（Aggregations）分析
#### 根据tag分组
    GET /ecommerce/product/_search
    {
      "size": 0,
      "aggs": {
        "all_tags": {
          "terms": { "field": "tags" }
        }
      }
    }
    类似sql： select tag,count(1) from xx  group by tag
#### 对名称匹配，计算每个tag下的商品数量
    GET /ecommerce/product/_search
    {
      "size": 0,
      "query": {
        "match": {
          "name": "yagao"
        }
      },
      "aggs": {
        "all_tags": {
          "terms": {
            "field": "tags"
          }
        }
      }
    }
    类似sql： select tag,count(1) from xx where name = 'yagao' group by tag
  
#### 先分组 在求组内平均值
      GET /ecommerce/product/_search
      {
          "size": 0,
          "aggs" : {
              "group_by_tags" : {
                  "terms" : { "field" : "tags" },
                  "aggs" : {
                      "avg_price" : {
                          "avg" : { "field" : "price" }
                      }
                  }
              }
          }
      }
      
#### 第四个数据分析需求：计算每个tag下的商品的平均价格，并且按照平均价格降序排序
      
      GET /ecommerce/product/_search
      {
          "size": 0,
          "aggs" : {
              "all_tags" : {
                  "terms" : { "field" : "tags", "order": { "avg_price": "desc" } },
                  "aggs" : {
                      "avg_price" : {
                          "avg" : { "field" : "price" }
                      }
                  }
              }
          
          
          
##### 第五个数据分析需求：按照指定的价格范围区间进行分组，然后在每组内再按照tag进行分组，最后再计算每组的平均价格

    GET /ecommerce/product/_search
    {
      "size": 0,
      "aggs": {
        "group_by_price": {
          "range": {
            "field": "price",
            "ranges": [
              {
                "from": 0,
                "to": 20
              },
              {
                "from": 20,
                "to": 40
              },
              {
                "from": 40,
                "to": 50
              }
            ]
          },
          "aggs": {
            "group_by_tags": {
              "terms": {
                "field": "tags"
              },
              "aggs": {
                "average_price": {
                  "avg": {
                    "field": "price"
                  }
                }
              }
            }
          }
        }
      }
    }