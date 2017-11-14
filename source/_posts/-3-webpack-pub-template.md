---
title: 'webpack-pub-template Webpack@2.6.1-发布环境-自配参数记录'
date: 2017-11-8 23:23:52
tags:
---

### webpack.config.publish.js

``` javascript 
const path = require('path'); 
const webpack = require('webpack');
const htmlWebpackPlugin = require('html-webpack-plugin'); // 自动生成HTMl文件的插件
const cleanWebpackPlugin = require('clean-webpack-plugin'); // 删除文件夹的webpack插件
const extractTextWebpackPlugin = require('extract-text-webpack-plugin'); // 抽取CSS样式的插件
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin'); // 导入优化压缩CSS文件的插件

module.exports = {
  // 通过entry属性，指定入口文件路径
  // 1. 改造入口
  entry: {
    app: path.join(__dirname, 'main.js'),
    vendors: ['jquery']
  },
  output: { // 指定打包好文件的出口
    path: path.join(__dirname, 'dist'), // 指定输出路径
    filename: 'bundle.js' // 指定输出的文件名
  },
  module: { // 作用：配置处理第三方文件类型的模块
    rules: [ // 第三方文件的匹配规则
      // { test: /\.css$/, use: ['style-loader', 'css-loader'] }, // 处理CSS文件的匹配规则
      {
        test: /\.css$/, use: extractTextWebpackPlugin.extract({
        fallback: 'style-loader',
        use: 'css-loader'
        //publicPath:'../' 若套了别的文件夹引起background问题
      })
      }, // 处理CSS文件的匹配规则
      // { test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }, // 处理sass文件
      {
        test: /\.scss$/, use: extractTextWebpackPlugin.extract({
        fallback: 'style-loader',
        use: ['css-loader', 'sass-loader']
      })
      }, // 处理sass文件的匹配规则
      {test: /\.(png|jpg|jpeg|gif|bmp)$/, use: 'url-loader?limit=43959&name=images/img-[hash:8].[ext]'}, 
      //处理图片文件的匹配规则
      {test: /\.(woff|ttf|svg|woff2|eot)$/, use: 'url-loader?name=fonts/[name].[ext]'}, 
      // 处理字体文件的匹配规则
      {test: /\.js$/, use: 'babel-loader', exclude: /node_modules/} 
      // 处理JS文件的匹配规则, 将node_modules文件夹排除
    ]
  },
  plugins: [
    new htmlWebpackPlugin({ // devserver删除后,不在内存中自动生成HTMl文件,转为在设置的dist目录里生成物理文件
      template: path.join(__dirname, 'index.html'), // 指定模板文件
      filename: 'index.html', // 指定内存中，生成文件的名称
      minify: { // 压缩生成的HTMl文件
        collapseWhitespace: true, // 启用合并空白字符
        removeComments: true, // 移除注释
        removeAttributeQuotes: true // 移除属性上的引号
      }
    }),
    new cleanWebpackPlugin(['dist']), // 每次重新运行发布的时候，都把dist目录删掉，重新生成一份最新的dist目录
    new webpack.optimize.CommonsChunkPlugin({ 
      name: 'vendors', // 在打包时候，通过name属性，从entry入口中，找到指定的属性，然后把这些包抽离为单独的文件
      filename: 'vendors.js' // 指定分离出来的第三方模块的JS文件名
    }),
    new webpack.optimize.UglifyJsPlugin({ // 创建一个压缩混淆JS代码的插件
      compress: { // 压缩代码
        warnings: false // 移除警告
      }
    }),
    new webpack.DefinePlugin({ // 定义当前为项目发布环境
      'process.env.NODE_ENV': '"production"'
    }),
    new extractTextWebpackPlugin('styles.css'), // 创建一个抽取CSS文件的插件，然后传递一个名称进去
    new OptimizeCssAssetsPlugin() // 使用压缩优化CSS的插件
  ]
}
```
 
### .babelrc
``` json
{
    "presets":["env","preset-0"],
    "plugins":["transform-runtime"]
}
```
[https://github.com/choteewang/webpack-pub-template](https://github.com/choteewang/webpack-pub-template)