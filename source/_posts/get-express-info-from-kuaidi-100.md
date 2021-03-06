title: 简单的从快递 100 中获取快递信息
date: 2016-07-27 13:24:19
tags: [php, laravel]
---

# 快递查询

经常有应用需求能根据单号查询快递状态，大多数时候都需求快递的运输信息。我们可以从 [快递 100](http://www.kuaidi100.com/) 中快速的获取到快递的物流信息。

## 接口分析

快递 100 中提供了两个接口用来结合查询物流信息：

- 根据单号智能识别物流方式（顺丰、韵达、申通等）
- 根据单号及物流方式获取物流信息

### 智能识别接口

接口地址： http://www.kuaidi100.com/autonumber/autoComNum?text=单号
请求方式： GET

成功返回：

``` js
    {
      "comCode": "",
      "num": "3310479082803",
      "auto": [
        {
        "comCode": "shentong",
        "id": "",
        "noCount": 94165,
        "noPre": "331047",
        "startTime": ""
        }
      ]
    }
```

如果是不能识别的单号那么将返回：

``` js
    {
      "comCode": "",
      "num": "xxx51017908280",
      "auto": []
    }
```

### 物流详情接口

接口地址： http://www.kuaidi100.com/query?type=comCode&postid=单号&id=1&valicode=&temp=时间戳
请求方式： GET

接口返回信息：

``` js
    {
        "message": "ok",
        "nu": "3310479082803",
        "ischeck": "0",
        "com": "shentong",
        "updatetime": "2016-07-27 13:15:45",
        "status": "200",
        "condition": "00",
        "data": [
            {
                "time": "2016-06-30 03:17:28",
                "location": "",
                "context": "快件已到达 湖北武汉武隆分部",
                "ftime": "2016-06-30 03:17:28"
            },
            {
                "time": "2016-06-29 16:35:02",
                "location": "",
                "context": "由 浙江杭州中转部 发往 安徽黄山公司",
                "ftime": "2016-06-29 16:35:02"
            },
            {
                "time": "2016-06-29 16:22:00",
                "location": "",
                "context": "由 湖北武汉航空部 发往 湖北武汉武隆分部",
                "ftime": "2016-06-29 16:22:00"
            },
            {
                "time": "2016-06-28 23:09:29",
                "location": "",
                "context": "由 湖北武汉航空部 发往 浙江杭州中转部",
                "ftime": "2016-06-28 23:09:29"
            },
            {
                "time": "2016-06-28 20:03:09",
                "location": "",
                "context": "由 湖北武汉航空部 发往 浙江杭州中转部",
                "ftime": "2016-06-28 20:03:09"
            },
            {
                "time": "2016-06-28 20:03:09",
                "location": "",
                "context": "湖北武汉航空部 正在进行 装袋 扫描",
                "ftime": "2016-06-28 20:03:09"
            },
            {
                "time": "2016-06-28 20:02:52",
                "location": "",
                "context": "由 湖北武汉航空部 发往 浙江杭州中转部",
                "ftime": "2016-06-28 20:02:52"
            },
            {
                "time": "2016-06-28 19:58:24",
                "location": "",
                "context": "快件已到达 湖北武汉航空部",
                "ftime": "2016-06-28 19:58:24"
            },
            {
                "time": "2016-06-28 17:12:28",
                "location": "",
                "context": "湖北武汉公司(027-83225888) 的收件员 汉口吴燕飞 已收件",
                "ftime": "2016-06-28 17:12:28"
            }
        ],
        "state": "0"
    }
```

异常返回：

``` js
    {
      "status": "403",
      "message": "快递公司参数异常：单号格式错误"
    }
```
