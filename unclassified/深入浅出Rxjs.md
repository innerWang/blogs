

## 1. 函数响应式编程



函数式编程对函数的使用所需要求：

* 声明式(Declarative)，与之相对的叫命令式编程(Imperative Programming)
  * 如：不管传入什么数据，都返回数据乘以2的结果
* 纯函数(Pure Function)
  * 如： 并不修改传入参数，而是对传入参数进行运算返回新的数据
  * 需满足条件1：函数的执行过程完全由输入参数决定，不会受除参数之外的任何数据影响
  * 需满足条件2：函数不会修改任何外部状态，如修改全局变量或者传入的参数对象
  * 若函数期望返回值x，将x作为参数传入函数，还能得到x，那么就是一个纯函数
  * 测试驱动开发(Test-Driven Development)：被测函数是纯函数，单元测试可轻松达到100%的覆盖率
* 数据不可变性 (Immutability)
  * 对象、数组等就不是Immutable数据类型，
  * 可使用Immutable.js 这类库来辅助实现 Immutable特性







函数式编程和面向对象编程的对比：

* 面向对象的思想：把数据封装在类的实例对象中，通过调用实例方法操作数据，代码属于黑盒，维护困难
* 函数式编程：数据和函数分离，函数可以处理数据，得到新数据来作为运算结果



响应式编程：

* 可以参考excel的数据统计，某一个单元格的数据变化时，求和就会跟着变化.

  

Reactive Extension ，

* 也叫 ReactiveX ，简称 Rx，指的是实践响应式编程的一套工具。
* Rx有很多语言版本的实现，RxJS就是Rx的JavaScript语言实现。
* Rx诞生的主要目的是为了解决异步处理的问题。



RxJS的特点：

* 数据流抽象了很多现实问题
* 擅长处理异步操作
* 把复杂问题分解成简单问题的组合

RxJS 对数据采用“推”的处理方式，当一个数据产生时，被推送给对应的处理函数，处理函数不需要关心数据是从同步产生的还是异步产生的。

学习RxJS 就是学习如何组合操作符来解决复杂问题。



## 2. RxJS 入门

V4 的代码位于 https://github.com/Reactive-Extensions/RxJS

V5/V6的代码位于 https://github.com/ReactiveX/rxjs

Tree-Shaking用于删除无用代码，仅对import的导入产生作用，对于CommonJS的require函数导入方式不起作用(require可以写在if语句中)，对RxJS也不起作用(Rx.js中全是require导入(v5!!!))。

V6版本采用了import 导入。



**Rxjs** 是一个基于 observable 序列编写异步和基于事件的程序的库，它提供了核心类型 Observable 以及附属类型 Observer、Schedulers、Subjects 以及 map、filter 等操作符。

- **Observable (可观察对象):** 表示一个概念，这个概念是一个可调用的未来值或事件的集合。
- **Observer (观察者):** 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。
- **Subscription (订阅):** 表示 Observable 的执行，当执行`observable.subscribe(observer)`之后，其返回值即为`subscription`，可使用`subscription.unsubscribe()`取消 Observable 的执行。
- **Operators (操作符):** 采用函数式编程风格的纯函数 (pure function)，使用像 `map`、`filter`、`concat`、`flatMap` 等这样的操作符来处理集合。
- **Subject (主体):** 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。
- **Schedulers (调度器):** 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 `setTimeout` 或 `requestAnimationFrame` 或其他。



```js
// 从 v6开始，导入方式如下
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
```





### 2.1 Observable 和 Observer

`Observable`即为“可以被观察的对象”，即“可被观察者”，`Observer`就是“观察者”，通过`Observable`对象的函数`subscribe`来连接两者。

`RxJS`中数据流就是Observable 对象，Observable 是观察者模式和迭代器模式的组合。



#### 2.1.1. 观察者模式

* 需要解决的问题是： 在一个持续产生事件的系统中，如何分割功能，让不同模块只需要处理一部分逻辑。

* 方法： 将逻辑分为`发布者(Publisher)`和`观察者(Observer)`
  * **发布者**只负责产生事件，通知所有注册的观察者，不关心事件如何被处理；
  * **观察者**可以被注册上某个发布者，接收到事件就处理，不关心数据如何产生。

`Observable`  对象就是一个发布者，通过这个对象的`sbuscribe`函数即可以把这个发布者和某个观察者(`Observer`)连接起来。

* 如何使用？
  * 如何产生事件，这是发布者的责任，是Observable对象的工作
  * 如何响应事件，是观察者的责任，由subscribe的参数来决定
  * 如何将发布者和观察者进行关联，也即何时调用subscribe



#### 2.1.2. 迭代器模式

迭代器是指能够遍历一个数据集合的对象，数据集合可以有多种表现形式，如数组、链表等，其作用是提供一个通用接口，使用者完全不必关心这个数据集合的具体实现方式。

迭代器的接口通常应该包含如下几个函数：

* `getCurrent`，  获取当前被游标指向的元素
* `moveToNext`，将游标移动到下一个元素
* `isDone`， 判断是否已经遍历完所有的元素





#### 2.1.3. Observable

每个`Observable`对象代表了在一段时间范围内发生的一系列事件，可以被表示为

```ts
Observable = Publisher + Iterator
```

可使用`new Observable`或者creation操作符(如`of`、`from`、`interval`等)创建Observables，传入参数仅有一个`subscribe`函数。



Observable的生命周期可分为如下几个步骤：

* 创建  `const observable = of(1,2,3)`
* 订阅 `const subscription  = observable.subscrible(next_cb, err_cb, complete_cb) `
* 执行，是一个惰性运算，只有在观察者订阅后才会执行。
* 清理  `subscription.unsubscribe()`


Observable 执行过程中可以发送三种类型的消息：

- `Next` notification: sends a value such as a Number, a String, an Object, etc.
- `Error` notification: sends a JavaScript Error or exception.
- `Complete` notification: does not send a value.

在一个 Observable的执行过程中，可以发送任意个`Next`通知，一旦发送`Error`或者`Complete`通知，则此后不能再发送任何通知。



```js
// Observable 的定义
class Observable {
  static create: Function
  static if: typeof iif
  static throw: typeof  throwError

  constructor(subscribe: Function){}
  subscribe(observerOrNext){return Subscription}

  pipe(...operations: OperatorFunction<any, any>[]): Observable<any>
  
}
```





#### 2.1.4. Observer

Observer  是 由 Observable 发送的值的消费者。观察者是一组回调函数的集合，每个回调函数对应一种 Observable 发送的通知类型。

```js
var observer = {
  next_cb: ()=>{},
  error_cb: ()=>{},
  complete_cb: ()=> {}
}
```



观察者是只有三个回调函数的对象，每个回调函数对应一种 Observable 发送的通知类型。

若要使用观察者，则需要把它传递给 Observable(可观察对象)的 `subscribe`方法

```js
observable.subscribe(observer)
```



#### 2.1.5. 如何使用Observable和Observer？

* 定义一个函数 `onSubscribe = observer => {}`，用于决定被观察对象的行为

