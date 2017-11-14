---
title: 'Redux 3: react-redux继续改造Redux数据流'
date: 2017-11-13 22:23:52
tags:
---
## TODO
React-Redux 将所有组件分成两大类：UI 组件（presentational component）和容器组件（container component）。
### UI 组件
- 只负责 UI 的呈现，不带有任何业务逻辑
- 没有状态（即不使用this.state这个变量）
- 所有数据都由参数（this.props）提供
- 不使用任何 Redux 的 API
### 容器组件
- 负责管理数据和业务逻辑，不负责 UI 的呈现
- 带有内部状态
- 使用 Redux 的 API

UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑。
将组件拆分成下面的结构：外面是一个容器组件，里面包了一个UI 组件。前者负责与外部的通信，将数据传给后者，由后者渲染出视图。
React-Redux 规定，所有的 UI 组件都由用户提供，容器组件则是由 React-Redux 自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。

`下面是将已模块化拆分的,支持异步的redux数据流, 继续改造为react-redux的代码`
## CODE
``` javascript
//myRedux.js 代码保持不变
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
//index.js 根节点挂载处
import App from './App';
import React from 'react';
import ReactDOM from 'react-dom';
// applyMiddleware用来处理异步中间件thunk,compose用来将chrome插件与react-thunk按固定顺序连接起来
import { createStore , applyMiddleware ,compose } from 'redux'
// 为了在根组件页面定义store并传入,需要将reducer引入
import { reducer } from './myRedux';
import { Provider } from 'react-redux'
import thunk from 'redux-thunk'

//chrome插件redux-devtools github文档规定的插件声明方式
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
const store = createStore(reducer,composeEnhancers(applyMiddleware(thunk)))

//删除subcribe和listener,把ReactDOM.renden重写
//const unsubscribe = store.subscribe(listener)

ReactDOM.render(
  //根组件外套一层Provider传入Store
  <Provider store={store}>
    <App/>
  </Provider>,
  document.getElementById('root'));

```

``` java
//App.js, UI组件定义处 与 包裹UI的容器组件生成处
import React, { Component } from 'react';
// 引入react-redux的connect函数
import { connect } from 'react-redux';
// 引入所有Action Creator
import { onIncrease, onDecrease, onIncreaseAsync } from './myRedux';


//UI组件,只用来做数据展示和分发,将来可以抽离出去
class AppUI extends Component {
  render() {
    return (
      <div>
        <h1>{this.props.count}</h1>
        <input type="button" value="increase" onClick={this.props.onIncrease} />
        <input type="button" value="decrease" onClick={this.props.onDecrease} />
        {<input type="button" value="increaseAsync" onClick={this.props.onIncreaseAsync} />}
      </div>
    );
  }
}
//mapStateToProps是一个函数,返回一个对象,key对应UI组件上的参数名称,"值"应该是state或算出state的方法的调用
const mapStateToProps = (state) => {
  return {
    count: state
  }
}
//mapDispatchToProps是一个对象,key对应UI组件对应的参数名称,"值"对应传入的actionCreater
const mapDispatchToProps = {
  onIncrease, onDecrease, onIncreaseAsync
}
//用connect函数生成包裹UI组件的容器组件
const App = connect(
  mapStateToProps,
  mapDispatchToProps
)(AppUI)
export default App;

```
![Jietu20171111-033033_iutjibikf](https://i.loli.net/2017/11/13/5a099fedeaa9e.png)
## REFERENCE
[阮一峰 Redux 3](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html)