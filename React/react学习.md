#### 1. 受控组件和非受控组件

react中，对于表单元素如input, textarea以及 select都有其内部状态，此时若作为非受控组件来使用，则无法对输入进行校验，采用的方法是，将其改为受控组件，通过react控制状态，如将`<input>`、`<textarea>`、`<select>`这些标签的value设置为`this.state.val`。



需要注意的是 `<input type="file">`由于其value是只读的，所以始终是一个非受控组件，但是我可以通过使用`React.createRef()`获取对其的引用读取信息。

```js
this.fileInput = React.createRef()

onSubmit = () => console.log(this.fileInput.current.files[0].name)

<input type="file" ref={this.fileInput} />
```



<br><hr><br>

#### 2. React哲学

2.1 最小state, 即如何判断状态是否是react的state?

1. 是否由props传递而来，是的话则不是state,
2. 是否随着时间的推移保持不变，是的话，则不是state，
3. 是否可以根据state或者props计算得到，是的话，则也不是state



2.2  **this.setState()**

this.setState() 的第一个参数可以有两种形式： 

* 是一个对象，`{}` 表示新的状态 
* 是一个函数，` (prevState, props) => {}`，此时第一个参数是上一个state，第二个参数是此次更新被应用时的props。 

<br><hr><br>

#### 3. 代码分割

##### 3.1 引入代码分割的最佳方法— — — — 动态 `import()` 语法

##### 3.2 React.lazy( () => import('./math') )

`React.lazy` 接受一个函数，这个函数需要动态调用 `import()`。它必须返回一个 `Promise`，该 Promise 需要 resolve 一个 `defalut` export 的 React 组件。

### 



<br><hr><br>

#### 4. Context API

Context 可以使用不需要通过组件树逐层传递props。如一些属性(地区偏好、UI主题)。主要应用场景是，在于*很多*不同层级的组件需要访问同样一些的数据。请谨慎使用，因为这会使得组件的复用性变差。



4.1） 层层传递多个props的解决方案，不使用context

A  ——>   B  ——>   C   ——> D  ——> E  , 其中子组件E要使用A组件传递的两个属性，此时的解决方法是

```js
function A(props) {
  const userLink = <E name={props.name} avator={props.avator} />
  return <B  userLink={userLink}/>
}

function C(props) {
  return <D userLink= props.userLink />
}
  
function D(props) {
  return {props.userLink}
}
```

使用这种方式，减少了在应用中传递props的数量，且中间的组件不局限接收单个子组件，可以将子组件和直接关联的父组件解耦，但是会提高了高层组件的复杂度，若是子组件在渲染前会和父组件有交流，则可以使用 `props.render`。



