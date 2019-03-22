# React Hooks -- useEffect

Hooks 是React 16.8的新特性。可以让我们在不用写class的情况下使用state以及其他React的特性。

Effect Hook 允许我们在函数组件中执行副作用。

数据获取，设置订阅，以及手动改变React组件的DOM都是副作用的示例。无论你是否将这些操作叫做副作用(或仅仅是“作用”)，你之前可能已在你的组件中执行过这些操作。

React组件中有两种副作用，一种是不需要清理的，一种是需要清理的，让我们来看看区别。

### Effects Without Cleanup

有时候，我们需要在 React 更新 DOM 之后运行一些额外的代码。网络请求，人为引起的DOM突变，以及日志记录是常见的不需要清理的作用(effects)的示例。我们之所以这样说是因为我们可以运行这些代码然后马上忘记他们，让我们来比较一下 classes 以及 Hooks 是如何表达这样的副作用的。

#### Example Using Classes

在 React的class组件中，render方法不应该引起副作用，我们一般希望在React已经更新DOM后执行副作用。

这就是为什么在 React class组件中，我们会将副作用放到 componentDidMount 以及 componentDidUpdate中。回到我们的示例，这是一个在React更新DOM后立刻更新document title的React计数class组件。

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}

```
需要注意我们不得不在类组件的两个生命周期方法中复制代码。

这是因为大多数情况下我们希望无论是在组件刚mount或者刚update时，都可以执行相同的副作用。从概念上讲，我们希望在每次render之后发生，但是React类组件并没有这样的方法，我们可以提取一个单独的方法，但是我们仍需要在两个地方调用它。

接下来让我们看看是如何使用 useEffect Hook 做相同的事情的。


#### Example Using Hooks

先看一下示例：

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

```

useEffect做了什么？？ 通过使用这个Hook，你会告诉React你的组件需要在render知乎做一些事情。R	eact会记住你传递的函数(我们将其记为“effect”)，会在执行完DOM的更新之后调用这个函数。在这个effect中，我们设置 document title，但是我们也可以执行数据的获取或者调用一些必须执行的API。

为什么是在组件内部调用useEffect？ 将 useEffect放在组件内部会让我们可以直接从 effect中访问 count 状态变量(或任何props)。我们不需要使用一个特殊的API来读取它，它已经存在函数组件的内部了。 Hooks使用JS闭包，并避免在JS已经提供解决方案的情况下引入特定于React的API。

useEffect 在每次render后都会执行吗？ 是的！默认情况下，它会在第一次render以及每一次update之后运行。为了方便理解，我们认为effects发生在“after render”之后而不是“mounting”或者“updating”。React会保证在运行effects时DOM已经更新。

#### Detailed Explanation(更详细的解释)

现在我们对effects了解的多了一些，下面这些代码就更好理解了。

```js

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

```
上述代码中，我们声明了 count 状态变量，然后我们通过给useEffect传一个函数来告诉React我们需要使用一个 effect，在我们的effect中，我们通过使用 浏览器API`document.title`来设置文档标题，由于 count变量是函数组件内部的全局变量，我们可以在effect中读到最新的count。当React渲染组件时，它会记得我们使用的effect，会在更新DOM之后运行我们的effect。这会在每一次render时发生，包括第一次。

有经验的JS开发者会注意到传递给 useEffect 的函数在每次render都不一样。这是故意的。实际上，这就是让我们可以从 effect的内部读取count的值且不用担心它变得陈旧的原因。每一次我们重新渲染时，我们调度了一个不同的effect来代替之前的那一个。以这种方式使得effects更像是渲染结果的一部分，每个effect属于一个特定的render。我们将在本页的后面更清楚的看到为什么这么做是有用的。

Tip: 不像componentDidMount 或者 componentDidUpdate ，使用 useEffect调度的 effects 不会阻塞浏览器更新屏幕。这会让你的应用更具有响应性。大多数effects不需要同步发生。在不常见的情况下（如测量布局），有一个单独的与 useEffect 的API一致的 useLayoutEffect Hook。 

### Effects With Cleanup

之前，我们看了怎样去表示不需要清理的副作用，然而，有一些副作用需要清理。例如，我们可能需要对一些外部数据资源设置一个订阅，在这种情况下，清理变得十分重要，如此我们才不会引入内存泄漏，接下来我们来比较一下我们怎样使用 classes以及Hooks完成这件事情的。

#### Example Using Classes

在React类组件中，我们一般在 componentDidMount 中设置订阅，然后在componentWillmount中清理它。例如，我们有一个 ChatAPI 模块，功能是订阅一个朋友的在线状态，下面是我们如何使用类组件订阅和展示状态的示例：

```js
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

请注意 componentDidMount 以及 componentWillUnmount 是如何相互镜像的。生命周期方法迫使我们拆分这个逻辑，即使他们中的代码从概念上讲与相同的effect有关。

Note: 眼尖的读者会注意到这个例子还需要一个 componentDidUpdate 方法才能完全正确。我们现在先忽略这个问题，在后面的部分会讲到这个。

#### Example Using Hooks

让我们来看看是如何使用Hooks来写这个组件的。

你可能会向我们需要一个单独的effect去执行清理，但是添加和删除订阅的代码关联如此紧密，所以uesEffect旨在将他们放在一起，如果你的 effect 返回一个function，React会在清理时运行它。

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

```

我们为什么要在effect中返回一个函数？ 这是 effects的可选机制。每一个effect会返回一个在它之后清理它的函数。这使得我们可以把增加和删除订阅的逻辑二者紧密连在一起。他们都是相同effect的一部分。

React什么时候清理一个 effect？ React会在组件卸载时执行清理。然后，我们之前了解到，effects 会在每次render之后运行而不是仅运行一次。这也就是为什么React会在下次运行effect之前清理前一次渲染的effects的原因。我们将会讨论为什么这么做会避免错误以及如何在以后引起性能问题时退出这个行为。

Note: 我们不需要从effect中返回命名函数。我们在这把它称为 cleanup 是为了表明它的目的，但是你也可以返回一个箭头函数或者给它取别的名。

#### 概括

我们已经知道 useEffect 允许我们在渲染一个组件后执行不同种类的副作用。一些需要清理的副作用会返回一个函数。

```js
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```
不需要清理的副作用则不用返回任何东西。

```js
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

Effect Hook 将两种情况统一到一个API。

### Tips for Using Effects

我们将深入看一下有经验的React用户会好奇的 useEffect 的一些方面。可以有需要的时候来看一下。

#### Tip: Use Multiple Effects to Separate Concerns

Hooks的动机中我们强调的一个问题是类组件的生命周期方法经常包含不相关的逻辑，但是相关的逻辑会被打散到几个不同的方法中。下面是一个之前提到的组合了计数器和展示好友状态的逻辑的例子：

```js
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```













































