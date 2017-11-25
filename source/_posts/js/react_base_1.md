---
title: 只要998 ，美味全家桶带回家
date: 2017-11-24 16:10:25
updated: 2017-11-25 09:40:00
categories: 学习
tags: [JS,React]
---
这个本来是在上周或者上上周在小平乓社和大家分享的，可惜因为突发事件，搁浅了，文章也还没写完。。。
<!--more-->

React、Redux、React-Router、React-Redux 、React-Router-Redux、Redux-Saga

### React

>A JavaScript library for building user interfaces
>一个用于构建用户界面的 JavaScript 库

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

**组件化**

一个完整的页面是由大大小小的组件堆叠而成，就好像搭积木，每一块积木都是一个组件，组件套组件组成了用户所能看到的完整的页面。

**JSX**
使用React，不一定非要使用JSX语法，可以使用原生的JS进行开发。但是JSX是在是太好用了，强烈建议大家使用。

```javascript
class HelloMessage extends React.Component {
  render() {
    return (
      return React.createElement('div', null, 'hello '+this.props.name);
  }
}

```
**Virtual DOM**
每个真实的页面对应了一个DOM树，在传统开发模式中，每次需要更新页面时，都需要对DOM进行操作，DOM操作十分昂贵，为减少对于真实DOM的操作，诞生了Virtual DOM的概念。

Virtual DOM将真实的DOM树用javascript描述一遍，每次数据更新之后，重新计算Virtual DOM，并和上一次的Virtual DOM对比，对发生的变化进行批量更新。React也提供了shouldComponentUpdate生命周期回调，来减少数据变化后不必要的Virtual DOM对比过程，提升了性能。

Virtual DOM最大的好处在于可以很方便的和其它平台集成，react-native基于Virtual DOM渲染出原生控件。具体渲染出的是Web DOM还是Android控件或是iOS控件就由平台决定了------Learn Once,Write Anywhere

**函数式编程**
在React中，数据的流动是单向的，即从父节点传递到子节点。也因此组件是简单的，他们只需要从父组件获取props渲染即可。如果顶层的props改变了，React会递归的向下遍历整个组件树，重新渲染所有使用这个属性的组件。那么父组件如何获取子组件数据呢？很简单，通过回调就可以了，父组件定义某个方法供给子组件调用，子组件调用方法传递给父组件数据。

**生命周期**
{% asset_img Phases-of-React-Component.jpg Phases-of-React-Component %}

{% asset_img lifecycle.png lifecycle-of-React-Component %}


**数据流**

**[Flux](https://github.com/facebook/flux)**
> An application architecture for React utilizing a unidirectional data flow.
> Flux 是一种让React使用单向数据流的架构思想。

{% asset_img flux2.png flux%}

Flux主要包括四个部分: dispatcher, stores 和 views(React的组件)。
- View： 视图层
- Action（动作）：视图层发出的消息（比如Click）
- Dispatcher（派发器）：用来接收Actions、执行回调函数，提供了把Action分发给Store的机制
- Store（数据层）：用来存放应用的状态，一旦发生变动，就提醒Views要更新页面


{% asset_img flux.png flux%}


### Redux
> Redux is a predictable state container for JavaScript apps.
> Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

#### 三大原则
**单一数据源**
> 整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。

**State 是只读的**
> 惟一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。

```javascript
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
});
```
**使用纯函数来执行修改**
> 它接收先前的 state 和 action，并返回新的 state

```javascript
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}
```
####  Action
> Action 是把数据从应用传到 store 的有效载荷。它是 store 数据的唯一来源。

```javascript
const ADD_TODO = 'ADD_TODO'
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```
约定action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作，其他的字段可以自己随意规定。
#### Action 创建函数
```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```
这样使我们可以更方便灵活的使用Action，也便于测试。
Redux 中只需把 action 创建函数的结果传给 dispatch() 方法即可发起一次 dispatch 过程。
```javascript
	dispatch(addTodo(text))
```
store 里能直接通过 store.dispatch() 调用 dispatch() 方法，但多数情况下是使用 react-redux 提供的 connect() 来调用。

bindActionCreators() 可以自动把多个 action 创建函数 绑定到 dispatch() 方法上。

#### Reducer

Action 只是描述了有事情发生了这一事实，并没有指明应用如何更新 state。而这正是 reducer 要做的事情。
reducer 就是一个纯函数，接收旧的 state 和 action，返回新的 state。

```javascript
(previousState, action) => newState
```

保持 reducer 纯净非常重要。永远不要在 reducer 里做这些操作：
- 修改传入参数；
- 执行有副作用的操作，如 API 请求和路由跳转；
- 调用非纯函数，如 Date.now() 或 Math.random()。

```javascript
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    default:
      return state
  }
}
```

```javascript
import { combineReducers } from 'redux';

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp;
```

**注意每个 reducer 只负责管理全局 state 中它负责的一部分。每个 reducer 的 state 参数都不同，分别对应它管理的那部分 state 数据。**

#### Store
action 来描述“发生了什么”，reducers 来根据 action 更新 state.
Store 就是把它们联系到一起的对象。Store 有以下职责：

- 维持应用的 state；
- 提供 getState() 方法获取 state；
- 提供 dispatch(action) 方法更新 state；
- 通过 subscribe(listener) 注册监听器;
- 通过 subscribe(listener) 返回的函数注销监听器。

**Redux 应用只有一个单一的 store。当需要拆分数据处理逻辑时，你应该使用 reducer 组合 而不是创建多个 store。**
```javascript
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```
createStore() 的第二个参数是可选的, 用于设置 state 初始状态。
```javascript
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```
```javascript
import { addTodo, toggleTodo, setVisibilityFilter, VisibilityFilters } from './actions'

// 打印初始状态
console.log(store.getState())

// 每次 state 更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// 发起一系列 action
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(toggleTodo(0))
store.dispatch(toggleTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// 停止监听 state 更新
unsubscribe();
```
结果就如下图：
{% asset_img output.png output%}

#### 数据流
**核心：严格的单向数据流**

1. 调用 store.dispatch(action)。
2. Redux store 调用传入的 reducer 函数。
3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。
4. Redux store 保存了根 reducer 返回的完整 state 树。

#### 搭配 React
Redux 和 React 之间没有关系。Redux 支持 React、Angular、Ember、jQuery 甚至纯 JavaScript。
** React Redux**
```javascript
npm install --save react-redux
```
**容器组件和展示组件**

|        |    容器组件 | 展示组件  |
|:--------:|:--------:|:--:|
| Location  | 最顶层，路由处理 |  中间和子组件  |
| Aware of Redux  | 是 | 否  |
| 读取数据  | 从 Redux 获取 state|  从 props 获取数据  |
| 修改数据  | 向 Redux 派发 actions |  从 props 调用回调函数  |


### React-Router-Redux

react-router-redux 是将react-router 和 redux 集成到一起的库，让你可以用redux的方式去操作react-router。例如，react-router 中跳转需要调用 router.push(path)，集成了react-router-redux 你就可以通过dispatch的方式使用router，例如跳转可以这样做 store.dispatch(push(url))。本质上，是把react-router自己维护的状态，例如location、history、path等等，也交给redux管理。一般情况下，是没有必要使用这个库的。