4.2 )  **API**

 [示例参考](https://codesandbox.io/s/xuexicontext-y0802)

1. React.createContext

   ```js
   const myContext = React.createContext(defaultValue)
   ```

   订阅了这个Context的组件会从组件树中离自己**最近的**，**匹配的**`Provider`读取当前的`context`的值，若没有匹配，则defaultValue生效。

   

2. Context.Provider

   ```js
   <myContext.Provider  value={/*某个值 = data */}>  // 传递的值为data，data可以是任何类型？？
   </ myContext.Provider>
   ```

   Provider接收一个value属性，传递给Consumer，Provider 及其内部 Consumer 组件都不受制于 `shouldComponentUpdate` 函数。 因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

   

3. Context.Consumer

   ```js
   <myContext.Consumer>
     {data => /* 基于传递的值进行渲染*/}
   </myContext.Consumer>
   ```

   传递给函数的 `data` 值等同于往上组件树离这个 context 最近的 Provider 提供的 `value`属性的值。如果没有对应的 Provider，`data` 参数等同于传递给 `React.createContext()`的 `defaultValue`。

   

4. 类组件的contextType

   ```js
   class TestClass extends React.Component{
     componentDidMount(){
       let data = this.context
     }
     
     render(){
       let data = this.context
       /* 使用data进行渲染 */
     }
   }
   
   TestClass.contextType = myContext 
   ```

   类组件的`contextType`属性可以被`React.createContext()`创建的Context对象赋值，此时就可以在类组件的生命周期中使用`this.context`来消费最近的匹配的Context的值。

   **思考： 多个类组件的实例，都可以消费同样的数据？？ 待测试**

   

5. 注意事项

   因为 context 会使用参考标识（reference identity）来决定何时进行渲染，当使用了Provider的组件的组件重新渲染时，若传递的value是一个对象，则会导致value被重新赋值为新的对象，可能会在 consumers 组件中触发意外的渲染，此时，可将value的值存到组件的state中，然后传递 state的值。



<br><hr><br>

#### 5. 错误边界


错误边界是一个组件，当一个组件定义了`static getDerivedStateFromError()` 和`componentDidCatch()`之一时，则这个组件就是一个错误边界，错误边界可以用于捕获子组件在渲染期间抛出的错误，当该组件的子组件抛出错误时，可以使用`static getDerivedStateFromError()` 渲染备用 UI ，使用 `componentDidCatch()` 打印错误信息。



<br><hr><br>

#### 6.  Refs Forward （转发 Ref）


Ref 转发用于将ref自动地通过组件传递到某个子组件。

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

function FatherComp(){
  // 你可以直接获取 DOM button 的 ref：
  const ref = React.createRef();
  return (
    <FancyButton ref={ref}>Click me! </FancyButton>
  )
}

```

如上述示例，`FancyButton` 使用 `React.forwardRef` 来获取传递给它的 `ref`，然后转发到它渲染的 DOM `button`, 这样`FatherComp`组件就可以获取到button的引用。

需要注意以下两点：

* 第二个参数 `ref` 只在使用 `React.forwardRef` 定义组件时存在。常规函数和 class 组件不接收 `ref` 参数，且 props 中也不存在 `ref`。
* Ref 转发不仅限于 DOM 组件(如上述的`<button>`)，你也可以转发 refs 到 class 组件实例中。

<br><hr><br>

#### 7.  `<Fragment>`

`<Fragment>`允许将子列表进行分组而无需增加额外的DOM标签

可以使用 `<Fragment> </Fragment>`或者`<> </>`包裹子列表，不同点在于前面的 `<Fragment>`标签可以传递`key`属性，当然`key`属性也是唯一可以传递给`<Fragment>`的属性。





<br><hr><br>

#### 8. 高阶组件HOC  —— ——  需要多看几遍！！！！


高阶组件是，参数为组件，返回一个新组件的函数。

在返回的新组件内部实现通用的功能，渲染传进来的组件，需要注意以下几点：

* 不要更改传入组件原型

* 透传与HOC无关的props

  ```js
  render(){
    const {extraProps, ...passThroughProps} = this.props
    // 将 props 注入到被包装的组件中。
    // 通常为 state 的值或者实例方法。
    const injectedProp = someStateOrInstanceMethod;
  
    // 将 props 传递给被包装组件
    return (
      <WrappedComponent
        injectedProp={injectedProp}
        {...passThroughProps}
      />
    );
  }
  ```

  

* 不要在render方法中使用 HOC

* 在高阶组件中一定要拷贝传入组件的静态方法

* 由于refs不属于props的一部分，所以refs不会被传递，此时需要考虑使用 `React.forwardRef`API



<br><hr><br>

#### 9. 深入JSX


JSX只是`React.createElement(component, props,...children)`函数的语法糖。

所以必须引入`React`



* 如果没给prop赋值，则其默认值为true，如下两个式子等价。

  ```js
  <MyTextBox autocomplete />
  <MyTextBox autocomplete={true} />
  ```

  

* `props.children` 是包含在开始和结束标签之间的JSX表达式的内容

* 需要注意，有一些 **"falsy"** 的值，如数字`"0"`，仍会被react渲染，要是想解决这个问题，要确保 **"&&"** 之前的表达式总是布尔值

  ```js
  <div>
    {props.messages.length && <MessageList messages={props.messages} />}
  </div>
  
  // 将 && 之前的表达式转化为布尔值
  <div>
    {props.messages.length > 0 && <MessageList messages={props.messages} />}
  </div>
  ```

* 若想渲染`false`、`true`、`null`、`undefined` 等值，你需要先将它们转化为字符串

  ```js
  <div>
    My JavaScript variable is {String(myVariable)}.
  </div>
  ```

  

<br><hr><br>

#### 10. 性能优化


* 构建优化

* 虚拟化长列表

  若应用渲染了长列表(成百上千的列表数据)， 则可考虑使用"虚拟滚动"技术，可会在有限的时间内仅渲染有限的内容。 可参考的工具库如[react-window](https://react-window.now.sh/) 和 [react-virtualized](https://bvaughn.github.io/react-virtualized/)。

* 使用 `shouldComponentUpdate` 优化渲染





<br><hr><br>

#### 11. Portals

可通过使用`Portals`将子节点渲染到父组件之外的DOM节点。

```js
ReactDOM.createPortal(child, container)  // child为子节点，container为DOM元素
```

如下示例，将子元素插入到一个指定的DOM节点中：

```js
render() {
  // React 并没有创建一个新的 div。它只是把子元素渲染到 `domNode` 中。
  // `domNode` 是一个可以在任何位置的有效 DOM 节点。
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```

一个 portal 的典型用例是当父组件有 `overflow: hidden` 或 `z-index` 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框。使用portal时，一定要注意管理键盘焦点。



<br><hr><br>

#### 12. Refs和DOM


##### 1. 创建Refs

Refs 是使用 `React.createRef()` 创建的，并通过 `ref` 属性附加到 React 元素。在构造组件时，通常将 Refs 分配给实例属性，以便可以在整个组件中引用它们。



##### 2. 访问Refs

将创建的ref传递给`render()`中的DOM元素时，可以通过`ref`的`current`属性来访问DOM元素。

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  
  componentDidMount(){
    const node = this.myRef.current;
  }
  
  render() {
    return <div ref={this.myRef} />;
  }
}

```

`ref`属性用于 **HTML元素 / 自定义class组件** 时，`ref`对象接收 **底层DOM元素 / 组件的挂载实例** 作为`current`属性。


**！！！函数组件没有实例，所以不可以在函数组件上使用ref属性！！！** 但是可以在函数组件内部使用ref属性，但是这个ref属性只能和DOM元素或者class组件进行绑定。

```js
function CompA(){
  return <div>I am CompA </div>
}

// CompA是一个函数组件，不可以如下使用ref!!!!
<CompA ref={this.myRef}>

```





##### 3. 回调Refs

之前使用 `Refs`的方式是，传递`React.createRef()`给`ref`属性，回调Refs则是传递一个函数，使用这个函数来接收React组件实例或者HTML DOM元素作为参数，以便存储和访问。

```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = null;
    // 使用回调Refs
    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      if (this.textInput) 
        this.textInput.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    // 使用 ref 的回调函数将 text 输入框 DOM 节点的引用存储到 React实例上（比如 this.textInput）
    return (
      <div>
        <input type="text" ref={this.setTextInputRef} />
        <button onClick={this.focusTextInput}>click me! </button>
      </div>
    );
  }
}

```

需要注意的是，最好将 ref 的回调函数定义为 class的绑定函数方式，而不是以内联函数的方式定义，否则在更新时对current的引用会执行两次。

这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素，然后就会执行两次。





<br><hr><br>

#### 13. Render Props

`render prop`是指在React组件之间使用 **`值为函数的prop`** 共享代码的简单技术。

具有render prop的组件接受一个函数，调用该函数则返回一个React元素。

```js
<DataProvider render={data => (<h1>Hello {data.target}</h1>)} />
```



<br><hr><br>

#### 14. 静态类型检查

建议使用 Flow 或者 TypeScript 代替 PropTypes 进行静态类型检查。

[添加ts到现有的create-react-app项目中](https://facebook.github.io/create-react-app/docs/adding-typescript)

[添加ts到现有项目中](https://zh-hans.reactjs.org/docs/static-type-checking.html#adding-typescript-to-a-project)

[tsconfig.json参考](https://github.com/Microsoft/TypeScript-React-Starter/blob/master/tsconfig.json)



<br><hr><br>

#### 15.  严格模式(`<React.StrictMode>`)

该严格模式检查仅在开发模式下运行，不会影响生产构建。同`<Fragment>`一致，不会渲染标签，仅会对其包裹的后代元素进行检查。



<br><hr><br>

#### 16. 使用PropTypes类型检查

从React v15.5起， `React.PropTypes`已移入另一个`prop-types库`，需要安装对应的依赖。出于性能的考虑，propTypes仅在开发模式下进行检查。



可以通过 `类名.defaultProps`来指定props的默认值。

```js
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>
  }
}

// 指定 props 的默认值, 确保在父组件没有指定其值时，有一个默认值。
Greeting.defaultProps = {
  name: 'Stranger'
};

// 渲染出 "Hello, Stranger"：
ReactDOM.render(<Greeting />, document.getElementById('example'));
```



`propTypes` 类型检查发生在 `defaultProps` 赋值后，所以类型检查也适用于 `defaultProps`。



<br><hr><br>

#### 17. 非受控组件

使用非受控组件，此时表单数据则由DOM节点来处理，此时可通过`使用ref`来从DOM节点中获取表单数据。

非受控组件会将真实数据存储在DOM节点中。相较于受控组件会减少代码量，可快速编写代码。



一般针对受控组件，我们会传递`value`用于显示，对于非受控组件，我们一般会传递`defaultValue`作为初值，不再控制后续的更新。



需要注意的是，`<input type="file" />`始终是一个非受控组件，因为它的值只能由用户设置，而不能通过代码控制。此时可通过 `ref`添加对该组件的引用，通过`this.myRef.current.files[0].name`获取文件名。





<br><hr><br>

#### 18.   SPA单页应用的前端路由

SPA 是 single page web application 的简称，译为单页Web应用。 SPA 就是一个WEB项目只有一个 HTML 页面，一旦页面加载完成，SPA 不会因为用户的操作而进行页面的重新加载或跳转。 取而代之的是利用 JS 动态的变换 HTML 的内容，从而来模拟多个视图间跳转。

前端路由就是保证只有一个HTML页面，且与用户交互时不刷新和跳转页面的同时，为SPA中每个视图匹配一个特殊的url，在刷新、前进、后退和SEO时均通过这个特殊的 url 来实现。所以需要实现两个功能：

* 改变url 且不让浏览器向服务器发送请求
* 可以监听到url的变化

如此产生了 history和hash模式。

##### 1. Hash 模式

​	hash指的是url之后的 **`#`** 号以及后面的字符。hash值的改变不会导致浏览器像服务器发送请求。**而且 hash 的改变会触发 hashchange 事件，浏览器的前进后退也能对其进行控制。**



##### 2. History 模式

 	html5之前已经有了history对象，但是只可用于多页面的跳转， 在HTML5中新加了几个API：

```js
history.pushState();   // 添加新的状态到历史状态栈, 在保留现有历史记录的同时，将 url 加入到历史记录中。
history.replaceState(); // 用新的状态代替当前状态, 会将历史记录中的当前页面历史替换为 url。
history.state                // 返回当前状态对象
```

history.pushState() 和 history.replaceState() 可以改变 url 同时，不会刷新页面，所以在 HTML5 中的 histroy 具备了实现前端路由的能力。

​**history 的改变并不会触发任何事件，所以我们无法直接监听 history 的改变而做出相应的改变。** 因此手动控制时，需要罗列出可能触发history改变的所有情况，并进行拦截，变相监听history的改变。



```js
   但需要注意的是，history 在修改 url 后，虽然页面并不会刷新，但我们在手动刷新，或通过 url 直接进入应用的时候，服务端是无法识别这个 url 的。因为我们是单页应用，只有一个 html 文件，服务端在处理其他路径的 url 的时候，就会出现404的情况。
   所以，如果要应用 history 模式，需要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回单页应用的 html 文件。

```



<br><hr><br>

#### 19.  React 的API

##### 1. React组件

* React.Component 

  使用 ES6 classes方式定义 React 组件的基类，

  

* React.PureComponent

  React.PureComponent 相较于 React.Component  通过浅层对比prop 和 state的方式实现了 shouldComponentUpdate()

  

* React.memo

  React.memo为高阶组件，类似React.PureComponent，但它适用于函数组件，但不适用于class组件。如果函数组件在给定相同 props 的情况下渲染相同的结果，那么可以通过将其包装在 `React.memo` 中调用，复用最近一次的渲染结果。

  

##### 2. 转换元素:  几个操作元素的API

* createElement()

  ```js
  React.createElement(type, [props], [...children])
  ```

  使用`JSX`编写的代码将会被转换成使用 `React.createElement() `的形式。

* cloneElement() 

   以element 元素为样板克隆并返回新的 React元素，返回元素的props是将新的props与原始元素的props浅合并后的结果。新的子元素将取代现有的子元素，而**来自原始元素的key和ref将被保留**。

  ```js
  React.cloneElement(element, [props], [...children])
  
  // 等价于
  <element.type {...element.props} {...props}>{children} </element.type>
  ```

  

* isValidElement() ： 验证对象是否为React元素

  

* React.Children ： 提供了用于处理 this.props.children 不透明数据结构的实用方法。

  * React.Children.map()
  * React.Children.forEach()
  * React.Children.count()  返回children中的组件总数量
  * React.Children.Only()  验证children是否只有一个子节点
  * React.Children.toArray() 将 `children` 这个复杂的数据结构以数组的方式扁平展开并返回，并为每个子节点分配一个 key。

  

##### 3. React.Fragment :  可用于减少不必要嵌套的组件

​	用于在不额外创建DOM元素的情况下，让render() 方法中返回多个元素。

##### 4. Refs 

* React.createRef()  创建一个能够通过ref属性附加到React元素的 ref
* React.forwardRef()  创建一个React组件，该组件可以将其接受的ref属性转发到子组件中。



##### 5. Suspense:

使组件可以“等待”某些操作结束后，再进行渲染。目前，Suspense支持的使用场景是，通过 React.lazy 动态加载组件。

* React.lazy  

  允许定义一个动态加载的组件，有助于缩减bundle的体检，并演示加载在初次渲染时未用到的组件。

  ```js
  // 这个组件是动态加载的
  const SomeComponent = React.lazy(() => import('./SomeComponent'));
  ```

  使用 `React.lazy` 的动态引入特性需要 JS 环境支持 Promise。在 IE11 及以下版本的浏览器中需要通过引入 polyfill 来使用该特性。

  

* React.Suspense

  可以在组件树的某些子组件尚未渲染时，渲染替代的组件。



##### 6. Hook

Hook作为React 16.8的新增特性，可以在不编写class的情况下使用state以及其他的React特性。



<br><hr><br>

#### 20.  ReactDOM

```js
// 安装
yarn add react-dom 

// 使用
import ReactDOM from 'react-dom'      // es6
var ReactDOM = require('react-dom')   // es5
```



##### 1. ReactDOM.render()

```js
ReactDOM.render(element, container[, callback])
```

在提供的container中渲染一个React元素，并返回对该组件的引用(无状态组件会返回null)，若已经渲染过，则会执行更新操作，必要时会改变DOM



##### 2. ReactDOM.creatPortal()

```js
ReactDOM.createPortal(child, container)
```

`Portal` 将提供一种将子节点渲染到 DOM 节点中的方式，该节点存在于 DOM 组件的层次结构之外。



<br><hr><br>

#### 21.  DOM元素

React中，所有的DOM特性和属性都是小驼峰命名，但是，`aria-*`以及`data-*`属性例外，一律使用小写字母命名。

React与html的属性差异

* checked  

* className  用于指定 CSS的 class

* dangerouslySetInnerHTML

  dangerouslySetInnerHTML 是 React 为浏览器DOM提供 innerHTML的替换方案，设置时，需要向其传递包含 key 为 __html的对象。

* style 

  style 接受一个采用小驼峰命名属性的 JavaScript 对象，而不是 css 字符串，React会自动添加"px"后缀到内联样式为数字的属性后。



<br><hr><br>

#### 22. 合成事件(SyntheticEvent)

`SyntheticEvent` 实例作为浏览器的原生事件的跨浏览器包装器，除兼容所有浏览器外，它还拥有和浏览器原生事件相同的接口，包括 `stopPropagation()` 和 `preventDefault()`。若想使用浏览器的底层事件，则可以通过`nativeEvent`属性获取。

`SyntheticEvent` 由于是合并而来，会被重用，`SyntheticEvent` 的所有属性会在触发事件的回调函数被调用后无效，所以不支持异步访问，若你想异步访问事件属性，你需在事件上调用 `event.persist()`，此方法会从池中移除合成事件，允许用户代码保留对事件的引用。

默认是在冒泡阶段触发事件处理函数，若想在捕获阶段触发，则需要使用 `onClickCapture`，而不是 `onClick` 。


在React中，处理事件不需要再使用 `addEventListener` 为已创建的DOM元素添加监听器，只需要在元素初始渲染的时候添加监听器就可以。

React支持的事件参考[这里](https://react.docschina.org/docs/events.html#supported-events)，其中包括但不限于 剪贴板事件、鼠标事件、滚轮事件、键盘事件、UI事件、表单事件等。

React 中，若是想[阻止事件调用太快或者太多次](https://zh-hans.reactjs.org/docs/faq-functions.html#how-can-i-prevent-a-function-from-being-called-too-quickly-or-too-many-times-in-a-row)，则可以引入节流和防抖。

* 节流阻止事件在指定的时间窗口内只可以调用一次
```js
import throttle from 'lodash.throttle';

class LoadMoreButton extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.handleClickThrottled = throttle(this.handleClick, 1000);
  }

  componentWillUnmount() {
    this.handleClickThrottled.cancel(); // 取消延迟回调
  }

  render() {
    return <button onClick={this.handleClickThrottled}>Load More</button>;
  }

  handleClick() {
    this.props.loadMore();
  }
}

```

* 防抖确保函数不会在上一次被调用之后一定量的时间内被执行。当必须进行一些费时的计算来响应快速派发的事件时（比如鼠标滚动或键盘事件时），防抖是非常有用的。
```js
import debounce from 'lodash.debounce';

class Searchbox extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.emitChangeDebounced = debounce(this.emitChange, 250);
  }

  componentWillUnmount() {
    this.emitChangeDebounced.cancel();
  }

  render() {
    return (
      <input
        type="text"
        onChange={this.handleChange}
        placeholder="Search..."
        defaultValue={this.props.value}
      />
    );
  }

  handleChange(e) {
    // React pools events, so we read the value before debounce.
    // Alternately we could call `event.persist()` and pass the entire event.
    // For more info see reactjs.org/docs/events.html#event-pooling
    this.emitChangeDebounced(e.target.value);
  }

  emitChange(value) {
    this.props.onChange(value);
  }
}

```



<br><hr><br>

#### 23. React Hook

Hook 是 React 16.8 的新增特性。要启用 Hook，所有 React 相关的 package 都必须升级到 16.8.0 或更高版本。如果忘记更新诸如 React DOM 之类的 package，Hook 将无法运行。与class组件的不同在于，函数组件本身即为class 组件的 render()。 
* 只能在函数的最外层调用 Hook
* 只能在React的函数组件中调用 Hook

##### 1. useState()

`useState` 可通过在函数组件中调用它来给组件添加一些内部的state, React会在重复渲染时保留这个state。`useState` 的唯一参数就是**initialState** ，这个 **initialState** 参数只有在**第一次渲染**时会被用到。

 state 只在组件首次渲染的时候被创建。在下一次重新渲染时，useState 返回给我们当前的 state。

 * setState 的函数式更新

    如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。该函数将接收先前的 state，并返回一个更新后的值。避免了显示引用当前的state。

    ```js
      function Counter({initialCount}) {
        const [count, setCount] = useState(initialCount);
        return (
          <>
            Count: {count}
            <button onClick={() => setCount(initialCount)}>Reset</button>
            <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
            <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
          </>
        );
      }
    ```

    与 class 组件中的 setState 方法不同，useState 不会自动合并更新对象。你可以用函数式的 setState 结合展开运算符来达到合并更新对象的效果。

    ```js
      setState(prevState => {
        // 也可以使用 Object.assign
        return {...prevState, ...updatedValues};
      });
    ```


 ##### 2. useEffect()

 `Effect Hook` 可以在函数组件中执行副作用(改变 DOM、添加订阅、设置定时器、记录日志等操作)。useEffect 会在每次渲染后都执行，React 保证了每次运行effect(effect 即为传递给useEffect()的函数)的同时，DOM已经更新完毕。

 一旦 effect 的依赖发生变化，它就会被重新创建。**即每次我们重新渲染，都会生成新的 effect，替换掉之前的**，传递给 useEffect 的函数每次都不同，这样才可以保证每次都可以获取最新的数据。

当effect是订阅外部数据源时，需要进行清除操作(如取消订阅等)，以防内存泄露。如果 effect 返回了一个函数，React 则会在组件卸载时执行effect的清除操作，此时则会调用effect返回的函数。

每次渲染之后都执行清理或执行effect可能会导致性能问题，此时只需要传递数组作为useEffect的第二个可选参数，只有当数组中存放的变量变化时，才会执行清理或执行effect。
  * 需要确保数组中包含了所有外部作用域中会随时间变化并且在 effect 中使用的变量
  * 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（[]）作为第二个参数。


经过测试发现，每次修改内部的state，组件重新渲染时，函数组件内所有的**同步语句会先执行**，然后**执行useEffect中return的清理函数**，最后会**执行effect**。所以若在useEffect中使用了定时器，只需在return的函数中清除定时器即可，**在组件卸载以及组件重新渲染时都会执行 effect 所 return的函数**，所以为了不重复停启定时器，当在useEffect中使用了定时器时，可以采用下述方案。



###### 2.1 **数组中的变量频繁变化时如何操作？**

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // 不依赖于传入的count变量，引用prevCount
    }, 1000);
    return () => clearInterval(id);
  }, []); // 我们的 effect 不依赖于组件作用域中的任何变量

  return <h1>{count}</h1>;
}
```
如上述示例，effect 原本依赖了count，但是若是直接将count放入effect的数组参数中，则会在count变化时导致定时器被重置，采用函数式更新setState的方式，则可以引用到最新的count，而不用重置定时器。[参考](https://zh-hans.reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)



##### 3. Hook 规则

使用 `eslint-plugin-react-hooks` 来强制执行**只在最顶层使用Hook**以及**只在React函数中使用Hook**这两条规则。
```js
// 安装依赖
yarn add eslint-plugin-react-hooks -D

// 在ESLint中配置规则
{
  "plugins": [
    "react-hooks"
  ],
  "rules": {
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖，此规则会在添加错误依赖时发出警告并给出修复建议。
  }
}
```

React依靠Hook的调用顺序确定state的对应关系： 
```js
function Form() {
  const [name, setName] = useState('Mary');

  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  const [surname, setSurname] = useState('Poppins');

  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

}


// 首次渲染
useState('Mary')           // 1. 使用 'Mary' 初始化变量名为 name 的 state
useEffect(persistForm)     // 2. 添加 effect 以保存 form 操作
useState('Poppins')        // 3. 使用 'Poppins' 初始化变量名为 surname 的 state
useEffect(updateTitle)     // 4. 添加 effect 以更新标题


// 二次渲染
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
useEffect(persistForm)     // 2. 替换保存 form 的 effect
useState('Poppins')        // 3. 读取变量名为 surname 的 state（参数被忽略）
useEffect(updateTitle)     // 4. 替换更新标题的 effect

// ...
```

##### 4. 自定义Hook

可通过自定义Hook 将组件逻辑提取到可重用的函数中。



##### 5. useContext()
useContext 的优势在于在使用context时不需要再使用`<Consumer>`来包裹子组件了。需要注意的是：useContext只是可以让我们读取 context 的值以及订阅 context 的变化。使用时仍然需要在上层组件树中使用 `<MyContext.Provider>` 来为下层组件提供 context。

```js
// context.js
import React, { useState } from 'react'
// 1. 创建 context
export const myContext = React.createContext()

// 2. 创建 Provider
export const MyProvider = props => {
  let [username, handleChangeUsername] = useState('')
  return (
    <myContext.Provider value={{username, handleChangeUsername}}>
      {props.children}
    </myContext.Provider>
  )
}

// 3. 创建 Consumer
export const MyConsumer = myContext.Consumer

```
接下来可以在子组件中使用 Context。一定要注意，**useContext的参数是 context 本身！！！**

```js
import React, { useContext, Fragment }  from 'react' 
import { MyProvider, myContext } from './context' 

function F1() {
  const { username, handleChangeUsername } = useContext(myContext);
  const handleChange = e => {
    handleChangeUsername(e.target.value);
  };
  return (
    <div>
      <div>user: {username}</div>
      <input onChange={handleChange} />
    </div>
  );
}

function App() {
  return (
    <Fragment>
      <MyProvider>
        <F1 />
      </MyProvider>
    </Fragment>
  );
}

```

##### 6. useReducer()
useReducer 作为 useState 的替代方案，接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。可以将初始值作为第二个参数传入 useReducer，也可以将init函数传入第三个参数，此时初始值则为 **`init(initialArg)`**。

```js
 const [state, dispatch] = useReducer(reducer, initialArg, init);
```


##### 7. useCallback()
把回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。

```js
const memoizedCallback = useCallback(
  () => doSomething(a, b) ,
  [a, b],
);
```

##### 8. useMemo()
把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。注意：传入 useMemo 的函数会在渲染期间执行，所以在该传入函数内部不要进行任何副作用的操作。

```js
const memoizedValue = useMemo(
  () => computeExpensiveValue(a, b), 
  [a, b]
);
```


##### 9. useRef()

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变，类似于一个 class 的实例属性。

```js
  const refContainer = useRef(initialValue);
```

useRef 创建了一个普通的 JavaScript对象，形如 `{current : ...}`，useRef与直接新建一个这样的对象的区别在于，每次渲染时，useRef返回的是同一个ref对象。

变更 `.current`属性并不会引起组件重新渲染，如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用回调 ref 来实现。

可以用 useRef 存储上一轮的props和state。




























































