# React基础知识概括

### 目录
1. [React组件](#1-React组件)
2. [组件通信](#2-组件通信)
3. [Redux](#3-Redux)
4. [React Context API](#4-React-Context-API)
5. [React Hooks API](#5-React-Hooks-API)
6. [React Router](#6-React-Router)
7. [React生命周期](#7-React生命周期)
8. [React的css in JS方案](#8-React的CSS-in-JS方案)
9. [React环境搭建](#9-React环境搭建)
10. [React-Redux的使用](#10-React-Redux的使用)
11. [Redux的中间件和异步操作](#11-Redux的中间件和异步操作)
<br><br>

#### 1. React组件 
* JSX语法：扩展的JS语法，可使用JSX语法在JS中编写类似HTML结构的代码
* React的组件名称首字母必须大写
* React中类的方法默认不绑定`this`，可使用`bind`或`箭头函数`手动绑定
* React的组件必须返回一个JSX元素，返回并列多个JSX元素不合法，需要用一个外层的JSX元素将所有内容包裹起来
* React中可通过`this.state`存储局部变量，当`this.state`发生变化时，会引发组件的重新渲染，React仅可通过`this.setState()`方法来修改state的值
* React的组件可通过函数或者class来定义，当组件有自身的内部状态时，使用class，此时使用`constructor`时，一定要调用`super()`
* React的组件有`this.props.children`属性，表示组件的所有子节点
* 组件类的`propTypes`属性，可用于定义组件实例的属性类型，验证传入参数类型是否符合要求

<br>[top](#目录)<hr>

#### 2. 组件通信 

##### 2.1 父子通信
* 父组件传递数据给子组件时，在调用子组件的地方将数据作为属性的值传入，子组件即可通过`props.属性`获取到数据
```javascript
 <Parent>
   <Child name='jack'/>  //在组件Child中可通过this.props.name获取值‘jack’
 </Parent>
```
* 子组件传递数据给父组件时，需要父组件事先定义一个函数，并通过props传给子组件，子组件在特定时机调用此函数，函数的传入参数即可作为传入的数据

##### 2.2 任意组件通信 
###### 2.2.1 使用EventHub
  * 即为发布订阅模式，使用trigger触发一个事件并传递数据，使用on监听事件并获取数据
  * 一个组件监听某个事件，另一组件触发相同事件并传参，即可实现两个组件的通信
  * 缺点：事件容易越来越多，不易控制代码复杂度，且对事件不好统一管理

  ```
  let money = {
    mount: 100000
  }
  let fnLists = {}
  let eventHub = {
    trigger(eventName,data){
      let  fnList= fnLists[eventName]
      if(!fnList){return}
      for(let i=0;i<fnList.length;i++){
        fnList[i](data)
      }
    },
    on(eventName,fn){
      if(!fnLists[eventName]){
        fnLists[eventName]=[]
      }
      fnLists[eventName].push(fn)
    }
  }

  //eventHub.on('我要花钱',(data)=>{money.mount-=data;})
  //eventHub.trigger('我要花钱',100)
  ```

###### 2.2.2 使用Redux  
  * 根据定义的reducer方法创建store,reducer方法为纯函数，根据原先的state以及接收到的action返回新的state
  * 子组件的操作派发(调用`store.dispatch()`)一个action对象(包含type以及其他数据)
  * reducer利用接收的action以及旧的state按照预先定义的规则更新store中的state
  * 使用store.subscribe()监听state变化，一旦state变化就重新render

  ```
    const reducer = (state=0,action)=>{
      switch(action.type){
        case 'add':
          return (state + action.data);
        default:
          return state;
      }
    }

    const store = createStore(reducer)

    //store.dispatch({type:'add',data:1})
    //store.subscribe(()=>{render()})
  ```
<br>[top](#目录)<hr>

#### 3. Redux 

　实际应用中，并不一定需要使用Redux，当UI层十分简单，互动较少时，使用Redux则会增加应用的复杂性。<br>
　当应用有如下场景时，可考虑使用Redux:

      · 某个组件的状态，需要共享
      · 某个状态需要在任何地方都可获取
      · 一个组件需要改变全局状态
      · 一个组件需要改变另一个组件的状态

##### 3.1 基本概念
3.1.1 **Store**
* `Store`是保存数据的的地方，整个应用只能有一个Store,可使用`createStore`函数来生成Store。

3.1.2 **State**
* `Store`对象包含所有的数据，若想得到某个时点的数据，就要对Store生成快照，这种时点的数据集合，叫做**State**。
* 当前时刻的 State ，可以通过`store.getState()`得到。
* Redux 规定，一个 State 对应一个 View ，只要 State 相同，View 就相同。

3.1.3 **Action**
* Action 是 View 发出的通知，表示 State 应该要发生变化了。
* Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置。
* 可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。

3.1.4 **store.dispatch()**<br>
* `store.dispatch()`是发出Action的唯一方法，它接受一个 Action 对象作为参数，将它发送出去。

3.1.5 **Reducer**
* Store 收到 Action 后，必须给出一个新的State，才会使View发生变化，State的这个计算过程就叫 Reducer。
* Reducer 是一个纯函数，接受 Action 和当前 State 作为参数，返回一个新的 State。纯函数就意味着，只要是同样的输入，必定得到同样的输出。所以，Reducer 函数里面不可改变State，必须返回一个全新的对象。 
* `store.dispatch()`会触发 Reducer 的自动执行，`createStore()`接受 Reducer 作为参数，生成一个新的 Store，每当`store.dispatch()`发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

3.1.6 **store.subscribe()**
* Store 允许使用`store.subscribe()` 方法设置监听函数，一旦State发生变化，就会自动执行这个函数。
* `store.subscribe()`方法会返回一个函数，调用这个函数就可以解除监听。

##### 3.2 Reducer的拆分
* 由于整个应用只有一个 State 对象，对于大型应用而言，这个 State 必然十分庞大，导致负责生成 State 的Reducer函数也十分庞大。我们可以把Reducer 函数进行拆分，不同函数负责处理不同属性，最终合并为一个大的Reducer即可
* Reducer提供了一个`combineReducer`方法，用于合并多个子组件所对应的 Reducer。
```
import {createStore, combineReducers} from 'redux';
import {reducer as todoReducer} from './todos';
import {reducer as filterReducer} from './filter';

//todos 以及 filter 分别为 state 的属性名
const reducer = combineReducers({
  todos: todoReducer,
  filter: filterReducer
});

export default createStore(reducer);
```

<br>[top](#目录)<hr>

#### 4. React Context API 

<br>[top](#目录)<hr>

#### 5. React Hooks API 
<br>[top](#目录)<hr>

#### 6. React Router 
<br>[top](#目录)<hr>

#### 7. React生命周期 
<br>[top](#目录)<hr>

#### 8. React的CSS-in-JS方案 
<br>[top](#目录)<hr>

#### 9. React环境搭建 
<br>[top](#目录)<hr>

#### 10. React-Redux的使用 

##### 10.1 展示组件和容器组件
* React-Redux将所有组件分为两大类:
  * 处于内层只负责渲染界面的组件，叫做UI组件(presentational component);
  * 处于外层负责和Redux Store打交道的组件，叫做容器组件(container component);
* React-Redux规定，所有的展示组件都由用户提供，容器组件则是React-Redux自动生成。

##### 10.2 connect方法 
React-Redux提供`connect`方法，用于从UI组件生成容器组件。connect接受两个参数 mapStateToProps 以及 mapDispatchToProps，其执行结果仍是一个函数，此时将UI组件Counter作为参数传入，得到的结果即为容器组件 CounterContainer。

```
import { connect } from 'react-redux'
const CounterContainer = connect(mapStateToProps,mapDispatchToProps)(Counter);
```
作为容器组件，要做以下两件事情：
* 将外部的数据(即 state 对象)转化为内层UI组件的props
* 把UI组件中，用户的动作转化为Action对象，派送给Store

这两个工作一个是内层UI组件的输入，一个是内层UI组件的输出，即分别对应参数 mapStateToProps 和 mapDispatchToProps 这两个函数所做的事情。

###### 10.2.1 mapStateToProps
* mapStateToProps 是一个函数，返回的是一个对象，包含组件所需使用的state中的部分数据，组件可直接通过`this.props.属性`引用数据。
* mapStateToProps 的第一个参数始终是state，还可以使用第二个参数`ownProps`，代表直接传递给容器组件的 props 对象，需要注意的是，若使用了`ownProps`作为参数，如果容器组件的参数发送变化，也会引起 UI 组件的重新渲染。
* connect 方法可以省略 mapStateToProps 参数，这样UI组件就不会订阅 Store，当Store发生更新时不会引起UI组件的更新。
```
const mapStateToProps = (state)=>{
  return {
    todos : selectTodos(state.todos,state.filter)
  }
}
```

###### 10.2.2 mapDispatchToProps
* mapDispatchToProps 用于把 UI 组件暴露出来的函数类型的prop关联上 `dispatch`函数的调用,它可以是一个函数，也可以是一个对象。
* mapDispatchToProps 是函数时，包含两个参数，分别为 dispatch(必须) 以及 ownProps。
```
function mapDispatchToProps(dispatch){
  return {
    add1: ()=>{
      dispatch({type:'add',payload:1}) 
    },
    add2: ()=>{
      dispatch({type:'add',payload:2}) 
    }
  }
}
```

* mapDispatchToProps 是对象时，它的每个键名是对应 UI组件的同名参数，键值是一个函数,返回的Action会由Redux自动发出。
```
const mapDispatchToProps= {
     add1: ()=>{
       return {type:'add',payload:1}
     }
}
```


##### 10.3 Provider
* React-Redux 提供 Provider 组件，可以让容器组件拿到 state 。
```
ReactDOM.render(
  <Provider store={store}>
    <TodoApp />
  </Provider>, 
  document.getElementById('root'));
```

　上述代码中，使用 Provider 在根组件 TodoApp 外面包了一层，这样 TodoApp 的所有子组件默认都可以拿到 state 了。

* Provider 的原理是 React 组件的 [context](#4-React-Context-API) 属性。
* React-Redux 定义了 Provider 的 `componentWillReceiveProps` 函数，在React组件的生命周期中，componentWillReceiveProps 函数在每次重新渲染时都会调用到。

<br>[top](#目录)<hr>

#### 11. Redux的中间件和异步操作

<br>[top](#目录)<hr>


<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>


