---
title: '基于Node.js,Express,MongoDB的支持多用户登录,markdown字符串转换的留言板'
date: 2017-11-8 23:25:52
tags:
---

### 技术栈
- 后端
	* Node.js
	* Express 
	* MongoDB 
	* mongoose 
	* nunjucks 
	* bodyParser 
	* session 
	* marked 
- 前端
	* bootstrap 
	* jQuery

### 特点
- MVC结构分层,公共部分的抽象提取,数据与逻辑的分离
- ES6语法全覆盖,异步API Promise化
- 服务端session的业务逻辑实现
- Express中间件思路统一处理404与业务逻辑error

### 源码地址
[https://github.com/choteewang/express_guest](https://github.com/choteewang/express_guest)
