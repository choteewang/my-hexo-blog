---
title: 'React,AntDesign 实现一个电影网站'
date: 2017-11-9 08:25:52
tags:
---

![Jietu20171108-184502](https://ooo.0o0.ooo/2017/11/14/5a0a42c594cd5.png)

![Jietu20171108-184517](https://i.loli.net/2017/11/14/5a0a42d1837f7.png)
---

### HOW TO USE?

1. open terminal in rootfolder
2. run 'npm install'
3. open termianl in 'react-movie-server' folder
4. run 'nodemon app.js'
5. open termianl in 'react-movie-front' folder
6. run 'npm run dev'
7. if browser can't open automatically, type '127.0.0.1:3000' & enter

---

### TECHNIQUES
- React
- Ant Design
- Fetch API
- Full ES6 syntax cover
- Promise handle fetch async
- React Router 4.0+
- Node.js + Express Server handle CORS & data fetching
- Webpack

---
### 源码地址
[https://github.com/choteewang/react-movie-intro](https://github.com/choteewang/react-movie-intro)
---
### SUMMARY

- Node.js设置CORS跨域

``` javascript
// 在Node.js中设置允许跨域请求数据
// 这里指浏览器端localhost:3000端口运行的前端 向 localhost:3001运行的服务端进行ajax请求时的跨域处理
app.use('*', function (req, res, next) {
  // 设置请求头为允许跨域
  res.header("Access-Control-Allow-Origin", "*");
  // 设置服务器支持的所有头信息字段
  res.header("Access-Control-Allow-Headers", "Content-Type,Content-Length, Authorization, Accept,X-Requested-With");
  // 设置服务器支持的所有跨域请求的方法
  res.header("Access-Control-Allow-Methods", "POST,GET");
  // next()方法表示进入下一个路由
  next();
});

```

- Promise + Fetch API 封装

``` javascript
// 模块导出
export default {
  getmovielist: (type) => {
    var url = 'http://127.0.0.1:3001/getmovielist?type=' + type
    // fetch API https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API 
    return new Promise((resolve,reject) => {
      fetch(url).then((response) => {
      // fetch(url)拿到一个promise对象,对象的resolve方法中传入的data是一个response对象.
        return response.json();
      }).then((data) => {
      // 此response对象上有各种方法,.json方法返回一个响应的json格式的Promise对象,再then一次可以拿到此json对象的data
        resolve(data)
      })
    })   
  }
}
```

``` javascript
   //外层调用
	import GetMovie from ''
    
    GetMovie.getmovielist(this.state.type).then((data) => {
      this.setState({
        movieList: data.subjects,
        isLoading: false
      })
    });
```

- MovieList组件,主要业务逻辑

``` javascript
import React, { Component } from 'react';
import { Spin, Alert,Rate } from 'antd';
// 导入自定义模块GetMovie
import GetMovie from '../services/GetMovie'
import '../css/movieList.css'
// https://api.douban.com/v2/movie/in_theaters
// https://api.douban.com/v2/movie/coming_soon
// https://api.douban.com/v2/movie/top250
export default class About extends Component {
  // constructor
  constructor(props) {
    super(props)
    this.state = {
      type: this.props.match.params.type,
      movieList: [],
      isLoading: true
    }
  }
  // render
  render() {
    return <div>
      {this.renderMovieList()}
    </div>
  }
  // renderMovieList
  renderMovieList = () => {
    if (this.state.isLoading) {
      return <Spin tip="加载中...">
        <Alert
          message="正在加载豆瓣API电影列表"
          description="请耐心等待."
          type="info"
        />
      </Spin>
    } else {
      return <div style={{ display:'flex',flexWrap:'wrap',textAlign:'center'}}>
        {this.state.movieList.map((value, index) => {
          return <div className="movielist" key={index} style={{border:'1px solid #ddd',margin:'5px',padding:'8px 5px',width:'180px',height:'260px',cursor:'pointer'}} onClick={ ()=>{
            this.toDetail(value.id)
          }}>
            <img src={value.images.medium} alt={value.tittle} width="100" height="140" />
            <h5>{value.title}</h5>
            <p><strong>电影类型：</strong>{value.genres.map((value,index) => {
              return <span key={index} style={{ padding:'2px',background:'#bbb',marginRight:'2px',color:'#fff'}}>{value}</span>
            })}</p>
            <p><strong>上映年份：</strong>{value.year}年</p>
            <div><strong>评分：</strong><Rate disabled defaultValue={value.rating.average / 2} /></div>
          </div>
        })}
      </div>
    }
  }
  // cwrp
  componentWillReceiveProps(nextProps) {
    this.setState({
      type: nextProps.match.params.type,
      isLoading:true
    },this.getMovieListByType)
  }
  // cdm
  // 这里需要获取电影列表的原因是,若是从1级路由重定向到in_theaters时,不会进入cwrp,不会去拿数据
  componentDidMount() {
    this.getMovieListByType()
  }
  // getMovieListByType
  getMovieListByType = () => {
    GetMovie.getmovielist(this.state.type).then((data) => {
      this.setState({
        movieList: data.subjects,
        isLoading: false
      })
    });
  }
  // toDetail
  toDetail = (id) => {
    this.props.history.push('/movie/detail/' + id)
  }
}
```