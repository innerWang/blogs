# setState 源码理解

参考的[16.8.4](https://github.com/facebook/react/tree/b87aabdfe1b7461e7331abb3601d9e6bb27544bc)版本 ，位于 `react/packages/react/src/ReactBaseClasses.js`

```js
const emptyObject = {};
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  // We initialize the default updater but the real one gets injected by the
  // renderer.
  this.updater = updater || ReactNoopUpdateQueue;
}

 /*
 * @param {object|function} partialState Next partial state or function to
 *        produce next partial state to be merged with current state.
 * @param {?function} callback Called after state is updated.
 */
Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};
```

可以看到组件类有三个参数，其中`refs`默认是一个空对象。而其中的updater则是下文的 `ReactNoopUpdateQueue`

```js
const ReactNoopUpdateQueue = {
	isMounted: function(publicInstance) {
    return false;
  },
  enqueueForceUpdate: function(publicInstance, callback, callerName) {
    warnNoop(publicInstance, 'forceUpdate');
  },
  enqueueReplaceState: function(publicInstance,completeState,callback,callerName) {
    warnNoop(publicInstance, 'replaceState');
  },
	enqueueSetState: function(publicInstance,partialState,callback,callerName,) {
    warnNoop(publicInstance, 'setState');
  },
};
```

`ReactNoopUpdateQueue` 提供了 update、 replaceState、 setState三个方法，主要是调用了 warnNoop警告方法。

```js
function warnNoop(publicInstance, callerName) {
  if (__DEV__) {
    const constructor = publicInstance.constructor;
    const componentName =
      (constructor && (constructor.displayName || constructor.name)) ||
      'ReactClass';
    const warningKey = `${componentName}.${callerName}`;
    if (didWarnStateUpdateForUnmountedComponent[warningKey]) {
      return;
    }
    warningWithoutStack(
      false,
      "Can't call %s on a component that is not yet mounted. " +
        'This is a no-op, but it might indicate a bug in your application. ' +
        'Instead, assign to `this.state` directly or define a `state = {};` ' +
        'class property with the desired state in the %s component.',
      callerName,
      componentName,
    );
    didWarnStateUpdateForUnmountedComponent[warningKey] = true;
  }
}
```