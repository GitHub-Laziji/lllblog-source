---
title: vue-cli构建库
date: 2019-12-28 11:00:43
categories: 环境部署
tags:
- Vue
---

使用`vue-cli-service build`对开发的库进行打包时使用`--target lib`

开发时Vue通常是以`devDependencies`方式引入的只在开发中使用, 使用`lib`方式打包也一样不会把Vue打包进去

> 官方文档中描述
>
> 在库模式中，Vue 是外置的。这意味着包中不会有 Vue，即便你在代码中导入了 Vue。


# 打包
打包时通常会提示缺少`vue-template-compiler` 按以下安装并打包, 打包默认输出路径是`./dist`
```
npm install -g @vue/cli-service
npm install -g vue-template-compiler

vue-cli-service build --target lib --name myLib [entry]
```

# 配置
在`vue.config.js`中进行打包相关的配置
```js
module.exports = {
  //不生成 .map 文件
  productionSourceMap: false,
  css: {
    //css合并入js中, 实际使用中发现合并的时候会自动去掉空的class, 导致$style.class 取不到值
    extract: false
  }
}
```

# 引入
发布到`npm`上后, 若需要CDN方式引入, 可以使用`unpkg` 例如这个项目
[https://github.com/GitHub-Laziji/menujs](https://github.com/GitHub-Laziji/menujs)
```html
<script src="https://unpkg.com/vue-contextmenujs/dist/contextmenu.umd.js">
```