title: 安卓端微信浏览器不支持promise
date: 2016-03-31 11:55:26
tags: [javascript]
---

# 关于微信不支持es6的promise特性的解决方案

## 原因

  最近使用es6的promise特性重构了一下微信端的点餐系统。完成后发现安卓版本的微信不能正常运作，而itouch却可以正常运行。
  于是开始了bug调试踩坑的旅程...
    
  刚开始发现程序不能运行正确，但是微信的调试工具并没有报任何的错误，我以为是缓存问题，想进了一切办法去解决缓存问题，然并软。
  然后发现VueComponent并没有得到正确的解析，于是探索为何dom节点没有被Component替换，这个坑踩了三个多小时，最后才想起是不是
  es6不支持的问题。。。

  于是,进行验证，发现只有使用了promise相关的component没有被正确解析，而未使用promise的组件解析正常，才确定下来是es6特性不支持的问题。


## 解决方案

  引入polyfill支持es6的promise特性

  ```
    npm install es6-promise --save
    # use
    var Promise = require('es6-promise').Promise

  ```
  重新打包生成，运行正确

## 总结

  类似的场景应该会非常常见，引入javascript新版本的高级特性确实能有效的提高开发效率，但是需要考虑低版本的浏览器支持的问题，
  尽量去引用一些polyfill去支持这些特性，不然调试微信这样的bug确实是非常浪费时间一件事。


  
