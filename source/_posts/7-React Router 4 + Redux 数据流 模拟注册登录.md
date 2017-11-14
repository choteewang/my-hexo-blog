---
title: 'Redux 6: React Router 4 + Redux 数据流 模拟注册登录'
date: 2017-11-14 08:03:53
tags:
---
# React Router 4 + Redux 数据流 模拟注册登录
 
## BEFORE
在Redux学习的三篇学习笔记之后,加上React-Router 4的学习,模拟了一个注册登录+计数器的Redux数据流

## CODE

``` java
// index.js Provider,路由,根组件挂载处
ReactDOM.render(
  <Provider store={store}>
    <BrowserRouter>
      {/* Switch:命中第一个路由后不再继续跳转 */}
      <Switch>
        {/* Route匹配路由跳转路径与组件间的关系,exact开启精确匹配,不再继续向下查找 */}
        <Route path='/login' exact component={Auth}></Route>
        <Route path='/dashboard' component={DashBoard}></Route>
        {/* Redirect发起重定向 */}
        <Redirect to='/dashboard'></Redirect>
      </Switch>
    </BrowserRouter>
  </Provider>,
  document.getElementById('root'));
```
``` javascript
// Auth.redux.js
// 权限登录的reducer与action creater
// isAuth代表是否登录
const LOGIN = 'LOGIN'
const LOGOUT = 'LOGOUT'
//reducer
export const AuthReducer = (state = { isAuth: false, user: 'choteewang' }, action) => {
  switch (action.type) {
    case LOGIN:
      return { ...state, isAuth: true }
    case LOGOUT:
      return { ...state, isAuth: false }
    default:
      return state
  }
}
//action creater
export const login = () => {
  return { type: LOGIN }
}

export const logout = () => {
  return { type: LOGOUT }
}

//myRedux.js
const INCREASE = 'increase'
const DECREASE = 'decrease'

export const reducer = (state  = 0, action) => {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}
export const onIncrease = () => {
  return { type: INCREASE }
}
export const onDecrease = () => {
  return { type: DECREASE }
}
export const onIncreaseAsync = () => {
  return (dispatch) => {
    setTimeout(() => {
      dispatch(onIncrease())
    },3000);
  }
}

//reducer.js
import { reducer } from './myRedux';
import { AuthReducer } from './Auth.redux';
import { combineReducers } from 'redux'
// 利用combineReducers合并两个Reducer
// 变为store中的state对象包裹2个reducer定义的state小对象的数据结构{counter:{},AutuReducer:{}}
// 这里要注意改键名,防止之前使用小state的组件访问根组件state时key名发生错误
// 所以最佳实践是reducer要起名要有语义
export default combineReducers({counter:reducer, AuthReducer:AuthReducer})
```

```java
// Auth.js
import React from 'react';
import {connect} from 'react-redux'
import {login} from './Auth.redux';
import {Redirect} from 'react-router-dom';
class Auth extends React.Component{
  constructor(props) {
    super(props)
  }
  render(){
    const redirect = <Redirect to='/dashboard'></Redirect>
    const login = (
      <div>
        <h3>您没登录,请登录</h3>
        <button onClick={this.props.login}>点我模拟登录</button>
      </div>
    )
    // 若登录了,则跳转到dashboard页面,若没登录,显示登录页面
    return this.props.isAuth ? redirect : login
  }
}

export default Auth = connect(
  (state) => {
    return {
      isAuth:state.AuthReducer.isAuth,
    }
  },
  {login}
)(Auth)
```

```java
// dashboard.js
import React from 'react';
import {connect} from 'react-redux'
import {login} from './Auth.redux';
import {Redirect} from 'react-router-dom';
class Auth extends React.Component{
  constructor(props) {
    super(props)
  }
  render(){
    const redirect = <Redirect to='/dashboard'></Redirect>
    const login = (
      <div>
        <h3>您没登录,请登录</h3>
        <button onClick={this.props.login}>点我模拟登录</button>
      </div>
    )
    // 若登录了,则跳转到dashboard页面,若没登录,显示登录页面
    return this.props.isAuth ? redirect : login
  }
}

export default Auth = connect(
  (state) => {
    return {
      isAuth:state.AuthReducer.isAuth,
    }
  },
  {login}
)(Auth)
```

```java
// App.js
import React from 'react';
import {connect} from 'react-redux'
import {login} from './Auth.redux';
import {Redirect} from 'react-router-dom';
class Auth extends React.Component{
  constructor(props) {
    super(props)
  }
  render(){
    const redirect = <Redirect to='/dashboard'></Redirect>
    const login = (
      <div>
        <h3>您没登录,请登录</h3>
        <button onClick={this.props.login}>点我模拟登录</button>
      </div>
    )
    // 若登录了,则跳转到dashboard页面,若没登录,显示登录页面
    return this.props.isAuth ? redirect : login
  }
}

export default Auth = connect(
  (state) => {
    return {
      isAuth:state.AuthReducer.isAuth,
    }
  },
  {login}
)(Auth)
```

## 效果图
![Jietu20171112-085631](https://i.loli.net/2017/11/14/5a0a34278b587.png)

![Jietu20171112-085643](https://i.loli.net/2017/11/14/5a0a343ce1d95.png)