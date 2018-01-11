### 启动 elasticsearch
./elasticsearch
    http://localhost:9200/?pretty 查看是否已经启动成功
### 修改一些基础配置 elasticsearch.yml

### 启动kibana
./kibana 

### 修改Kibana配置  kibana.yml
    进入kibana dev_tools http://localhost:5601/app/kibana#/dev_tools/console?_g=()
    
    
### 快速查看集群中有哪些索引
    GET /_cat/indices?v  ?v 显示详细信息

health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .kibana rUm9n9wMRQCCrRDEhqneBg   1   1          1            0      3.1kb          3.1kb

### 简单的索引操作

    创建索引：PUT /test_index?pretty
    删除索引：DELETE /test_index?pretty



### 4、商品的CRUD操作

    （1）新增商品：新增文档，建立索引

        PUT /index/type/id
        {   
             "json数据"
        }   
    es会自动建立index和type，不需要提前创建，而且es默认会对document每个field都建立倒排索引，让其可以被搜索
        

    PUT /ecommerce/product/1
    {
        "name" : "gaolujie yagao",
        "desc" :  "gaoxiao meibai",
        "price" :  30,
        "producer" :      "gaolujie producer",
        "tags": [ "meibai", "fangzhu" ]
    }

    PUT /ecommerce/product/2
    {
        "name" : "jiajieshi yagao",
        "desc" :  "youxiao fangzhu",
        "price" :  25,
        "producer" :      "jiajieshi producer",
        "tags": [ "fangzhu" ]
    }

    PUT /ecommerce/product/3
    {
        "name" : "zhonghua yagao",
        "desc" :  "caoben zhiwu",
        "price" :  40,
        "producer" :      "zhonghua producer",
        "tags": [ "qingxin" ]
    }



    （2）查询商品：检索文档

        GET /index/type/id
        GET /ecommerce/product/1

    （3）修改商品：替换文档

    PUT /ecommerce/product/1
    {
        "name" : "jiaqiangban gaolujie yagao",
        "desc" :  "gaoxiao meibai",
        "price" :  30,
        "producer" :      "gaolujie producer",
        "tags": [ "meibai", "fangzhu" ]
    }

    PUT /ecommerce/product/1
    {
        "name" : "jiaqiangban gaolujie yagao"
    }
    
    GET /ecommerce/product/1
    
    {
      "_index": "ecommerce",
      "_type": "product",
      "_id": "1",
      "_version": 2,
      "found": true,
      "_source": {
        "name": "jiaqiangban gaolujie yagao"
      }
    }

    替换方式有一个不好，即使必须带上所有的field，才能去进行信息的修改

    （4）修改商品：更新文档

    POST /ecommerce/product/1/_update
    {
      "doc": {
        "name": "jiaqiangban gaolujie yagao"
      }
    }


    （5）删除商品：删除文档

        DELETE /ecommerce/product/1



   