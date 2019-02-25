# JS异步操作

## 目录

### 1. [Promise](#promise)
### 2. [async/await](#async)

<br>

## 引言

当我们想要依次执行三个函数时，通常的做法如下：
```js
  const fn1 = (callback)=>{setTimeout(()=>{console.log('fn1');callback && callback()},1000)}
  const fn2 = (callback)=>{setTimeout(()=>{console.log('fn2');callback && callback()},1000)}
  const fn3 = ()=>{setTimeout(()=>console.log('fn3'),1000)}

  fn1(function(){
    fn2(function(){
      fn3()
    })
  })
```
如上所示，当依次执行的步骤越多时，嵌套就会越深，引发回调地狱，而Promise的出现解决了这个问题。

## 1. <a name="promise">Promise</a>

<br>[top](#目录)

## 2. <a name="async">async/await</a>

<br>[top](#目录)

