---
title: "Elastic Search"
date: 2022-04-21T16:08:17+08:00
draft: true
---

```
POST tianjinhepingkindergarten*_*/_delete_by_query?slices=10&conflicts=proceed&wait_for_completion=false
{
    "query" : {
        "range" : {
            "create_time" : {
                "lte": 1626278400
            }
        }
    }
}


/* 2天数据删除*/
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:569348247

/* 1个月数据删除 耗时21h */
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:606204269

/* 删除3.17 - 6.17日之间的数据*/
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:672045970

/* 删除6.17 - 7.17日之间的数据*/
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:1058446833

/* 删除7.17 - 8.30日之间的数据*/
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:1109520583

/* 删除10月1日前的数据*/
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:1299975320

/* 删除2021年3月1日前的数据*/
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:78818309

/* 删除2021年5月1日前的数据 */
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:101483769

/* 删除2021年7月15日前的数据 */
GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:133345102

GET _tasks/aJJAVdwMQB6Vvz6VFq35iA:214953924

GET _tasks

GET tianjinhepingkindergarten*_*/_search?size=5
{
    "query" : {
       "bool" : {}
    },
    "sort": { "create_time": { "order": "asc" }}
}

GET _cat/tasks
```