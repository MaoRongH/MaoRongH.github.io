---
layout:     post
title:      ElasticSearch 学习
date:       2021-08-18
author:     Maorong
header-img: img/69530.jpg
catalog: true
tags:
    - ElasticSearch
---

### es大致结构

```
{
    "track_total_hits": true, //获取query里面查询到的总条数
    "query":{},//查询语句筛选条件
    "aggs":{},//管道
    "sort":[],//排序
    "_source":[],//字段筛选
    "size":0,//查询的大小
    "from":0//从哪里查
}
```

### 关于query
#### match_phrase 与 match
match_phrased 必须要包含查询的词组，math可以包含查询的词组也可以将词组拆分查询

来源:[https://blog.csdn.net/liuxiao723846/article/details/78365078](https://blog.csdn.net/liuxiao723846/article/details/78365078)

#### terms与term
##### term（精确值查找）
 term查询， 可以用它处理数字（numbers）、布尔值（Booleans）、日期（dates）以及文本（text）
##### trems（查找多个精确值）
terms 是 包含（contains） 操作，而非 等值（equals）

其中 country_number_id 需要使用.keyword
```
{
  "terms": {
    "country_number_id.keyword": [
      "CN",
      "HK",
      "IN",
      "JP",
      "KR",
      "MO"
    ]
  }
}
```

### 重点aggs
返回数据 是从aggregations中获取

示例：
```
"aggs": {
    "country_number_id": {
      "terms": {
        "field": "country_number_id.keyword",
        "size": 10000 //这里指的是需要返回country_number_id的桶的个数，但不代表是查询的数据量
      },
      "aggs": {
        "category_number_id": {
          "terms": {
            "field": "category_number_id.keyword",
            "size": 10000,//同上，不过是category_number_id
            "order": {
              "revenue": "desc"//这个字段用的是下面sum聚合的revenue
            }
          },
          "aggs": {
            "revenue": {
              "sum": {//sum，avg，min，max
                "field": "revenue"
              }
            },
            "downloads": {
              "sum": {
                "field": "downloads"
              }
            },
            "top": {//随便取名
              "top_hits": {//关键字
                "size": 1,//category_number_id下面桶里数据的条数
                "_source": [//里面命中数据字段筛选
                  "os",
                  "revenue",
                  "country_number_id",
                  "category_number_id",
                  "category_name",
                  "country_name",
                  "downloads"
                ]
              }
            }
          }
        }
      }
    }
```

#### 关于aggs的疑惑
1. 多个字段聚合的时候怎么做排序，比如这边用country_number_id和category_number_id聚合，使用sum来的revenue做一个排序，但是实际是country_number_id大桶里面包含小桶，小桶里面可以根据revenue排序，无法做一整个排序
2. 多个字段聚合的时候将两个字段嵌套顺序替换，查出来的数据量变了，其他筛选条件什么都没变

