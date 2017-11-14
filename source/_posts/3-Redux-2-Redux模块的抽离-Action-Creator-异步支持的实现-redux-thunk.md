---
title: 'Redux 2: Redux模块的抽离,Action Creator,异步支持的实现(redux-thunk)'
date: 2017-11-13 21:23:52
tags:
---
## TODO
[上一篇redux的总结](./redux的实现原理(发布订阅模式,闭包).md)中,写了一个小的Counter Demo, 但是代码耦合度太高, 在复杂项目中组织代码难度增加, 且无法进行异步redux操作, 这篇总结中, 将代码以模块化思想解耦合, 并让redux可以支持异步操作.

需要用到redux-thunk这个库,这个库的作用是让store.dispatch接受以函数作为参数,从而从函数参数中拿到dispatch与getState执行异步操作, 最底层的原理可以移步阮一峰博客redux第二篇,这里只做代码实现,和详细注释

## CODE
```java
// Index.js : 根组件挂载处
import App from './App';
import React from 'react';
import ReactDOM from 'react-dom';
// 引入redux-thunk,使dispatch可以接受一个函数作为参数,从而支持applyMiddleware的中间件处理
import thunk from 'redux-thunk';
import { createStore, applyMiddleware } from 'redux'
// 引入自己抽离的actionCreater与reducer模块
import { onIncrease, onDecrease, onIncreaseAsync,reducer } from './myRedux';
// 在根组件的挂载处挂载store与reducer,actionCreater控制整个app的redux数据流
const store = createStore(reducer,applyMiddleware(thunk))
const unsubscribe = store.subscribe(listener)

function listener() {
  ReactDOM.render(
    <App
      //将store和actionCreater传入组件内部
      store={store}
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onIncreaseAsync={onIncreaseAsync}
    >
    </App>,
    document.getElementById('root'));
}
listener()
```

```java
// App.js : UI组件定义处
import React, { Component } from 'react';
// App就是上篇文章的Counter,不同的是它是一个UI组件,只负责UI的展示和数据流的分派

class App extends Component {
  render() {
    // store与ActionCreater从根组件挂载处index.js处拿到
    const store = this.props.store
    const onIncrease = this.props.onIncrease
    const onDecrease = this.props.onDecrease
    const onIncreaseAsync = this.props.onIncreaseAsync
    return (
      <div>
        <h1>{store.getState()}</h1>
        <input type="button" value="increase" onClick={() => { store.dispatch(onIncrease()) }} />
        <input type="button" value="decrease" onClick={() => { store.dispatch(onDecrease()) }} />
        {/* 添加异步操作UI */}
        {<input type="button" value="increaseAsync" onClick={() => { store.dispatch(onIncreaseAsync()) }} />}
      </div>
    );
  }
}
export default App;
```

```javascript
// myRedux.js
// 将actionCreater与reducer抽离成一个单独的js模块

// 定义action.type对应的常量,防止后期频繁修改
const INCREASE = 'increase'
const DECREASE = 'decrease'

export const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}

//为了不频繁写action,创建actionCreater方法,自动生成action
export const onIncrease = () => {
  return { type: INCREASE }
}

export const onDecrease = () => {
  return { type: DECREASE }
}
// redux异步的解决方案:写出返回函数的actionCreator，使用reduxThunk中间件改造dispatch使函数可以作为其参数。

// 定义异步的actionCreater,返回一个函数
export const onIncreaseAsync = () => {
  // 这里return出的方法参数是dispatch与getState,后续applyMiddleWare方法会将store.dispatch与store.getState传入
  // 这里return出的方法是一个中间件方法,applyMiddleWare会在这个方法执行之前发一个action,执行之后再发一个action,从而实现异步处理.
  return (dispatch) => {
    setTimeout(() => {
      dispatch(onIncrease())
    },3000);
  }
}
```
![Jietu20171111-033033](https://i.loli.net/2017/11/13/5a099c902645b.png)
## 参考资料
[阮一峰 redux(2)](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)
