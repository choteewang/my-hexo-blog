---
title: 'webpack-dev-template Webpack@2.6.1-开发环境-自配参数记录'
date: 2017-11-8 22:23:52
tags:
---

## webpack.config.js
``` javascript
const path = require('path'); 
const webpack = require('webpack');
const htmlWebpackPlugin = require('html-webpack-plugin'); // 自动生成HTMl文件的插件

module.exports = {
  // 通过entry属性，指定入口文件路径
  entry: path.join(__dirname, 'main.js'),
  output: { // 指定打包好文件的出口
    path: path.join(__dirname, 'dist'), // 指定输出路径
    filename: 'bundle.js' // 指定输出的文件名
  },
  module: { // 作用：配置处理第三方文件类型的模块
    rules: [ // 第三方文件的匹配规则
      {test: /\.css$/, use: ['style-loader', 'css-loader']}, // 处理CSS文件的匹配规则
      // 处理sass文件的匹配规则,sass-loader内部依赖node-sass也得安撞
      {test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader']}, 
      //limit后面跟一个图片大小,单位是byte字节,大于这个值保存图片,小于这个值转base64
      //处理图片文件的匹配规则,url-loader内部依赖file-loader也得安装
      {test: /\.(png|jpg|jpeg|gif|bmp)$/, use: 'url-loader?limit=43959'}, 
      {test: /\.(woff|ttf|svg|woff2|eot)$/, use: 'url-loader'}, // 处理字体文件的匹配规则
      //处理es6语法,babel,排除node_modules文件夹
      { test: /\.js$/, use: ['babel-loader'], exclude: /node_modules/ }
    ]
  },
  devServer: { 
    //也可以在package.json的script中的webpack-dev-server命令后面跟--open --port 3000 --hot
    open: true, // 自动打开浏览器
    port: 3000, // 指定端口号
    hot: true // 指定启用热更新
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(), // 使用webpack的热更新插件
    new htmlWebpackPlugin({ // 在内存中自动生成HTMl文件
      template: path.join(__dirname, 'index.html'), // 指定模板文件
      filename: 'index.html' // 指定内存中，生成文件的名称
    })
  ]
}
```

## .babelrc
``` json
{
    "presets":["env","preset-0"],
    "plugins":["transform-runtime"]
}
```

## 源码地址
[https://github.com/choteewang/webpack-dev-template](https://github.com/choteewang/webpack-dev-template)