* 使用`Observable`构造函数，传入上一步定义的 `onSubscriber`，实例化得到被观察对象，这时候， `onSubscribe`并没有被调⽤， 它只是在等待被观察对象的subscribe被调⽤。
* 定义观察者`observer`的值
* 调用被观察对象的subscribe方法，注册观察者，此时对应的`onSubscribe`函数就会被调用，`onSubscribe`函数的入参就是观察者，其内部就可以操作观察者。

```js
// 定义 onSubscribe 函数
const onSubscribe = observer => {
  observer.next(1)  // 此操作为给传入的observer的next回调传入参数1
}

// 新建一个被观察者对象
const observable = new Observable(onSubscribe)

// 定义观察者
const observer = {
  next: data => console.log(data)
}

// 注册，返回值即为正在执行的动作
const subscription = observable.subscribe(observer)

// 退订，此时就会取消执行
subscription.onsubscribe()

// 需要注意的是，如果使用create() 新建被观察对象，则必须在onSubscribe中返回一个`unsubscribe`函数
```





### 2.2 Hot observable 和 Cold observable

* Hot  observable :  表示只能获取注册之后被推送的值，类似于看直播
* Cold observable：表示还可以获取注册之前被推送的值，类似于视频点播





### 2.3 操作符简介

操作符是用来产生全新的Observable对象的函数。不过， 有的操作符是根据其他Observable对象产⽣返回的Observable对象， 有的操作符则是利⽤其他类型输⼊产⽣返回的Observable对象， 还有⼀些操作符不需要输⼊就可以凭空创造⼀个Observable对象。



### 2.4 弹珠图

* https://rxmarbles.com/ 可以看到主要的操作的弹珠图
* https://rxviz.com/ 可以写任何 observable 对象来查看弹珠图





## 3. 操作符基础

操作符都是函数，有两种形式的操作符：

* Pipeable Operators 是一个纯函数，可以输入一个` Observable$`，返回一个新的`Observable$`
* Creation Operators 作为另一种操作符，可以作为独立的函数用于创建一个新的`Observable$`，



### 3.1 Piping

将所有的pipe操作符都包在`pipe()`函数里面，每个pipe操作符以逗号分隔

```ts
obs.pipe(op1(),op2(),op3(),op4())
```

还有一个`pipe()`函数，与`.pipe()`类似，但是需要单独从`rxjs`引入。

```js
import { pipe } from 'rxjs'
import { filter, map } from 'rxjs/operators'

function getOddDoubled(){
    return pipe(
    	filter(v => v%2 !== 0),
        map(v => 2* v)
    )
}
```



### 3.2  Creation Operators

```ts
import { interval } from 'rxjs'
const obs = interval(1000)
```



### 3.3 Higher-order Observables(高阶Observables)

有时候需要处理`Observables of  Observables`，也即`HOO`，对于HOO需要将其转化为普通的`Observable$`

```ts
const fileObs = urlObs.pipe(
		map(url => http.get(url)),
  	concatAll()
	)
```

* `concatAll()`
* `mergeAll()`
* `switchAll()`
* `exhaust()` 



### 3.4 弹珠图

