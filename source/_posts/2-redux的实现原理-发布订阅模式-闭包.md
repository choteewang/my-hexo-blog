---
title: 'Redux 1: Redux的实现原理(发布订阅模式,闭包)'
date: 2017-11-13 20:57:01
tags:
---
## 前置知识:发布订阅者模式
[发布订阅者模式_阅读](https://www.cnblogs.com/tugenhua0707/p/4687947.html)
## createStore的实现,闭包,发布订阅模式

``` js
const createStore = (reducer) => {
  // 闭包内要操作的数据,state是数据,listeners是发布订阅模式的订阅数组
  let state;
  let listeners = [];
  // 通过getState返回state的值
  const getState = () => state;
  
  //通过dispatch的参数接收action对象
  const dispatch = (action) => {
  	 // 将action对象传入reducer更新state
    state = reducer(state, action);
    // 一旦state被更新,订阅数组中的所有listener方法被执行
    listeners.forEach(listener => listener());
  };
  
  // 通过subscribe的参数接收listener方法
  const subscribe = (listener) => {
  	 // 将listner方法放入订阅数组listeners
    listeners.push(listener);
    // 同时return一个方法形成闭包,这个方法用来从订阅数组listeners中删除掉listener方法
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };
  
  // 执行一次dispatch,形成最初的state数据
  dispatch({});
  
  // 将getState, dispatch, subscribe方法向外return,形成闭包
  return { getState, dispatch, subscribe };
};
```
## 最基础的Redux数据流
![Jietu20171110-094632](https://i.loli.net/2017/11/13/5a099379e0660.png)


``` java
import React from 'react';
import ReactDOM from 'react-dom';
// 从redux中拿到createStore函数
import { createStore } from "redux";
// ui组件,只有props从外界接收数据并展示,让redux帮我们处理state数据的传递
class Counter extends React.Component {
  constructor(props) {
    super(props)
  }
  render() {
    return (
      <div>
        <h1>{this.props.count}</h1>
        <input type="button" value="Increase" onClick={this.props.onIncrease} />
        <input type="button" value="Decrease" onClick={this.props.onDecrease} />
      </div>
    )
  }
}

// reducer,在createStore形成的闭包内部处理state的过滤器,
// 在createStore形成的闭包内部 使用state=reducer(state,action)实现,return的值重新赋值给state
const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'increase':
      return state + 1
    case 'decrease':
      return state - 1
    default:
      return state
  }
}

// store是createStore闭包的返回值,是一个对象,结构是{getState,dispatch,subscribe},
// createStore通过参数传入的reducer形成state的生成规则
// createStore方法还可以接受第二个参数，表示整个应用的state的初始状态,会覆盖reducer函数中的默认初始值
const store = createStore(reducer)
// 通过store.subscribe(listener)方法拿到的listener函数进行订阅发布者模式监听,只要state的值变化,订阅数组listenrs中的所有listener方法就会被执行
const unsubscribe = store.subscribe(listener)
// store.subscribe的返回值是一个方法unsubscribe(),可以从订阅数组listeners中移除listener方法,取消监听

//这是subscribe调用的listener方法,state的值改变后会被调用,内部让其自动执行渲染函数更新UI组件Counter
function listener() {
  ReactDOM.render(
    <Counter
      // 新生成的state值可以通过store.getState()方法得到闭包快照
      count={store.getState()}
      // 通过store.dispatch(action)方法拿到action对象传入reducer参数更新state值,state=reducer(state,action)
      // action 是一个对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置
      onIncrease={() => { store.dispatch({ type: 'increase' }) }}
      onDecrease={() => { store.dispatch({ type: 'decrease' }) }}
    >
    </Counter>,
    document.getElementById('root'));
}
// 首次进入页面先渲染一次页面
listener()
```
![Jietu20171110-082736](https://ooo.0o0.ooo/2017/11/13/5a0993bd51d99.png)
## 参考文档
[阮一峰 redux (1)](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)