---
layout:     post
title:      "golang js 之间 时间戳 统一"
subtitle:   ""
date:       2019-07-14 1:00:00
author:     "GXKe"
#header-img: ""
catalog:      true
tags:
    - 时间戳
    - GO
    - JS
---

##### golang js 时间戳标准

> golang服务与js交互时，时间戳表达格式不同，容易导致精度或兼容性问题
>
> 统一使用‘TimeJsISOFormat = "2006-01-02T15:04:05.999Z07:00"’格式

###### 1. 定义go端时间对象，JSTime

- 支持api字段自动解析
- 兼容ORM

```
package utils

import (
	"time"
)

type JSTime time.Time

const (
	TimeApiFormart  = "2006-01-02 15:04:05"
	TimeDataFormat  = "2006-01-02"
	TimeJsISOFormat = "2006-01-02T15:04:05.999Z07:00"
)

func (t *JSTime) UnmarshalJSON(data []byte) (err error) {
	now, err := time.ParseInLocation(`"`+TimeJsISOFormat+`"`, string(data), time.Local)
	*t = JSTime(now)
	return
}

func (t JSTime) MarshalJSON() ([]byte, error) {
	b := make([]byte, 0, len(TimeJsISOFormat)+2)
	b = append(b, '"')
	b = time.Time(t).AppendFormat(b, TimeJsISOFormat)
	b = append(b, '"')
	return b, nil
}

func (t JSTime) String() string {
	return time.Time(t).Format(TimeJsISOFormat)
}

```

###### 2.JSTime对象使用


```

type Card struct {
	f utils.JSTime

	Created utils.JSTime `xorm:"created "`
	Updated utils.JSTime `xorm:"updated "`
}
```

###### 3.JS前端字段处理


```
new Date(Date.now()).toISOString()  <!--  "2006-01-02T15:04:05.999Z07:00"   -->
```
