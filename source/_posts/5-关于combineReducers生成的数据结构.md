---
title: 'Redux 4 :关于combineReducers生成的数据结构'
date: 2017-11-14 07:33:50
tags:
---
reducer太多后,要将所有reducer合成为一个,使用combineReducers方法

``` javascript
// myReducer.js
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
```
``` javascript
// Auth.reducer.js
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
```

```javascript
// reducers.js
// 利用combineReducers合并两个Reducer
// 变为store中的state对象包裹2个reducer定义的state小对象的数据结构{counter:{},AutuReducer:{}}
// 这里要注意改键名,防止之前使用小state的组件访问根组件state时key名发生错误
// 所以最佳实践是reducer要起名要有语义
export default combineReducers({counter:reducer, AuthReducer:AuthReducer})
```

`根组件挂载store后拿到的store.getState()` 可以清晰的看到数据结构
![Jietu20171112-080243](https://i.loli.net/2017/11/14/5a0a2b3033179.png)