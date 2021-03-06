# JS异步操作

## 目录

#### 1. [Promise](#promise)
* [语法](#promise-grammer)
* [应用举例](#promise-example)
* [Promise.prototype.then()](#promise-then)
* [Promise.prototype.catch()](#promise-catch)
* [Promise.all()](#promise-all)
* [Promise.race()](#promise-race)

#### 2. [async/await](#async)
* [语法](#async-grammer)
* [错误处理机制](#async-handleErr)

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
`Promise` 是一个容器，保存着某个未来才会结束的事件的结果，从语法上而言，Promise 是一个对象，对象里面存储着一个状态，这个状态对应着异步操作的结果，可通过它获取异步操作的消息。 

#### 1.1 <a name="promise-grammer">语法</a>
```js
  // 创建一个 Promise 实例
  const promise = new Promise(function(resolve,reject){  /* do sth...  */   });
```
可以看到，创建一个 Promise 对象仅需要调用 Promise 构造函数，该构造函数的参数是一个带有 resolve 和 reject 两个参数的函数，当 resolve 和 reject 函数被调用时，分别将 promise 的状态改为 fulfilled 或 rejected ，同时会将异步操作的**结果**或**错误**作为参数传递出去。

<br>

#### 1.2 <a name="promise-example">应用举例</a>
```js
  function getJSON(ip){
    return new Promise(function(resolve,reject){
      var xhr = new XMLHttpRequest();
      xhr.open('GET','/xxxx?ip='+ip,true);
      xhr.onload = function(){
        if(xhr.status >= 200 && xhr.status<300 || xhr.status == 304){
          resolve(Json.parse(xhr.responseText))
        }
      }

      xhr.onerror = function(){
        reject('连接失败')
      }

      setTimeout(function(){
        xhr.send();
      },Math.random() * 10);
    })
  }

  getJSON('10.1.2.3').then((json)=>console.log(json))
                   .catch((err)=>console.log(err))
	
```
上例是一个典型的使用Promise进行异步操作的例子， getJSON 函数返回一个 Promise 对象，在该对象内部定义了resolve 和 reject 两个回调函数的执行时机，当获取数据成功时，调用 resolve 回调函数，传递数据；当发生错误时，调用 reject 回调函数，传递错误信息。 然后使用 then 方法和 catch 方法指定回调函数，分别接收 resolve 和 reject 传递过来的内容。

<br>

#### 1.3  <a name="promise-then">Promise.prototype.then()</a>
`then` 方法是定义在原型对象 `Promise.prototype`上的，它主要作用是为 Promise 实例添加状态改变时的回调函数。 then 方法的第一个参数是 resolved 状态的回调函数，第二个参数(可选)是 rejected 状态的回调函数。
<br><br>
`then` 方法返回的是一个新的 Promise实例，因为可以采用链式写法 ，即`promise.then().then()`, 其中，第一个回调函数的结果，是第二个回调函数的参数。当前一个回调函数返回的是一个 Promise 对象时，则后一个回调函数会等待该 Promise 对象的状态发生变化，才会被调用。
```js
getJSON(songId).then(
    data => getJSON(data.lyricsId)
  ).then(
    lyrics => console.log("resolved: ",lyrics),
    err => console.log("rejected: ",err)
  );

```
如上，当第一个 then 方法返回的 Promise 发生状态的变化时，才会对应去调用第二个 then 方法所指定的 resolved 以及 rejected 回调函数。

<br>

#### 1.4 <a name="promise-catch">Promise.prototype.catch()</a>
`Promise.prototype.catch` 方法是 `.then(null,rejected)`或 `.then(undefined,rejected)`的别名，用于指定发生错误时的回调函数。
```js
getJSON('xxx').then(()=>{}).catch( e => console.log('err: ',e))
```
上例中 getJSON 方法返回 Promise 对象，当其状态变为 resolved 时调用 then 指定的回调函数，当其状态变为 rejected 时，调用 catch 指定的回调函数。 需要注意以下几点：
* 在执行 then 指定的回调函数过程中，若是抛出错误，也会被 catch 捕获。
* 如果 Promise 的状态已经变为 resolved，再抛出错误则无效 。 因为 Promise 的状态一旦改变，就永久保持该状态，不会再变。
* Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也即、错误总是会被下一个 catch 捕获。一般来说，不在 then 方法里面定义 reject 状态对应的回调函数，而是使用 catch 方法。 这样还可以捕获 then 方法执行中的错误。
* catch 里面也可以抛出错误，需要后续定义 catch 捕获该错误。

<br>

#### 1.5 <a name="promise-all">Promise.all()</a>
该方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
```js
  const p = Promise.all([p1,p2,p3]);
```
Promise.all 方法的参数可以不是数组，但必须有 Iterator 接口，且返回的每个成员都是 Promise 实例。
1. 只有 `p1`、`p2`、`p3` 的状态都变为 fulfilled，`p` 的状态才会变为 fulfilled，此时 `p1`、`p2`、`p3` 的返回值构成一个数组，传递给 `p` 的回调函数。
2. 只要 `p1`、`p2`、`p3` 之中有一个被 rejected， `p` 的状态就会变为 rejected， 此时第一个被 reject 的实例的返回值，会传递给 `p` 的回调函数。

<br>

#### 1.6 <a name="promise-race">Promise.race()</a>
该方法同样是用于将多个 Promise 实例，包装成一个新的 Promise 实例。
```js
  const p = Promise.race([p1,p2,p3]);
```
只要`p1`、`p2`、`p3` 三个实例中，有一个实例的状态率先发生改变，`p`的状态就会跟着改变，率先改变的 Promise 实例的返回值，就传递给 p 的回调函数。

<br>

<br>[top](#目录)

## 2. <a name="async">async/await</a>

ES2017 标准引入了 async 函数，使得异步操作变得更加方便。 `async`用于修饰函数，表明函数里有异步操作，`await`仅能在 async 函数体内使用，用于等待一个 Promise 对象的结果并返回，如果等待的不是 Promise 对象，则返回该值本身。
<br><br>
调用 async 函数时，会返回一个 Promise 对象，当 async 函数返回一个值时， Promise 的 resolve 方法会负责传递这个值，当 async 函数抛出异常时， Promise 的 reject 方法会传递这个异常值。

#### 2.1 <a name="async-grammer">语法</a>
```js
  // 定义一个异步函数，隐式返回一个 Promise 对象
  const fn = async (songid) => {
    const albumid = await getAlbumID(songid);
    const nextsong = await getRandomSong(albumid);
    return nextsong;
  }

  fn(1579).then(song => console.log(song)).catch(err => console.log(err))

```
调用异步函数，必须等到内部所有 await 命令后面的 Promise 对象执行完，其返回的 Promise 对象才会发生状态改变，除非先遇到 return 语句或者抛出错误， 如上例，异步函数 fn 内部需要执行完获取唱片ID，根据唱片ID获取下一首歌的信息，才会调用 then 方法定义的回调函数，输出返回的信息。

#### 2.2 <a name="async-handleErr">错误处理机制</a>
任何一个 await 语句后面的 Promise 对象变为 reject 状态，那么整个 async 函数都会中断执行。
```js
 async function fn(){
   await Promise.reject('err');
   await Promise.resolve('ok'); // 不会执行
 }

 fn().then( data => console.log(data)).catch(err => console.log(err))  //'err'
```
此时，前一个await 语句异步操作状态变为 rejected, 后面的异步操作会被中止，此时若我们期望即使前一个操作失败，也不要中断后面的操作，则可以将第一个 await 放在 try..catch 结构中，即不管第一个异步操作成功与否，第二个都会执行。
```js
 async function fn(){
   try{
      await Promise.reject('err');
   }catch(e){
      console.log('first await err: '+e)
   }
   return await Promise.resolve('ok'); 
 }

 fn().then( data => console.log(data)).catch(err => console.log(err))  // 'first await err: err' 及 'ok'
```

* 使用注意点
  * await 后面的 Promise 对象的状态可能会变为 rejected， 所以最好把 await 命令放在 try..catch 代码块中；
  * 多个 await 命令后面的异步操作若是没有依赖关系，最好让他们同时触发
  ```js
  // 此种写法会先执行第一个await语句再执行第二个，比较耗时
   let a = await getA();
   let b = await getB();

  //此种写法，会同时触发getA() 和 getB() ,缩短程序的执行时间
  //Promise.all 会让多个异步操作同时执行
  let [a,b] = Promise.all([getA(),getB()]);

  ```
  * await 命令仅可用在 async 函数中，用在普通函数中，会报错。









<br>[top](#目录)




















