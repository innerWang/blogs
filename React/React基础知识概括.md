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
<br><br>

#### 1. React组件
* JSX语法：扩展的JS语法，可使用JSX语法在JS中编写类似HTML结构的代码
* React的组件名称首字母必须大写
* React中类的方法默认不绑定`this`，可使用`bind`或`箭头函数`手动绑定
* React的组件必须返回一个JSX元素，返回并列多个JSX元素不合法，需要用一个外层的JSX元素将所有内容包裹起来
* React中可通过`this.state`存储局部变量，当`this.state`发生变化时，会引发组件的重新渲染，React仅可通过`this.setState()`方法来修改state的值
* 组件可通过函数或者class来定义，当组件有自身的内部状态时，使用class，此时使用constructor时，一定要调用`super()`
<br>

#### 2. 组件通信
##### 2.1 父子通信
* 父组件传递数据给子组件时，在调用子组件的地方将数据作为属性的值传入，子组件即可通过`props.属性`获取到数据
```javascript
 <Parent>
   <Child name='jack'/>  //在组件Child中可通过props.name获取值‘jack’
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
<br>

#### 3. Redux

<br>

#### 4. React Context API

<br>

#### 5. React Hooks API
<br>

#### 6. React Router
<br>

#### 7. React生命周期
<br>

#### 8. React的CSS-in-JS方案
<br>

#### 9. React环境搭建
<br>




