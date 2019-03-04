# JS 模块化


## CommonJS
##### 1. 定义模块
根据CommonJS规范，一个单独的文件就是一个模块，每个模块都是一个单独的作用域，也即，在该模块内部定义的变量，无法被其他模块读取，除非定义为global对象的属性。

##### 2. 模块输出
模块仅有一个出口，也即`module.exports`对象，需要把模块希望输出的内容放入该对象

##### 3. 加载模块
使用`require`方法加载模块，该方法读取一个文件并执行，返回文件内部的`module.exports`对象

##### 4.示例
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

## 痛点
从上述示例可以看出，`require`方法加载模块是同步加载，从浏览器端，脚本标签都是异步加载的，这样的话，传统的CommonJS模块在浏览器环境中无法正常加载。