[rxviz.com](https://rxviz.com/examples/custom)



### 3.5 操作符的种类：

* 创建类(creation) ： 
  * ajax 、bindCallback、bindNodeCallback、defer、empty、from、fromEvent、fromEventPattern、generate、interval、of、range、throwError、timer、iif 等
  * `Join Creation Operators`
    * combineLatest、concat、forkJoin、merge、race、zip
* 转化类(transformation)：
  * buffer、concatMap、exhaust、map、mergeMap、switchMap、window 等
* 过滤类(filter)：
  * audit  、filter 、debounce 、skip、take、throttle 等
* 合并类
  * combineAll、concatAll、withLatestFrom 等
* 多播类
  * multicast、publish、share等
* 错误处理类
  * catchError、retry、retryWhen
* 辅助工具类
  * tap、delay、observeOn、timeout、toArray 等
* 条件分支类
  * default、every、find、isEmpty 等
* 数学和合计类
  * count、max、min、reduce 等





### 3.6 根据存在形式分类的操作符

* 静态操作符： 不需要被观察对象的实例即可以被调用
* 实例操作符：







## 4. Creation Operators

| 操作符                              | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| `ajax`                              | 根据ajax请求结果创建数据流，可带url、headers等对象用于配置或者直接使用url字符串 |
| `bindCallback`/  `bindNodeCallback` | 将一个回调的API转换为一个返回Observable的函数                |
| `defer`                             | 延迟产生数据流                                               |
| `empty`                             | 产生空的数据流                                               |
| `from`                              | 从数组等枚举类型数据产生数据流，参数可以是数组、类数组对象、Promise、可迭代的对象或类似于可观察对象的对象 |
| `fromEvent`/`fromEventPattern`      | 从外部事件对象产生数据流                                     |
| `generate`                          | 以循环方式产生同步数据流<br />`generate(initialState, condition, iterate, scheduler?)`<br />`generate(1,x=> x<10, x=> x+1.5)` |
| `interval`/`timer`                  | 间隔给定事件持续产生数据流                                   |
| `of`                                | 根据有限的数据产生同步的数据流，参数是列表形式               |
| `range`                             | 根据指定的数值范围产生数据流                                 |
| `throwError`                        | 产生错误通知                                                 |
| `iif`                               | 用于确定在subscription阶段哪个Observable会被真正的subscribed |
| `repeat`                            | 用于重复数据流，但是输入的数据流必须要有结束(complete)才可以重复 |





## 5. Join Operators



合并多个Observable的“最新数据”，选取操作符的原则：

* 完全独立的Observables，则使用`combineLatest`
* 若要把⼀个Observable对象“映射”成新的数据流， 同时要从其他Observable对象获取“最新数据”， 就使用`withLatestFrom`。  



高阶Observable：该Observable对象产生的数据仍为Observable对象

`concatAll`、`mergeAll`、`zipAll`、`combineAll`四个操作符，都是用于把一个高阶Observable内部的所有Observable组合起来

* `combineALL`是的`combineLatest`处理高阶Observable的版本



| 操作符                                            | 描述                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| `concat`和`concatAll`                             | 将多个数据流以首尾相接的形式合并，若前一个数据流没有完结，则无法输出后一个数据流的数据 |
| `merge`和`mergeAll`                               | 将多个数据流中的数据以先到先得的方式合并，若上游数据流无法完结，则`merge`输出的新数据流也不会完结，merge应用于合并产生异步数据的Observable对象，常用场景时合并DOM事件，此时就可以统一用一个事件处理函数来处理多个事件 |
| `zip`和`zipAll`                                   | 拉链式组合，将多个数据流中数据以一一对应方式合并。`zip`会把上游数据组合成数组形式，若其中有一个数据流完结，则`zip`输出的数据流完结。 |
| `conbineLatest`、`combineAll` 和 `withLatestFrom` | 持续合并多个数据流中最新产生的数据，当某一个上游数据流产生数据时，就会合并所有上游数据流中的最新数据，会有数据重复使用的情况，且只有当上游都产生数据时，才会往下游输出数据；当所有上游数据流完结时，`combineLatest`输出的数据流就完结了。<br />`combineLatest`可使用参数`project`，在将数据传递给下游之前利用`project`函数进行组合处理<br />`withLatestFrom`是仅在源Observables吐出数据时，才会组合最新的数据 |
| `race`                                            | 从多个数据流中选取第一个产生内容的数据流                     |
| `startWith`                                       | 在被订阅的数据流前面添加若干指定数据，添加的数据会被同步输出 |
| `forkJoin`                                        | 当所有Observable对象都完结， 确定不会有新的数据产生的时候， forkJoin就会把所有输⼊Observable对象产生的最后⼀个数据合并成给下游唯⼀的数据。类似`Promise.all` |
| `switch`和`exhaust`                               | 从高阶数据流中切换数据源，防止数据积压造成内存泄漏。<br />switch是指总是切换到最新产生的内部Observable对象中获取数据；<br />exhaust是“耗尽”，只有当前订阅的内部Observable对象已经完结，才会订阅新产生的，在当前Observable对象完结之前的新产生的Observable对象都会被忽略。<br />这两个操作符都需要上游且内部的Observable对象完结才可以完结。 |

```js
import { zip, interval, of,combineLatest,race,concat } from 'rxjs'
import { withLatestFrom, exhaust, startWith,concatAll } from 'rxjs/operators'

const source1$ = of(1, 2, 3);
const source2$ = of(4, 5, 6);
concat(source1$,source2$).subscribe(x => console.log(x));

// merge: 哪个上游数据吐出数据，下游数据就是哪个
const timerA = timer(200,1000).pipe(map(x => x+'A'));
const timerB = timer(500,1000).pipe(map(x => x+'b'));
merge(timerA, timerB).subscribe(x => console.log(x));

// withLastestFrom: 下游数据 = 源Observable 吐出的数据 + 其他input Observable的最新数据
const timera = interval(1000).pipe(map( x => 'interval'+x));
const timerb = timer(500,1000).pipe(map( x => 'timer'+x));
timerb.pipe(withLatestFrom(timera)).subscribe(x => console.log(x));

// exhaust: 一个高阶的Observable，内部会发出Observable，当前一个Observable还未完成会丢弃新发出的
const clicks = fromEvent(document, 'click');
const higherOrder = clicks.pipe(map((ev) => interval(1000).pipe(take(5))),);
higherOrder.pipe(exhaust()).subscribe(x => console.log(x));

// startWith: 先输出新增的数据流，再输出源observable的数据流
of("from source").pipe(startWith("first", "second")).subscribe(x => console.log(x));


// 输出 [a,'t1:0','t2:0'], [b,'t1:1','t2:1'],[c,'t1:2','t2:2'],[d,'t1:3','t2:3'],结束
zip(
  of('a', 'b','c','d'), 
  interval(1000).pipe(map(x=> `t1:${x}`)), 
  interval(500).pipe(map(x=> `t2:${x}`))
)

// 输出由project决定的字符串
combineLatest(  
  timer(500,2000).pipe(take(3)),
  of('a','b','c')
).pipe(
	map(([x,y])=> `timer1 outs: ${x}, of outs: ${y}`)
)

// 输出 '0 and 2', '1 and 5',...
interval(2000).pipe(
  withLatestFrom(interval(600)), 
  map(([x,y])=> `${x} and ${y}`)
)

// 输出 'a0', 'a1',...
race(
  timer(0,2000).pipe(map(x=> 'a'+x)),
  timer(200,1000).pipe(map(x=> 'b'+x))
)

```



## 6. Utility Operators

### 6.1 Math Operators

数学操作符都是实例操作符，且只有等上游数据完结时才会进行运算将唯一数据传递给下游。

| 操作符   | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| `count`  | 用于统计上游吐出的数据的个数                                 |
| `max`    | 取得上游吐出数据的最大值，由于吐出的数据可能是复杂类型(对象)，所以该操作符接受一个比较函数作为参数 |
| `min`    | 取得上游吐出数据的最小值                                     |
| `reduce` | 获取上游数据处理的累加值，同`Array.prototype.reduce()`，接受`accumulate`和`seed`两个参数 |

```js
import { of } from 'rxjs'
import { count } from 'rxjs/operators'

of([1,2,3]).pipe(count())
```



### 6.2  Conditional & Boolean Operators

| 操作符                | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `every`               | 判断上游吐出的每个数据是否满足传入的条件函数，满足则返回`true`，不满足返回`false` |
| `find` 和 `findIndex` | 找到上游数据中满足条件的第一个数据，不同在于`find`吐出数据，`findIndex`吐出序号，若没有满足条件的数据，`find`会吐出`undefined`，`findIndex`吐出`-1` |
| `isEmpty`             | 用于检查上游数据是不是空的                                   |
| `defaultIfEmpty`      | 当上游数据是空的时候，返回null或者自定义的值                 |

```js
import { of } from 'rxjs'
import { every, defaultIfEmpty } from 'rxjs/operators'

of([3,4,5])
  .pipe(every(x=> x>3))
  .subscribe(v => console.log(v))

of([]).pipe(isEmpty())

of([]).pipe(defaultIfEmpty('empty'))
```



### 6.3 根据官方文档的补充

| 操作符          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `tap`           | 拦截吐出的数据并执行指定的函数，只要没有发生错误，其输出与源一致，常用于debug，`tap`只会监视执行，但是不会触发执行 |
| `delay`         |                                                              |
| `delayWhen`     |                                                              |
| `dematerialize` |                                                              |
| `materialize`   |                                                              |
| `observeOn`     | 根据上游Observable对象生成一个新的Observable对象，让新的Observable对象吐出的数据由指定的Scheduler来控制。 |
| `subscribeOn`   | 控制订阅上游Observable的时机，此时就不一定是默认的同步订阅。 |
| `timeInterval`  | 吐出的数据是一个对象，包含有当前值以及上次吐出数据到这次吐出数据之间的时间间隔。`{value: 0, interval: 1000}` |
| `timestamp`     |                                                              |
| `timeout`       |                                                              |
| `timeoutWith`   |                                                              |
| `toArray`       |                                                              |

```js
import { interval } from 'rxjs'
import { timeInterval, map } from 'rxjs/operators'

// 输出 {value: 0, interval: 1002}, .....
interval(1000).pipe(
	timeInterval(),
  map(val => JSON.stringify(val))
)
```





## 7. Filter Operators

过滤类操作符主要用于判断给定的数据流中每个数据是否满足指定条件(判定函数)，满足则传递给下游，否则就抛弃。

数据过滤是进行回压控制的最简单的方式，通过丢弃一些数据来缓解压力，实现“有损回压”。

| 操作符                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `audit ` 和 `auditTime`                                      | 与 `throttle`类似，不同的是选择节流时间窗口内的最后一个数据传递给下游 |
| `debounce`                                                   | 去抖动。仅在`obs2`指定的延时时间之后，才会将`obs1`吐出的数据传递给下游。 |
| `debounceTime`                                               | 去抖动。上游传递给下游每个数据的时间间隔不可小于`deltaT`，若上游产生了一个数据，则需要等待`deltaT`，若都没有产生新数据，则将此数据传给下游，否则按照新数据产生的时间重新等待`delatT` |
| `distinct` 、`distinctUntilChanged` 、 `distinctUntilKeyChanged` | 不同。上游同样的数据仅在第一次产生时会被传递给下游。<br />`distinct`需要维护庞大的“唯一数据集合”，容易造成内存泄漏，所以可使用其第二个参数`flush`在适当时机清空此集合。 |
| `elementAt`                                                  | 将上游数据当成数组，输出指定下标的数据，当指定下标数据不存在时，可指定默认值 |
| `filter`                                                     | 实时对上游数据进行过滤输出给下游                             |
| `first`                                                      | 输出第一个数据或者满足条件的第一个数据给下游，相较于`find`或`findIndex`，`first`操作符可不传入条件，此时输出值即为上游的第一个数据 |
| `ignoreElements`                                             | 忽略上游所有数据，仅传递`completed`和`error`                 |
| `last`                                                       | 与`first`类似                                                |
| `sapmle` 和 `sampleTime`                                     | 采样。周期性的间隔内取上游的最新数据，丢弃其他数据。与`audit`不同的是，`audit`需要上游吐出一个数据后再开始周期性计时，而`sample`的开始规则与上游数据无关，且`sample`的参数不是返回一个`obs$`的函数，而是`obs$`，当`obs$`产生一个数据时，`sample`会从上游取已产生的最新数据。 |
| `single`                                                     | 用于检验上游是否只有一个数据满足指定条件，若是，则传递该数据，否则，传递“异常” |
| `skip` 、`skipLast` 、`skipUntil` 、`skipWhile`              | 跳过一些数据再取数据，分别为：从第N+1个数据开始取数据；丢弃最后的N个数据；... |
| `take`、`takeLast`、`takeUntil `、`takeWhile`                | 从上游拿数据，具体数量或者条件由参数指定。分别为：取前N个数据；取倒数N个数据；取数据直到满足另一个`obs$`吐出一个数据或者完结；满足条件则一直取数据 |
| `throttle`                                                   | 节流。`obs1`输出一个数据，该数据也作为`obs2`的输入数据，此时直到`obs2`输出第一个数据，会忽略掉`obs1`输出的所有数据。 |
| `throttleTime`                                               | 节流。传递上游吐出给下游的一个数据后，需要等待`delatT`再传递新吐出的数据 |



```js
import { fromEvent,interval,of,timer } from 'rxjs'
import { first, take, takeUntil, debounceTime, throttleTime } from 'rxjs/operators'

fromEvent(document,'click')
  .pipe(first(e => e.target.tagName === 'LI'))

// 输出前三个数据： 0,1,2
interval(1000).pipe(take(3))
// 在计时器到达4500ms之前，一直取interval输出的数据
interval(1000).pipe(takeUntil(timer(4500)))


//输出 0, 2,4
interval(1000).pipe(throttleTime(1999))
// 一直没有数据输出
interval(1000).pipe(debounceTime(1000))
// 输出 0， 2， 4, 6...
interval(1000).pipe(debounceTime(999))
// 输出 1,3,5,7...
interval(1000).pipe(auditTime(1900))

//输出0, 3, 6，
interval(500).pipe(throttle(()=>timer(1200)))
interval(500).pipe(
  filter(x => x%3==0),
  debounce(()=>timer(1200))
)

// 输出 7 1 2 3 4
of(7,1,2,2,3,7,2,4,3).pipe(delay(1000),distinct())

of(7,1,2).pipe(elementAt(3,'hello'))

// 输出异常，提示： sequence contains more than one element
interval(1000).pipe(single(x=> x%2 === 0))
```





## 8. Transform Operators

转化操作符用于改变数据流中的数据。

* 对每个数据做转化，上游数据和下游数据依然是“一对一”的关系，叫做“映射数据”。
* 对数据进行组合再传递给下游， 不改变数据本身，即“无损回压”

### 8.1 映射数据操作符

| 操作符  | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| `map`   | 调用方式为：`map(project: (val, idx)=>R, thisArg?)`<br />使用指定的参数(函数project)对上游数据进行处理，可使用`thisArg`参数指定执行函数`project`时`this`的值 |
| `mapTo` | 无论上游是什么数据，传递给下游的都是相同的数据               |
| `pluck` | 把上游数据中特定字段的值拔出来，即取出指定的嵌套属性对应的值，当发现某一属性不存在时，则会给下游传递 `undefined`，不会引起错误。 |

```js
import { of } from 'rxjs'
import { map } from 'rxjs/operators'

// 需要注意的是，若第一个参数`project`使用了箭头函数，则this会自动绑定上下文，指定的this则不生效。
of(1,2,3).pipe(
  map(function(x){return x*10+this.delta},{delta:4})
)

of(1,2,3).pipe(mapTo('A'))

fromEvent(document, 'click').pipe(
  pluck('target', 'tagName')
)

```



### 8.2 缓存窗口，无损回压控制

过滤类操作符通过丢弃一些数据实现有损回压控制，转换类操作符则使得无损回压控制成为可能，主要是将一段时间内的数据放在一个数据集合中，然后一次性传递给下游，数据集合的形式分为数组、Observable对象两种。

* 数组形式的数据集合：以buffer开头的操作符，将上游数据放在数组中传递给下游
* Observable对象形式的数据集合：以window开头的操作符，将上游数据放在`Observable`中传递给下游

 

高阶Map： 包含有 `concatMap`等，此类操作符可以传递一个返回`Observable`的`project`函数，然后会把`map`操作产生的高阶`Observable`利用对应的组合操作符合并为一阶的`Observable`对象。



高阶MapTo：包含有`(concat|merge|switch)MapTo`等。其参数是一个`Observable`对象(注意：不是可以返回Observable的对象的函数)



数据分组：用于将一个数据流拆分为多个数据流，主要有`groupBy`和`partition`两个操作符。



由于某些时候，我们还需要获取当前输入和前一个时刻输入综合所得到的结果，RxJS提供了可以累计所有上游数据的操作符，例如 `scan` 和 `mergeScan` 操作符



| 操作符                                       | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| `bufferTime` / `windowTime`                  | 根据时间缓存上游的数据，使用参数来指定产生缓冲窗口的间隔，<br />`bufferTime(arg1, arg2, arg3)`，<br />其中，arg1 指时间窗口大小，arg2是指创建时间窗口的间隔，arg3是指缓存的数据最大数量 |
| `bufferCount` / `windowCount`                | 根据数据个数划分，此时第一个参数为每个时间窗口需要缓存的上游数据的个数 |
| `bufferWhen` / `windowWhen`                  | ⽤Observable对象来控制Observable对象的生成，此时参数为返回Observable的函数`closingSel` |
| `bufferToggle` / `windowToggle`              | 利⽤Observable来控制缓冲窗口的开和关 。<br />第一个参数`open$`产生一个数据表示一个缓冲窗口的开始， <br />第二个参数`closingSel`也会同时被调用（其入参为`open$`的输出）， 用于获取缓冲窗口结束的通知。 |
| `buffer` / `window`                          | 仅有一个参数`notifier$，当其产生一个数据时，表示是前一个缓存窗口的结束，也是后一个缓存窗口的开始。 |
| `concatMap`                                  | `concatMap`适合处理需要顺序连接不同Observable对象中数据的操作，如网页DOM的拖曳操作。 |
| `mergeMap`                                   | `mergeMap`是`map`和`mergeAll`的组合，mergeMap对于每个内部Observable对象直接合并，也就是任何内部Observable对象中的数据， 来⼀个给下游传⼀个， 不做任何等待。特别适合进行异步处理的操作，如点击发起ajax请求收到结果再处理的流程 |
| `switchMap`                                  | `switchMap`的不同之处在于，后产生的`inner Observable对象`的优先级总是更高，会立刻退订之前的`inner Observable对象`，改为从最新的`inner Observable对象`取数据。该特点适用于总是想要获取最新的ajax请求的结果 |
| `exhaustMap`                                 | `exhaustMap`与`switchMap`正好相反，先产生的`inner observable`的优先级总是更高，后产生的`inner Observable对象`被利用的唯一机会是在之前的`inner Observable对象`已经完结之后。可用于“使用ajax建立长连接”的场景。 |
| `concatMapTo` / `mergeMapTo` / `switchMapTo` |                                                              |
| `expand`                                     | 类似于`mergeMap`，但是其传给下游的数据，也会传递给自己       |
| `groupBy`                                    | 输出一个`高阶Observable`，对于上游推送下来的数据，检测其key值，若是第一次出现，则产生一个新的`inner Observable对象`，此数据为该对象的第一个数据，若已出现，则直接将其塞给对应的对象，其key值由第一个参数`keySelector`来决定。 |
| `partition`                                  | `partition`接收一个判定函数作为参数，对上游的每个数据进行判定，满足条件的放入一个`inner Observable对象`，不满足的放到另一个，如此进行分组，仅分两组。返回的是一个包含两个元素的数组，第一个元素是满足条件的`Observable对象`，第二个是不满足的。一般使用partition后不会进行链式调用，会使用变量存储对应的`Observable对象`， |
| `scan`                                       | 类似`reduce`操作符，可使用一个规约函数参数及一个可选的seed种子参数，scan 对于每一个上游数据都会产生一个规约结果；而reduce是对上游所有数据进行规约，总共产生一个规约结果传递给下游，上游数据不完结，reduce 也无法输出结果给下游。<br />当实际应用需要维护状态时，可选用scan操作符。 |
| `mergeScan`                                  | 类似于`scan`，但是其规约函数返回的是一个`Observable对象`而不是简单的一个数据；可使用的应用场景如前一个ajax的请求结果作为后一个ajax的参数，类似下拉加载发送新的请求这类情况。 |



```js
import { timer } from 'rxjs'
import { bufferToggle, mergeMap } from 'rxjs/operators'

// 输出  [0,1], [3], [5]...
timer(0,500).pipe(
  bufferToggle(timer(0,1001), ()=>timer(600))
)

// 输出 []， [0,1,2,3], [4,5,6,7],...
timer(0,500).pipe(buffer(timer(0,2000)))

// 连续输出 0,1,2，0,1,2,0,1,2
of(1,2,3).pipe(
  concatMap(
    x => interval(1000).pipe(take(3))
  )
)

// 输出为 [0], [1], [2,0], [3,1], [4,2],...
interval(1000).pipe(
	take(2),
  mergeMap(()=>interval(500))
)


// 输出为 1, 4, 9, 16,....
interval(1000).pipe(map(x=>2*x+1), scan((acc, cur)=>acc+cur))
```





## 9. Error Handle Operators

在异步处理中，try/catch 方式派不上用场，使用回调函数或者Promise的方式来处理异常虽然可以解决问题但是依然存在诸多缺点。



* 在浏览器环境中，若没有捕获js程序的异常，那么对应的事件处理就会中断，之后的指令不再执行
* 在Node.js环境中，若没有捕获js程序的异常，那么默认情况下会让Node.js应用崩溃退出。



JavaScript 自带的 try/catch 比较麻烦，发生错误时，JavaScript运行环境会中止当前指令，创建⼀个指向产⽣错误指令的栈跟踪信息（Stack Trace） ， 包含错误信息、 代码⾏号和代码所在⽂件名，把这些信息封在Error对象中， 然后沿着执⾏栈⼀层⼀层往上找catch区块，如果找不到， 那就只能交给JavaScript运⾏环境⽤默认⽅法处理。

可以发现，try/catch 比较适用于处理同步操作发生的错误，对于异步，则完全无用武之地了。



在JavaScript中异步操作都需要使⽤回调函数， 要传递异步操作中产⽣的错误异常， ⼀般就要借助于回调函数的参数，但是这种做法容易引起回调地狱。



使用Promise可以解决回调地狱的问题，但是Promise有其自身缺点，一是不能重试，而是并不强制要求异常被捕获。



在函数式编程中，每个函数都应该是纯函数，也就是毫无副作用的函数，对于输入返回输出结果；若一个函数可以抛出异常，那么该函数则不再是纯函数，因为 throw 相当于增加了一个函数出口，以此可能会改变外部状态，违背了函数式变成的要求。



在RxJS中，错误异常和数据一样，会沿着数据流管道从上游向下游流动，最后触发Observer的error方法，但是若发生的一些异常如逻辑错误，应该在管道中进行处理而不是一股脑抛给Observer。

处理错误的方式有两种：

* 恢复(recover)：本来产生了错误异常，但是依然让运算继续进行，如使用默认值取代错误的返回值继续执行
* 重试(retry)：重新尝试之前发生错误的操作。



###   9.1 `catchError` 操作符

该操作符有一个`selector`参数，该参数是一个包含有`err`(包含有错误详情)以及`caught$`(源`obs$`，可用于重试)的函数，返回值是一个可用于继续数据流的`obs$`



```js
// catchError 的表达式, 使用 Obs$ 吐出的数据继续数据流
catchError((err, caught$)=> Obs$)

// 1,2,3, 'replace 4'
of(1,2,3,4,5).pipe(
	delay(2000),
  map(n => {
    if(n===4){
      throw "it's four"
    }
    return n
  }),
  catchError(err => of('replace 4') )
)
```

`catchError`操作符主要实现了**恢复**的功能，主要是往数据流管道里面塞入对应的数据，让数据流得以继续。但是塞入的数据不一定是期望值，若重试就可能得到期望结果，还是需要使用`重试`。

### 

### 9.2 `retry` 和 `retryWhen`操作符

`retry`会进行限定次数的重试操作

```js
// 参数为 count，默认值为-1
retry(count: -1)
```

`retry`的问题在于若上游传递一个错误，则会立刻开始重试，此时可能还是拿不到想要的结果，这种需要延时重试的操作可以使用`retryWhen 操作符`



`retryWhen`接受⼀个函数作为参数， 这个参数称为`notifer`， ⽤于控制**重试**的节奏和次数， 这⽐`retry`单纯只能控制重试次数要前进⼀步。

```js
retryWhen(errs$=> obs$)
```

可以使用`retryWhen`和`scan`操作符，进行统计，实现`retry`的功能



### 9.3 `finalize`操作符



```js
finalize(callback)
```

`finalize`操作符的参数是一个回调函数，当其所映射的`obs$`的状态是`complete`或者`error`时,会执行这个回调函数。





## 10. 多播 multicast

多播就是让一个数据流的内容被多个Observer订阅。

为了实现多播，RxJS引入了特殊的类型`Subject`，对该类型的扩充形态包含有

* `BehaviorSubject`、
* `ReplaySubject`、
* `AsyncSubject`

同时还有一系列支持多播的操作符：

* `multicast`
* `publishLast`
* `publishReplay`
* `publishBehavior`



RxJS 中，Observable 和 Observer 是发布内容和接收内容的关系，发布内容的形式可以分为三种：

* 单播，一对一
* 广播，一对所有，如`Node.js`中的`EventEmitter`
* 多播，一对指定，RxJS支持一个Observable被多次subscribe



Cold Observable 和 Hot Observable 

* Cold Observable 是每次被subscribe都会产生一个全新的数据序列的数据流
* Hot Observable的内容创建与observer无关
* 两者都是有被订阅才会执行，Cold Observable 在没有订阅者的情况下不会产生数据，而Hot Observable在没有订阅者时仍然会产生数据，但是数据不会传入管道。
* 使用RxJS 的`Subject`可以将Cold Observable 改为 Hot Observable

RxJS中的创建类操作符，大多数都是生成的Cold Observable，如`interval、range`等，一些操作符的数据源是在外部，如`fromEvent、fromPromise、fromEventPattern`等生成的则是Hot Observable



### 10.1 Subject

函数式编程需要保持不可变性，对于Observable对象的转化，需要进行封装，生成一个新对象，然后下游订阅新生成的对象。此时中间的转化者需要有两个功能：

* 提供subscribe方法，让其他人可以订阅自己的数据源，相当于一个Observable
* 能够接受推送的数据，包括Cold Observable推送的数据，相当于一个Observer

`Subject`类型就具有上述两个接口。



`Subject`不是操作符，所以无法实现链式调用，可以自己定义一个makeHot的操作符用于实现链式调用的功能

```js
// rxjs v5 版本，自定义makeHot 操作符
Observable.prototype.makeHot = function () {
  const cold$ = this;
  const subject = new Subject();
  cold$.subscribe(subject);
  return Observable.create((observer) => subject.subscribe(observer));
}
```



Subject  拥有自己的状态，所以一旦一个Subject对象被调用了complete 或者 error 函数，则其作为Observable的生命周期就结束了，此时若还调用其next方法传递参数给下游，是不会生效的。



### 10.2 Subject 处理多个上游

理论上Subject 可以订阅多个上游数据流，然后进行合并传递给下游，但是如果上游Observable传递一个 complete  信号，则 Subject 也会完结，无法继续往下游传递数据，此时若想合并上游数据，最好使用`merge`操作符(数据的合并顺序为fifo)。

对于Subject的错误处理，如果Subject产生一个错误异常并没有被下游的Observer处理，则此Subject的其他下游Observer都会失败。此时就需要Subject的每个下游Observer都具有处理错误的能力。



### 10.3 支持多播的操作符

多播操作符，主要是将 Cold Observable 转化为 Hot Observable？ 保证所有订阅者的一致性？？？

| 操作符      | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| `multicast` | `multicast(subjectOrSubjectFactory, selector?)`              |
| `publish`   | 是经由`multicast`封装实现，用户只需选择是否传递selector参数。 |
| `share`     | 返回一个调用了refCount的 ConnectedObservable对象，可以根据Observer的数量自动完成对上游的订阅和取消订阅 ，语法类似为：`Observable.prototype.share= ()=> this.multicast(()=>new Subject()).refCount()` |

#### 10.3.1 multicast

上游的 Cold Observable 一旦被subscribe，就会开始推送数据，此时如果multicast产生的Hot Observable 没有注册Observer，则推送的那些数据都会被丢弃。

还有一种应用场景是，当下游Observer都退订Subject后，需要Subject 也可以实现退订上游数据，由于Subject可以有多个Observer，所以需要对订阅Subject数据的Observer的数量进行跟踪，当Observer数量大于0时Subject自动订阅上游，为0时自动退订，如此即为 refCount的功能。

如果要使用refCount的功能，需要multicast的第一个参数是返回Subject实例的函数，而不是Subject实例本身。

Multicast 如果使用了`selector`，则指定了multicast返回的Observable对象的生成方法，如此生成的就不是ConnectedObservable类型，也就没有对应的connect 和 refCount 方法。`selector`可以使用上游数据任意多次，但是不会重复订阅上游数据流。





### 10.4 高级多播

| 操作符            | 对应的Subject子类 | 描述                                                         |
| ----------------- | ----------------- | ------------------------------------------------------------ |
| `publishLast`     | `AsyncSubject`    | 多播的内容是上游最后一个数据，当上游完结时，会将最后一个数据传递给Observer |
| `publishReplay`   | `ReplaySubject`   | 类似于录像，支持“多播”和“录播”，可以把上游传递的数据储存下来，当添加新的Observer时，可以接收到订阅之前产生的数据。使用此操作符时不带参数容易引发内存问题，建议指定合理的参数以限制缓存大小。 |
| `publishBehavior` | `BehaviorSubject` | 当添加Observer时，即使上游没有吐出数据Observer也可以收到这个默认数据，默认数据总是被上游吐出的最新数据替代。 |

replay 是指把Observable对象吐出的数据重新走一遍，而不是让数据管道重新产生一遍数据。



`BehaviorSubject`是一种Subject，它需要一个初始值，它会存储发送给消费者的最新的值，任何一个新的Observer订阅它时，Observer会立马收到来自`BehaviorSubject`的`current value`。





### 10.5 四种 Subject的对比

| subject类型       | 是否存储数据                 | 是否需要初始值 | 何时向订阅者发布数据     |
| ----------------- | ---------------------------- | -------------- | ------------------------ |
| `Subject`         | 否                           | 否             | 有新数据就发布           |
| `BehaviorSubject` | 是，存储初值或者最后一条数据 | 是             | 有新数据就发布           |
| `ReplaySubject`   | 是，存储所有数据             | 否             | 有新数据就发布           |
| `AsyncSubject`    | 是，存储最后一条数据         | 是             | 数据源complete时才会发布 |





## 11. 掌握时间的Scheduler

### 11.1 Scheduler的调度作用

可以将Scheduler理解为调度器，用于控制RxJS数据流中数据消息的推送节奏。

使用操作符的时候不传递scheduler参数，则代表使用默认的scheduler实现操作符的功能。

* Scheduler是一种数据结构，可以根据任务优先级或者其他条件安排任务执行队列
* Scheduler是一个执行环境，可以指定一个任务何时何地执行
* Scheduler拥有一个虚拟时钟， TestScheduler可以伪造时钟，方便对RxJS数据流进行单元测试



| Scheduler                 | Purpose                                                      |
| ------------------------- | ------------------------------------------------------------ |
| `null`                    | 表示同步执行的`scheduler`                                    |
| `queueScheduler`          | 利用队列实现的`scheduler`，用于迭代一个大集合的场景          |
| `asapScheduler`           | 尽快执行的`scheduler`                                        |
| `asyncScheduler`          | 使用`setInterval`实现的 `scheduler`，用于基于时间吐出数据的场景 |
| `animationFrameScheduler` | 用于动画场景的`scheduler`                                    |



JavaScript 是一个单线程语言，也即可以理解为永远仅有一个线程来运行所有的代码。

某些浏览器有Web Worker的功能，一个Web Worker相当于一个新的线程，在具有Web Worker的浏览器中，可以把JavaScript的计算量比较高的工作分配给Web Worker 去执行，注意，Web Worker中是无法直接操作DOM的。



#### 1) JavaScript 如何在只有一个线程的情况下处理异步和并发任务？

JavaScript 的解析和运行环境叫做“JavaScript引擎”，JS引擎有很多实现，如Chrome浏览器和Node.js使用的是`v8`，Edge浏览器使用的是`Chakra`，Firefox使用过`Gecko`等多种引擎，这些引擎无一例外都要实现以下两个功能：

* `调用栈(Call Stack)`和`事件循环(Event Loop)`

调用栈是所有编程语⾔都存在的概念， 当调⽤⼀个函数的时候， 就在调⽤栈上创建这个函数运⾏的空间， 参数的传递、 局部变量的创建都是通过调⽤栈完成； 当⼀个函数执⾏完毕的时候， 对应调⽤栈上这个函数的本次运⾏空间就被清除。

调⽤栈的执⾏⽅式很适合简单的数据运算，JavaScript需要对多种不同的事件进⾏响应， 所以， 就有了“事件循环”这个概念。

为了⽀持事件处理， 需要有⼀个“事件队列”， 存储所有等待处理的事件， 这些事件有很多来源，使⽤setTimeout和setInterval、 调⽤AJAX功能的API、 使⽤Promise都可以给“事件队列”添加待处理事件。

“事件循环”可以看作⼀个死循环， 重复的⼯作就是从“事件队列”中拿到需要处理的事件任务， 然后把这个任务交给调⽤栈去执⾏， 当这个任务处理结束之后， 再从“事件队列”中拿下⼀个任务塞给调⽤栈……如此周⽽复始， 永不停歇。

因为JavaScript是单线程执⾏， 所以当调⽤栈正在执⾏⼀个任务的时候， 事件循环也只能等着， 只有当前⼀个务完成之后， 才能塞给调⽤栈下⼀个任务。

在JavaScript编程中， 要特别注意的是让每个任务不要耗时太长， 要知道， 不光JavaScript的任务要在唯⼀的线程上执⾏， 浏览器窗口渲染HTML内容、 响应⽤户输⼊也要⽤上这个唯⼀的线程， 如果单个任务耗时太长， 必定造成卡顿， 影响⽤户体验。

**事件队列**中的任务又分为以下两种：

* `Micro Task` ：不同的`JS引擎`有不同的定义
  * `Node.js`：`process.nextTick`
*  `Macro Task`： 如`setTimeout`、`setInterval`、`AJAX`请求，

当调用栈处理完一个任务后，事件循环会先看` Micro Task`队列是否有任务，如果有，则会先处理所有的`Micro Task`，

需要注意的是`Promise`在不同的JS引擎中，可能会被当做`Micro Task` ，也可能被当做 `Macro Task`。



#### 2） Scheduler 的工作方式？

在RxJS中，若不特意指定 Scheduler，都是采用调用栈来完成的。对于这种情况，如果产生的数据量很大，则会占用唯一的JS线程很久，引起页面卡顿。对于用户的感知则会造成很不好的影响。

* 在RxJS中， `asap`会尽量利⽤`Micro Task`， 也就是在异步的情况下， 尽量早地安排下⼀个数据的产⽣， 这也就是为什么它叫asap（as soon as possible） 。
* `async` 则会利用 `Macro Task`
* `queue`在调用时若`scheduler`的`delay`参数为0，则以同步的方式调用，若`delay`参数大于0，则其表现形式与`async`一致。

实际开发时，只要合理的应用RxJS提供的现有的Scheduler即可，也即使用RxJS提供的操作符。



#### 3） Scheduler 操作符

对于普通的创建或组合类操作符，Scheduler只是一个可选参数，不采用此参数操作符也可以正常工作，采用此参数则会让吐出数据的节奏发生变化。

还有一类应用Scheduler的操作符，对于这类操作符必须使用Scheduler实例作为参数。

| 使用scheduler的操作符 | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `observeOn`           | 根据上游Observable对象生成一个新的Observable对象，让新的Observable对象吐出的数据由指定的Scheduler来控制。 |
| `subscribeOn`         | 控制订阅上游Observable的时机，此时就不一定是默认的同步订阅。 |



上述两个操作符最好在管道末尾，subscribe之前使用。





## 12. RxJS的调试和测试

特定于应⽤了RxJS的代码， 调试和测试都是围绕Observable数据流展开， 因为使⽤RxJS的状态管理实际上就是数据流管理。

### 12.1) 调试方法

现代浏览器自带的 Debugger 对于 RxJS 都无用武之地，原因在于 RxJS的异步操作是无法通过 Step In 这些操作来跟踪的，可以通过比较传统的`打log`来进行调试。

在对数据管道进行调试时，可以使用`do`(RxJS v6中使用`tap`)操作符，`do`操作符不会改变流经数据管道的任何数据，数据只不过经它的手走一遍而已，对于上游和下游都感知不到`do`的存在。

使用`do`操作符观察数据会增加冗余代码，可以进行改进：

```js
// 定义 debug 操作符, 使用全局变量来控制
Observable.prototype.debug = function(fn){
  if(global.debug){
    return this.do(fn)
  }else{
		return this
  }
}
```



更进一步，可以使用枚举对日志进行分级：`debug`、`info`、`warn`、`error`。

有时候为了发现程序中的问题，可以分析程序，画出数据流依赖图，分析数据的流转情况，如此也可以排查。

对于比较复杂的情况，数据流依赖图也无法简化，就可以选择使用弹珠图来分析。



### 12.2) 单元测试

JavaScript 是动态语言。

利用lint、TypeScript 、flow等工具可以对JS进行一些静态代码检查。

RxJS贯彻的是函数式编程的思想，对于函数式编程而言，单元测试变得十分容易。

#### 1). Mocha

`Mocha`是最常用的一个单元测试框架。

任何测试都有的两个概念：

* 测试套件(Test Suite)
* 测试用例(Test Case)

测试⽤例是对⼀个功能点的测试过程， ⽽测试套件则包含多个测试⽤例， ⽤于把相关测试⽤例组织到⼀起， ⽽且， 测试套件还可以包含其他的测试套件， ⽤这种⽅式，所有的单元测试可以被组织成⼀个树形的结构。

在Mocha中， 通常⽤describe来描述⼀个测试套件， ⽤it来描述⼀个测试⽤例。

```js
// 示例
describe('sample suite', () => {    // 最外层是⼀个describe函数调⽤， 创建⼀个测试套件
  it('should work', () => {           // it函数的作⽤是创建测试⽤例， 
  });
  it('should work as well', () => {
  });
  context('another suite', () => {   // context只不过是describe的⼀个别名⽽已，功能一致
    it('should do something', () => {
    });
  });
});
```



测试驱动开发(TDD)的原则是，先写测试用例，再实现功能，如此测试用例不会受功能实现细节的影响。



#### 2) Chai

`Chai`是一个具有判定功能的库，可使用其`assert`功能集做判定



#### 3) RxJS异步操作的单元测试

Mocha默认要求⼀个单元测试要在2秒钟之内结束， 不然， 这个单元测试就超时出错。

在Mocha中， 可以通过this.timeout函数修改某个测试⽤例的超时时间。

Mocha还是提供了异步操作的测试⽅法， 如果代表测试过程的函数增加⼀个参数， 那么Mocha认为这是⼀个包含异步操作的单元测试，显示调用这个传入的函数参数，可以告知Mocha用例执行结束。此外，也可以在测试函数中返回一个 Promise对象，当该Promise对象成功结束时，才会认为这个单元测试结束。



**TestScheduler**

TestScheduler 带来一种全新的测试RxJS数据流方式，称为弹珠测试(Marble Test)，利用字符串代表弹珠图，让测试用例中的数据流十分清晰。

TestScheduler是不能重⽤的， 每个测试⽤例的运⾏都需要重新创建⼀个Test-Scheduler实例，因此在⽂件模块的位置定义了⼀个scheduler变量， 这个变量在每个测试⽤例执⾏之前都会被赋值⼀个全新的TestScheduler实例。 

```js
import Rx, {TestScheduler} from 'rxjs';
import {assert} from 'chai';let scheduler;
describe('Observable', () => {
  // beforeEach是Mocha提供的⼀个函数， 作⽤是让所处的测试套件中所有测试⽤例执⾏之前执⾏⼀个函数
  beforeEach(() => { 
    // deepEqual 需要this是 assert时才可正常工作，所以需要bind进行绑定
  	scheduler = new TestScheduler(assert.deepEqual.bind(assert));
  });
  it('should parse marble diagrams', () => {
    // 用字符串表达的弹珠图可以被TestScehduler实例用于创建Observable对象
    const source = '--a--b--|';
    const expected = '--a--b--|';
    const source$ = scheduler.createColdObservable(source);
    scheduler.expectObservable(source$).toBe(expected);
    scheduler.flush();
  });
});
```



#### 4) 可测试的代码

如何写出可测试的代码？

*  逻辑拆分：生产者(数据源)、数据管道、观察者

编写⾼可测试性代码的⼀个⽅法， 就是把逻辑尽量放到数据管道之中， 把⼀些难以单元测试的逻辑推到边缘部分， 让数据源和Observer的代码尽量简单， 简单到⽆需单元测试， 这样，我们就可以对真正的业务逻辑部分进⾏全⾯的单元测试。



如果数据管道中也涉及到输入输出怎么办呢？

> 例如：一个input搜索框，及对应列表显示搜索的推荐信息

```js
// 1. 数据源，检测KeyUp，因为onChange事件是在失去焦点的时候才会触发？
const createKeyUp$ = () => {
  return fromEvent(document.querySelector('#search-input'), 'keyup')
}

// 使用 debounceTime 控制只要有输入，就延迟发送请求
// 使用 pluck 提取 input的的value字段
// 使用 map 去除输入的字符串首尾的空格
// 使用 filter 过滤内容为空的数据
// 在数据流中做输入输出操作使用 mergeMap 或者 switchMap, 此处只取最新的请求，使用 switchMap
const searchRepo$ = src$ => {
  return src$.pipe(
  	debounceTime(150),
    pluck('target','value'),
    map(text => text.trim()),
    filter(query => query.length !== 0),
    switchMap(query => {
      const url=`http://api.github.com/search/repositories?q=${query}`
      return fromPromise(fetchAPI(url))
    })
  )
}


const fetchAPI = (url) => {
  return fetch(url)
  .then(response => {
    if (response.status !== 200) {
      throw new Error(`Invalid status for ${url}`);
    }
    return response.json();
  })
  .then(json => {
    return {
      total_count: json.total_count,
      items: json.items.map(x => ({
        name: x.name,
        full_name: x.full_name,
      }))
    };
  });
}

const observer = {
  next: (value) => {
  	document.querySelector('#results').innerHTML = 
      value.items.map(repo => `<li>${repo.full_name}</li>`
  }
};

```

上述实现的 searchRepo$ 可测试性还是不强，可以抽离为如下的形式：

```js
const searchRepo$ = (key$, fetch$, dueTime, scheduler) => {
  return key$.pipe(
  	debounceTime(dueTime, scheduler),
  	pluck('target', 'value'),
    map(text => text.trim()),
    filter(query => query.length !== 0),
  	switchMap(fetch$)
  )
};
```



对于数据管道部分的代码可以进行合理的分割，让单元测试无需考虑外部资源的模拟。



## 13. RxJS 驱动 React

React 中一切皆为组件，其界面展示由数据驱动，是声明式编程。

* 开发者只需要“声明”功能是怎样就可以， ⽽不纠结于“怎样去做”， 这种编程思维就叫“声明式编程”，比如React 组件接收props，根据props中的状态字变更渲染的状态。

RxJS 是数据流管理工具，并不支持界面渲染，所以可以使用RxJS来管理数据状态，使用React渲染界面



**一直不理解的一点，使用 next 传递数据，是Observable的功能，还是observer的功能？？**

* 是Observable的功能，observer 只是提供对应的回调





## 14. RxJS 和 Redux 结合

### 14.1 RxJS 实现 Redux

Redux 进行数据管理包含以下几个要点：

* store: 用于存储数据，使用createStore 生成
* reducer: 用于接收 action，更新store中的state
* action: 是一个对象，一般包含type和payload 两个部分，type是要做的操作类型，payload是数据



```js
// 极简实现示例
action$.pipe(scan(reducer, initialState)).subscribe(renderView);
```

* `action$`是⼀个Observable对象， ⾥⾯的数据就是action对象



使用`RxJS`实现`redux`，主要是实现`createStore`的功能，调用`createStore`需要返回⼀个对象， 这个对象具备三个函数dispatch、 getState和subscribe。





### 14.2  Redux 和 RxJS 的结合版 Redux-Observable

Redux-Observable 中有一个核心概念叫做 `epic`，它是一个纯函数，与reducer类似，接受两个参数，返回一个结果，其参数分别为：`Observable对象`和redux的`store对象`，返回的也是`Observable对象`，

```js
// 基于RxJS v6 的 redux-observable 返回值也是一个 Observable
function (action$: Observable<Action>, state$: StateObservable<State>):
Observable<Action>;
```

* `action$`是由派送给Store的所有action对象组成的Observable对象
* Redux会订阅epic函数返回的Observable对象， 并且对产⽣的所有action对象做dispatch的动作， 因此⼀般没必要让epic函数直接调⽤dispatch
* 每dispatch⼀个action对象都会调⽤reducer， 不过， Redux只会在启动的时候调⽤epic⼀次， 之后只会持续给参数`action$`推送action对象， 然后把epic返回的Observable对象中每个数据做dispatch操作。 当⼀个action对象被送到`action$`中， 到底是⽴刻让epic返回的Observable对象产⽣⼀个新的action对象， 还是延时⼀段时间再产⽣action对象， 亦或是调⽤AJAX请求获得结果之后再产⽣action对象， 完全看epic函数如何实现， 正因为如此，epic给予了Redux中数据管理最⼤的灵活性
* epic应该要对action$进⾏筛选， 只对满⾜条件的action才产⽣新的action对象。









