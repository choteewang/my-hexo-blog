---
title: 'Redux 5: 使用axios 配合redux 实现 异步请求的Redux数据流'
date: 2017-11-14 08:02:52
tags:
---
## 前置知识 axios API
[axios API](https://www.kancloud.cn/yunye/axios/234845)
## CODE

```javascript
// Express Server Code
const express = require('express');
const mongoose = require('mongoose')

//创建express服务
const app = express();
app.get('/data',(req,res) => {
  res.json({hobby:"basketball"})
})

app.listen(3001,() => {
  console.log('chotee server')
})
```

### package.json 配置proxy字段,解决跨域问题
```json
 "proxy":"http://localhost:3001"
```
### Auth.redux.js
``` javascript
// 权限登录的AuthReducer与action creater
import axios from 'axios'
const LOGIN = 'LOGIN'
const LOGOUT = 'LOGOUT'
const GETUSERDATA = 'GETUSERDATA'

//initState
const initState = {
  name: 'choteewang',
  isAuth: false,
  hobby: 'football'
}
//reducer
export const AuthReducer = (state = initState, action) => {
  switch (action.type) {
    case LOGIN:
      return { ...state, isAuth: true }
    case LOGOUT:
      return { ...state, isAuth: false }
    case GETUSERDATA:
      return { ...state, hobby: action.payload }
    default:
      return state
  }
}

//action creater

// 新增支持异步请求的action creater, 返回一个函数,函数参数是store.dispatch与store.getState
export const getUserData = () => {
  // 在return出的函数内进行异步请求
  return (dispatch) => {
    axios.get('/data').then((res) => {
      console.log(res)
      // res是axios的响应数据,格式见下面图
      if (res.status === 200) {
        dispatch(getuserdataAsync(res.data.hobby))
      }
    })
  }
}

export const getuserdataAsync = (data) => {
  return {type: GETUSERDATA, payload: data}
}

export const login = () => {
  return { type: LOGIN }
}

export const logout = () => {
  return { type: LOGOUT }
}
```
![Jietu20171114-083933](https://i.loli.net/2017/11/14/5a0a3b5b0874e.png)

### Auth.js 对应路由/login
``` javascript
import React from 'react';
import { connect } from 'react-redux'
import { login, getUserData } from './Auth.redux';
import { Redirect } from 'react-router-dom';

class Auth extends React.Component {
  constructor(props) {
    super(props)
  }
  componentDidMount() {
    // 组件mount完成后发起异步请求,这里的getUserData是经过react-thunk中间件处理的action creater
    this.props.getUserData()
  }
  render() {
    const redirect = <Redirect to='/dashboard'></Redirect>
    const login = (
      <div>
        <p>{`我叫${this.props.name},我的爱好是${this.props.hobby}`}</p>
        <h3>您没登录,请登录</h3>
        <button onClick={this.props.login}>点我模拟登录</button>
      </div>
    )
    // 若登录了,则跳转到dashboard页面,若没登录,显示登录页面
    return this.props.isAuth ? redirect : login
  }
}

export default Auth = connect(
  //引入AuthReducer的字段给Auth的UI组件作为参数
  (state) => state.AuthReducer,
  // 拿到getUserData的actionCreater
  { login, getUserData }
)(Auth)
```

### config.js 设置全局拦截器 
``` javascript
// 配置全局拦截器的config.js,在根组件页面index.js页面引入
import axios from 'axios'
import { Toast } from 'antd-mobile'

// 添加请求拦截器
axios.interceptors.request.use(function (config) {
  // 在发送请求之前做些什么
  Toast.loading('请求中', 0)
  return config;
}, function (error) {
  // 对请求错误做些什么
  return Promise.reject(error);
});

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
  // 对响应数据做点什么
  setTimeout(function () {
    Toast.hide()
  }, 5000);
  return response;
}, function (error) {
  // 对响应错误做点什么
  return Promise.reject(error);
});
```

## 效果图
![Jietu20171112-111033](https://i.loli.net/2017/11/14/5a0a32dfd3da7.png)

## 源码地址
[https://github.com/choteewang/BLOG-NOTE/tree/master/Demos/axios+redux_demo](https://github.com/choteewang/BLOG-NOTE/tree/master/Demos/axios+redux_demo)