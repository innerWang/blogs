# 深入理解 EventLoop


## <a name="catalog">目录</a>

#### 1. [什么是Event Loop](#one)
#### 2. [浏览器的Event Loop](#two)
#### 3. [NodeJS的Event Loop](#three)


<br><hr><br>


## 1. <a name="one">什么是Event Loop</a>

事件循环(Event Loop) 是让 JavaScript 做到既是单线程但又不会阻塞的核心机制。
浏览器和NodeJS对于 Event Loop的实现并不一致，接下来将分别讲解。


## 2. <a name="two">浏览器的Event Loop</a>

浏览器环境下，所有的同步任务在主线程上执行，形成一个执行栈；此外还会维护一个任务队列，当异步任务有了结果，就会在任务队列中放一个事件；一旦执行栈中的所有同步任务执行完毕，系统就会读取任务队列，将事件对应的异步任务结束等待状态，放入执行栈执行。

异步任务必须指定回调函数，当主线程开始执行异步任务时，即为执行对应的回调函数。

实际上，在浏览器中总共有两种任务：
* **MacroTask(宏任务)** ： script(整体代码), setTimeout, setInterval, setImmediate（node独有）, I/O, UI rendering
* **MicroTask(微任务)** ： process.nextTick（node独有）, Promises, Object.observe(废弃), MutationObserve

event loop至少有一个任务队列，里面存放的是宏任务，此外event loop还有一个微任务队列。<br>
event loop 还有一个初值为 null 的`currently running task` 以及一个初值为false的`performing a microtask checkpoint flag`。

[HTML标准](https://www.w3.org/TR/html5/webappapis.html#event-loops-processing-model)定义的 event loop的步骤如下：

1. 任选一个任务队列，选定任务队列中最先进入的任务，若没有要选择的任务，则跳到下面的 Microtasks步骤；
2. 将上一步骤选定的任务设置为 event loop 的当前任务；
3. 执行选定的任务；
4. 将event loop 的当前任务设置为null；
5. 从任务队列中删除上述运行的任务；
6. Microtasks步骤： 执行微任务检查点(perform a microtask checkpoint)；
7. 更新渲染

当执行微任务检查点时，如果`performing a microtask checkpoint flag`为`false`, 则会执行以下步骤：

1. 将`performing a microtask checkpoint flag` 置为`true`
2. 处理微任务队列，若微任务队列为空，则跳到下面的 Done步骤；
3. 选取最先进入微任务队列的微任务；
4. 将上一步骤选定的任务设置为 event loop 的当前任务；
5. 执行选定的任务；
6. 将event loop 的当前任务设置为null；
7. 从任务队列中删除上述运行的任务，再返回步骤2(Microtask queue handling);
8. Done
9. 清除索引数据库的事务
10. 将`performing a microtask checkpoint flag` 置为`false`

由上可以看出，在事件循环中，用户代理会不断的从任务队列中取宏任务执行，每执行完一个宏任务则会检查微任务队列是否为空，若不为空，则会将整个微任务队列中的微任务全部执行完，然后再进行下一个循环，取下一个task执行...


分析以下示例:

```js
async function async1() {
  console.log(1);
  await async2();
  console.log(2);    
}
async function async2() {
  console.log(3)
}

async1();

new Promise(function (resolve) {
  console.log(4);
  resolve();
}).then(function () {
  console.log(5);
});

```	

上述示例中，需要注意以下几点：
* new Promise(fn) 中，带有参数resolve及reject的**fn是立即执行**的；
* 当promise变为 resolved/rejectd 状态时，会执行 then()所指定的回调，此时即会把回调函数放入微任务队列；
* 当遇到 await时，需要转换为 promise 进行分析；

对代码进行分析，执行 async1()时，先输出1，将`console.log(2)`记为 `f1 = ()=>console.log(2)`，则 `await async2();console.log(2)` 可以转换为 `Promise.resolve(async2).then(f1)`, 即为输出3，由于 async2的状态变为 resolved，此时会将 f1 放入微任务队列；接着执行后面的代码，会先输出4，然后promise的状态变为 resolved，则会执行then，将 `console.log(5)`对应的回调函数放入微任务队列，此时主线程执行栈中的代码执行完毕，无宏任务队列，转去执行微任务队列中所有的任务，依次输出2 、5 。<br>
即最终结果为: `1 3 4 2 5`;




## 3. <a name="thress">NodeJS的Event Loop</a>









