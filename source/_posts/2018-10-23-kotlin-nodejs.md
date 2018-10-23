---
title: Kotlin + Node.js 搭建教程
date: 2018-10-23 15:04:21
categories: 技术分享
tags:
- Kotlin
- Node.js
- JavaScript
---

`Kotlin`是JetBrains推出的一款语言, 相比`Java`有更简洁的语法, 能编译为`Java Class`, 也能编译为`JavaScript`
`Node.js`则是可以运行在服务端的`JavaScript`, 这里把二者结合, 搭建一个用`Kotlin`编写的服务端应用

# 创建
打开Idea 创建一个 Kotlin(JavaScript) 项目
编写一个测试文件, 检查是否可以正常编译
## Test.kt
```Java
fun main(args: Array<String>) {
    println("hello kt")
}
```
按`Ctrl+F9`编译, 如果看到生成了编译文件, 就可以了, 其中`{projectName}.js`就是编译后的文件, 打开可以看到已经被编译为`JavaScript`了, 其中也有`println('hello kt');`
如果没问题的话就可以正式开始接下来的了, 创建`App.kt`
## App.kt
监听`8888`端口 对任何请求都返回`hello world`
```Java
import kotlin.js.json

external fun require(module: String): dynamic

fun main(args: Array<String>) {
    println("hello kt")
    val http = require("http")

    http.createServer { _, response ->
        response.writeHead(200, json("Content-Type" to "text/plain"))
        response.end("Hello World")
    }.listen(8888)
}
```
# NPM
打开终端运行
```
$ npm init
```
## package.json
```JSON
{
  "name": "kt-node",
  "version": "1.0.0",
  "description": "kt-node",
  "scripts": {
    "start": "node ./out/production/kt-node/kt-node.js"   //这里改成你编译后文件的位置
  },
  "author": "laziji",
  "dependencies": {
    "express": "^4.15.4",
    "kotlin": "^1.1.4",
    "mongoose": "^4.11.7"
  }
}
```

```
$ npm install
$ npm start
```

打开`localhost:8888` 查看效果

# 若有报错
如果运行的时候报错
打开`project settings -> Kotlin Complier` 
将 `Module kind` 改为 `UMD` 再尝试编译 运行
