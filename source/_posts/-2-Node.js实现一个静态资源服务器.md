---
title: 'Node.js实现一个静态资源服务器'
date: 2017-11-8 23:24:52
tags:
---

### 这是什么?
原生Node.js写的静态资源服务器,借助浏览器窗口实现计算机任意目录的结构显示

### 启动方法
1. 命令行程序入口处要加 #!/usr/bin/env node,之后可能需要修改此node文件权限,如`chmod 755 app.js`
2. 在package.json中加入字段
``` json
"bin": {
    "choteeserver": "app.js"
  },
```
3. 在命令行执行`npm link`,mac未设置管理员权限要加`sudo`
4. 在任意命令行目录下键入 `choteeserver ./`,浏览器访问`127.0.0.1/3000`

### 源码地址
[https://github.com/choteewang/Nodejs_static_server](https://github.com/choteewang/Nodejs_static_server)
### 关于Node.js命令行工具的参考资料
参考资料来自阮一峰博客 [Node.js 命令行程序开发教程](http://www.ruanyifeng.com/blog/2015/05/command-line-with-node.html)