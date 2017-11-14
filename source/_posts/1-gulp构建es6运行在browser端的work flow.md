---
title: 'gulp构建 es6运行在browser端 自动化工作流'
date: 2017-11-10 22:23:52
tags:
---

### 用来做什么?

1.支持监视前端页面js,css,ejs模板文件热更新,babel语法自动转换,自动刷新浏览器页面,

2.支持后台静态资源的处理,js文件的压缩打包,更新静态css与ejs模板文件,也可以在express端构建逻辑mock数据

----------
### 项目自动化构建思路
![1](https://ooo.0o0.ooo/2017/11/14/5a0a30c88f806.png)
### 自动化构建逻辑
1. 若app文件夹(前端静. 态页面)资源发生改变 ->
2. 调用browser.js脚本 ->
3. browser.js运行script脚本 ->
4. 将新的js文件打包后写入server目录public目录下 ->
5. 此行为触发server.js监听到服务端js静态资源文件被修改 ->
6. 执行服务器重启重新渲染页面 ->
7. 前台看到浏览器热更新
### 构建目录结构,安装服务端脚手架工具
1. 创建项目目录 `app`(放置静态页面资源),`server`(用express脚手架初始化,将来放入热更新后的静态资源),`tasks`放置所有上述过程的脚本文件
2. 初始化服务端express
		`npm install -g express-generator`  //安装express脚手架
		`express -e .` //使用express脚手架命令,初始化脚手架,-e代表使用ejs模板引擎
3. 在根目录创建 .babelrc文件,此为babel转码器
4. 在根目录创建`gulpfile.js`, 若用es6语法写gulpfile文件就创建`gulpfile.babel.js`


----------

## gulp工作流代码构建流程


### args.js --处理命令行参数
``` javascript
import yargs from 'yargs'; //处理命令行参数的包
//区分开发环境和线上环境
const args = yargs  
  //提取--production参数
  .option('production',{ 
    boolean:true,//选项是布尔类型
    default:false,//默认是false
    describe:'min all scripts' //只是给人看的描述
  })
  //用来监听文件改变的选项
  .option('watch',{
    boolean:true,
    default:false,
    describe:'watch all files'
  })
  //要不要输出命令行详细监视的日志
  .option('verbose',{
    boolean:true,
    default:false,
    describe:'log'
  })
  //强制生成sourcemaps映射
  .option('sourcemaps',{
    describe:'force the creation of sroucemaps'
  })
  //端口号
  .option('port',{
    string:true,
    default:8080,
    describe:'server port'
  })
  //表示把输入的命令以字符串的方式进行解析
  .argv

export default args;
```
### script.js --处理js
``` javascript
import gulp from 'gulp';
import gulpif from 'gulp-if'; //gulp语句中做if判断
import concat from 'gulp-concat'; //gulp中处理文件拼接
import webpack from 'webpack';
import gulpWebpack from 'webpack-stream'; //支持webpack在gulp stream中的功能
import named from 'vinyl-named'; //保证webpack生成的文件名能够和原文件对上
import livereload from 'gulp-livereload'; //浏览器热更新
import plumber from 'gulp-plumber'; //处理文件信息流
import rename from 'gulp-rename'; //对文件重命名
import uglify from 'gulp-uglify'; //压缩js
import {log, colors} from 'gulp-util'; //命令行工具包,log与色彩输出
import args from './util/args'; //刚自己写的对命令行参数进行解析的包

// 为了集中处理项目js文件抛出异常引起gulp流出现问题,需用plumber统一处理错误
gulp.task('scripts', () => {
  return gulp
    .src(['app/js/index.js'])
    .pipe(plumber({errorHandle: function () {}}))
    .pipe(named())
    .pipe(gulpWebpack({
      module: {
        loaders: [
          {
            test: /\.js$/,
            loader: 'babel-loader'
          }
        ]
      }
    }), null, (err, stats) => {
      log(`Finished '${colors.cyan('scripts')}'`, stats.toString({chunks: false}))
    })
    //gulp处理完的js指定写入路径,api:gulp.dest
    .pipe(gulp.dest('server/public/js'))
    //js文件重命名为cp.min.js,还没压缩,只是复制一份
    .pipe(rename({basename: 'cp', extname: '.min.js'}))
    // 压缩
    .pipe(uglify({
      compress: {
        properties: false
      },
      output: {
        'quote_keys': true
      }
    }))
    // 把压缩后的文件放入服务器目录
    .pipe(gulp.dest('server/public/js'))
    // 使用gulpif监视命令行传入的参数,若有--watch,则执行热更新
    .pipe(gulpif(args.watch, livereload()))
})
```
### pages.js --处理后台页面模板
``` javascript
import gulp from 'gulp';
import gulpif from 'gulp-if';
import livereload from 'gulp-livereload';
import args from './util/args';

gulp.task('pages',()=>{
  return gulp.src('app/**/*.ejs')
    .pipe(gulp.dest('server')) //文件被写入的路径是以所给的相对路径根据所给的目标目录计算而来。类似的，相对路径也可以根据所给的 base 来计算。这里实际写到的路径是server下的/**/*.ejs,即server/views/*.ejs
    .pipe(gulpif(args.watch,livereload()))
})

```
### css.js --处理css
``` javascript
import gulp from 'gulp';
import gulpif from 'gulp-if';
import livereload from 'gulp-livereload';
import args from './util/args';

gulp.task('css',()=>{
  return gulp.src('app/**/*.css')
    .pipe(gulp.dest('server/public'))
    //文件被写入的路径是以所给的相对路径根据所给的目标目录计算而来。类似的，相对路径也可以根据所给的 base 来计算。这里实际写到的路径是server下的/**/*.css,即server/public/css/*.css
    .pipe(gulpif(args.watch,livereload()))
})
```

### server.js --处理服务端热重启
``` javascript
import gulp from 'gulp';
import gulpif from 'gulp-if';
import liveserver from 'gulp-live-server'; //启动gulp服务器的包
import args from './util/args';

gulp.task('serve',(cb)=>{
  //如果没在监听,直接运行回调函数
  if(!args.watch) return cb(); 
  //启动express脚手架默认的服务器脚本
  const server = liveserver.new(['--harmony','server/bin/www']); 
  server.start();
  //监听server目录下的js文件和ejs模板文件,通知服务器哪些文件改变了
  gulp.watch(['server/public/**/*.js','server/views/**/*.ejs'],function(file){
    server.notify.apply(server,[file]);
  })
  //监视服务器路由及入口文件的改变,进行服务器重启
  gulp.watch(['server/routes/**/*.js','server/app.js'],function(){
    server.start.bind(server)()
  });
})

```

### browser.js --监视前端文件改变,触发热更新
``` javascript
import gulp from 'gulp';
import gulpif from 'gulp-if';
import gutil from 'gulp-util';
import args from './util/args';

gulp.task('browser',(cb)=>{
  if(!args.watch) return cb();//若没监听,则直接执行回调
  gulp.watch('app/**/*.js',['scripts']);//若js文件发生改变,则调用刚才创建的scripts脚本
  gulp.watch('app/**/*.ejs',['pages']); //同上
  gulp.watch('app/**/*.css',['css']); //同上
});

```

### clean.js --处理服务器清除旧文件
每次服务器监听到静态资源文件的改变, 会触发重启, 用新的静态资源去render页面,此时需要删除旧的静态资源文件
``` javascript
import gulp from 'gulp';
import del from 'del';
import args from './util/args';
//清除服务端的静态资源文件和模板文件
gulp.task('clean',()=>{
  return del(['server/public','server/views'])
})
```

### build.js --处理所有gulp文件运行关联顺序
``` javascript
import gulp from 'gulp';
import gulpSequence from 'gulp-sequence'; //处理文件关联关系和先后顺序

//先clean,再css,再pages,再编译js,最后一个数组说明数组里的任务都放在前面四个任务执行过一次之后再执行,且serve端更新一定 在 browser静态资源改变之后 
gulp.task('build',gulpSequence('clean','css','pages','scripts',['browser','serve']));
```

### default.js --gulp工作流默认入口
gulp在无命令行参数时会优先运行defalut.js
``` javascript
import gulp from 'gulp';
gulp.task('default',['build']);
```

### gulpfile.babel.js --gulp程序入口 
``` javascript
import requireDir from 'require-dir';//需要运行某一文件夹的gulp任务
requireDir('./tasks'); //放入tasks目录
```

### 编辑.babelrc
``` json
{
  "presets":["env","stage-0"],
  "plugins":["transform-decorators-legacy"]
}

```

### 给express脚手架添加热更新中间件
``` javascript
//在处理路由前,express优先处理静态资源,若在这个static方法定义的目录中没有找到req.url对应的静态资源,则调用Next()方法传入下一个中间件,最终会传递到路由中间件上
app.use(express.static(path.join(__dirname, 'public')));
//一定要再静态资源设置之后使用热更新中间件,此插件的安装要再最外层项目目录下的依赖安装,而不是server目录的依赖
app.use(require('connect-livereload')());
```

### gulp程序启动
`gulp --watch`
在app/public/js下写一个简单的js,再在app/public/css下写一个chotee.css,例如

``` javascript
class Chotee{
  constructor(){
    this.name = 'chotee 啊,成功!'
  }
}

const ct1 = new Chotee

document.body.innerHTML = ct1.name;
```
``` css
body {
  background-color: pink;
}
```

在app/views/目录下的index.ejs模板中引入
``` html 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <link rel="stylesheet" href="/css/chotee.css">
</head>

<body>
  hello chotee
  <script src="/js/index.js" charset="utf-8"></script>
</body>
</html>
```

打开浏览器访问localhost:3000端口(express脚手架默认端口), 看到
![2](https://i.loli.net/2017/11/14/5a0a30b96f6c1.png)

成功! 此时再去修改js文件,模板文件,热更新文件,成功

## 源码地址:
[https://github.com/choteewang/gulp-es6-work-flow](https://github.com/choteewang/gulp-es6-work-flow)