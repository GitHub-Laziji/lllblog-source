---
title: JavaScript实现类似Java的泛型
date: 2018-10-13 02:04:07
tags: JavaScript
---

现在有个需求, 后端对于每一张数据库的表都暴露三个主要API`namespace/list`、`namespace/save`、`namespace/update` 以及其他扩展接口。
现在需要用JavaScript编写一种较通用可扩展的代码。

当然在JavaScript中实现这个需求方法很多, 这里讲述的是如何在`ECMAScript 6`中用类似Java中的泛型, 继承来解决

最后使用如下
```JavaScript
import AppApi from "./AppApi"

AppApi.list()
```

# AppApi.js
Api类如下 十分的OO
```JavaScript
import BaseApi from "./BaseDBApi"

export default class AppApi extends BaseApi("/app") {
    //...extension method
}
```
# BaseApi.js
由于JS动态的特性, 可以比Java更加灵活 动态构造类, static 标注的方法可以直接通过类名, 子类继承后 也可以通过子类的类名调用
```JavaScript
import request from "@/common/request"

export default function builder(baseUrl) {
  return class BaseApi {
    static list(form) {
      return request({
        url: `${baseUrl}/list`,
        data: form
      })
    }

    static save(form) {
      return request({
        url: `${baseUrl}/save`,
        data: form
      })
    }

    static update(form) {
      return request({
        url: `${baseUrl}/update`,
        data: form
      })
    }
  }
}
```
