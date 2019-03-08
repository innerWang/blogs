# Scss 入门纪要

## <a name="catalog">目录</a>

##### 1. [常见语法功能](#One)


<hr>

Sass的官方解释器是开源的且用Ruby语言写成，起源是Ruby社区的框架 Rails 是一个全栈Web框架，Ruby的工程师觉得 CSS难用，就发明了 Sass(a代表 awesome)，去掉了括号并采用缩进式语法，难以为大多用户接受，后续就开发了新语法 SCSS， 使用类似CSS一样的块语法， 使用大括号区分不同的规则，使用分号区分具体的样式。

初学SCSS语法使用webpack去编译打包会比较复杂，此处推荐两种比较简单的方式。
* 本地开发使用parcel
```c
  npm init -y    //添加并初始化package.json
  npm install --save-dev parcel    //安装parcel
  npx parcel index.html    //若本地未安装node-sass,则会先自动安装,后在浏览器打开提示的链接，即可看到对应的html
```

* 使用各种在线环境
	* <a href="https://codepen.io">codepen.io</a>  
	* <a href="https://jsbin.com">jsbin.com</a> 


## <a name="One">1. 常见语法功能</a>

#### 1.1 嵌套选择器
  * 使用 “&” 引用父选择器

#### 1.2 变量
  * 定义变量并赋值 `$border-color: red;` ,可通过 `border: 1px solid $border-color` 引用;

#### 1.3 mixin(混入)
使用`@mixin`指令定义一个代码块，在它后面跟混入的名称及任选的参数，以及混入的内容块。类似js的函数定义。
```css
 @mixin debug($border-color:red){
   border: 1px solid $border-color;
 }
```
使用时则通过 `@include` 指令引用。

```css
  ul {
    @include debug;
    padding: 10px;

    >li {
      list-style: none;
      @include debug(green);
    }
  }
```
通过 @include 方式引用 @mixin 定义的代码块会在最终的css文件中针对不同的选择器添加同样的样式代码。

#### 1.4 %(占位符选择器) 和 extend(选择器继承)
`%` 叫做"占位符选择器"(placeholder selector)，可以像class选择器或者id选择器那样使用，但是其本身不会被编译。 不过需要注意的是`%`需要在 @extend 指令中使用，若未使用 @extend ，则使用占位符选择器的规则集将不会被渲染为 CSS。

```css
  %debug{
    border: $border-width solid red;
  }

  ul{
    padding: 10px;
    @extend %debug;
    >li{
      list-style: none;
      @extend %debug;
    }
  }
```
编译为：
```css
ul > li, ul {
  border: 2px solid red;
}

ul {
  padding: 10px;
}
ul > li {
  list-style: none;
}
```
可以看到此种方式不会拷贝代码块，而是把选择器提前，样式只写一次。

<br>[Top](#catalog)

## <a name="Two">2. SCSS基础1</a>

#### 2.1 嵌套规则
##### 2.1.1 BEM(Block Element Modifier)命名法 
* B : 先定义一个最外层元素的类名作为主要类名，如 'user-card'
* E : 后代元素的类名以"主要类名+两个下划线+字符串"来表示，如 'user-card__name'
* M : 某个元素的不同状态以"主要类名+两个中划线+字符串"来表示，如 'user-card__name--active'

优化： 主要类名用驼峰命名法，每个后代元素用中划线连接，不同状态分开写类名，`'userCard-name active'`



<br>[Top](#catalog)

## <a name="Three">3. SCSS实践1</a>
[shake-button](https://codepen.io/innerwang/pen/rRjajV?editors=1100)


## <a name="Three">4. SCSS实践2---响应式</a>



<br>[Top](#catalog)


























