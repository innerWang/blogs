# JS 模块化


## CommonJS
##### 1. 定义模块
根据CommonJS规范，一个单独的文件就是一个模块，每个模块都是一个单独的作用域，也即，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。

##### 2. 模块输出
模块仅有一个出口，也即`module.exports`对象，需要把模块希望输出的内容放入该对象

##### 3. 加载模块
使用`require`方法加载模块，该方法读取一个文件并执行，返回文件内部的`module.exports`对象

<details>
	<summary>代码示例</summary>

```js
 //模块定义 myModel.js
 var age = 10;
 function  getAge(){
   console.log(age);
   return age;
 }

 function setAge(newAge){
   age = newAge
   console.log(age);
 }

 module.exports = {
   getAge: getAge,
   setAge: setAge
 }


 //从 App.js加载模块
 var ageModule = require('./myModule.js');
 ageModule.getAge();

```
</details>



##### 痛点
从上述示例可以看出，`require`方法加载模块是同步加载， 从浏览器端，脚本标签都是异步加载的， 这样的话， 传统的 CommonJS 模块在浏览器环境中无法正常加载。

##### CommonJS 与 ES6 模块的区别
* CommonJS 模块的输出时一个值的拷贝，ES6模块输出的是值的引用
* CommonJS 模块是运行时加载，ES6模块是编译时输出接口。

<a href="http://es6.ruanyifeng.com/#docs/module-loader#ES6-%E6%A8%A1%E5%9D%97%E4%B8%8E-CommonJS-%E6%A8%A1%E5%9D%97%E7%9A%84%E5%B7%AE%E5%BC%82">阮一峰 ES6模块与 CommonJS模块的差异</a>











