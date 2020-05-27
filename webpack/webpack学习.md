# Webpack 学习

## 1. 安装 webpack

由于 webpack4 开始，将 webpack 内核以及 webpack-cli 分开了，所以使用时需要同时安装 webpack 以及 webpack-cli

```shell
 $ yarn add webpack webpack-cli -D
 # 查看项目中安装的webpack的版本
 $ ./node_modules/.bin/webpack -v
```

### 1.1 简要使用 webpack

创建一个`webpack.config.js`文件，

```js
"use strict";
const path = require("path");
module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.join(__dirname, "dist"),
    filename: "bundle.js"
  },
  mode: "production"
};

// 在./src/index.js 编写 js， 运行webpack ， 即可以在./dist中找到打包后的bundle.js文件
```

局部安装的模块，会在`node_modules/.bin`目录创建软链接，`package.json`可以默认读取到.bin 目录下的命令，则可以通过在`package.json`的`scripts`属性中添加一条 :

```json
{
  "scripts": {
    "build": "webpack"
  }
}
```

```shell
#此时运行webpack可使用如下指令, 则会自动去node_modules中去寻找webpack
$ yarn build
```

<br><hr><br>

## 2. Webpack 基础用法一： 基础概念

### 2.1 Entry

webpack 中会将代码以及非代码如图片、字体依赖等加到依赖图中，依赖图的入口是**Entry**。对应于源代码。

webpack 会根据入口文件去寻找依赖，将依赖都加入依赖树，根据依赖树打包生成最后的文件。

#### 2.1.1 单入口： entry 是一个字符串， 适用于单页应用

```
module.exports = {
  entry: './src/index.js'
}
```

#### 2.1.2 多入口： entry 是一个对象，适用于多页应用

```js
module.exports = {
  entry: {
    app: "./src/app.js",
    adminApp: "./src/adminApp.js"
  }
};
```

### 2.2 Output

Output 用来告诉 webpack 如何将编译后的文件输出到磁盘。对应于结果代码。

#### 2.2.1 单入口

```js
module.exports = {
  entry: "./path/to/my/entry/file.js",
  output: {
    filename: "bundle.js",
    path: __dirname + "/dist"
  }
};
```

#### 2.2.2 多入口

```js
module.exports = {
  entry: {
    index: "./src/index.js",
    search: "./src/search.js"
  },
  output: {
    // 通过占位符确保文件名称的唯一， name为打包后文件的名称，还可使用 [id] 、[hash] 、[chunkhash]
    filename: "[name].js",
    path: __dirname + "/dist"
  }
};
```

build 之后，就可以在 `/dist` 目录发现`index.js`以及`search.js`

### 2.3 Loaders

Webpack 开箱即用只支持 JS 和 JSON 两种文件类型，通过 Loaders 去支持其它文件类型并且把它们转化为有效的模块，如分析转换 scss 为 css，或者把 ES6 等语法的 js 文件转换为浏览器兼容的版本，还比如将 jsx 转换为 js 等。

Loaders 本身是一个函数，接受源文件作为参数，返回转换的结果。

#### 2.3.1 常见的 Loaders

| 名称           | 描述                                                            |
| :------------- | :-------------------------------------------------------------- |
| babel-loader   | 转换 ES6、ES7 等 JS 新特性语法                                  |
| style-loader   | 将计算后的样式通过 `<style>`标签加入页面中                      |
| css-loader     | 支持.css 文件的加载和解析，将其转换为 CommonJS 对象             |
| less-loader    | 将 less 文件转换成 css                                          |
| sass-loader    | 将 sass 文件转换成 css                                          |
| ts-loader      | 将 TS 转换为 JS                                                 |
| file-loader    | 进行图片、字体等富媒体文件的打包                                |
| url-loader     | 同 file-loader，但会将小尺寸的文件进行 base64 转码生成 Data URL |
| raw-loader     | 将文件以字符串形式导入                                          |
| thread-loader  | 多进程打包 JS 和 CSS                                            |
| postcss-loader | 将 css 文件转换成 css ，如结合 autoprefixer 使用                |

#### 2.3.2 Loaders 的用法

```js
module.exports = {
  output: {
    filename: "bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.txt$/, // test 指定匹配规则
        use: "raw-loader" // use 指定使用的 loader名称
      }
    ]
  }
};
```

### 2.4 Plugins

Plugins（插件）用于打包输出的 bundle.js 文件的优化，资源管理和环境变量注入，

Plugins 作用于整个构建过程。

#### 2.4.1 常见的 Plugins 有哪些？

| 名称                 | 描述                                                                                                                          |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------- |
| SplitChunksPlugin    | 将 chunks 相同的模块代码提取成公共 js(在 `optimization.splitChunks`中配置)，替代原有的`CommonsChunkPlugin`                    |
| CleanWebpackPlugin   | 在 build 之前先清理构建目录                                                                                                   |
| MiniCssExtractPlugin | 将 CSS 文件从 bundle.js 文件中提取成一个独立的 css 文件                                                                       |
| CopyWebpackPlugin    | 将文件或者文件夹拷贝到构建的输出目录                                                                                          |
| HtmlWebpackPlugin    | 创建 html 文件去承载输出的 bundle，常用于多页面打包，可自动生成`*.html`文件，而不需要手动去 dist/ 目录中新建一个 `*.html`文件 |
| TerserWebpackPlugin  | 压缩 JS, 替换原有的`uglifyJS`                                                                                                 |
| ZipWebpackPlugin     | 将打包出的资源生成一个 zip 包                                                                                                 |

#### 2.4.2 使用 plugins

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
  output: {
    filename: "bundle.js"
  },
  plugins: [
    // 使用该插件可以根据指定的模板生成打包后的 index.html
    new HtmlWebpackPlugin({
      template: "./src/index.html"
    })
  ]
};
```

### 2.5 mode

Mode 用于指定当前的构建环境是： production、development 还是 none

设置 mode 可以使用 webpack 内置的函数，默认值为 production

mode 是 webpack4 才提出的概念。

#### 2.5.1 mode 的内置函数功能

| 选项            | 描述                                                                                                                                                                              |
| :-------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `"development"` | 设置 `process.env.NODE_ENV`的值为 `development`.会自动启用`NamedChunksPlugin` 和`NamedModulePlugin`，可以在代码热更新阶段在控制台打印是哪个模块发生了热更新，以及打印模块的路径。 |
| `"production"`  | 设置 `process.env.NODE_ENV`的值为 `production`.会自动开启`TerserPlugin` ,`SideEffectsFlagPlugin`等插件的功能，此时 webpack 会默认去压缩 js，检测变量是否有副作用等等。            |
| `"none"`        | 不开启任何优化选项                                                                                                                                                                |

<br><hr><br>

## 3. Webpack 基础用法二： 资源解析

### 3.1 解析 ES6 和 React JSX

使用 **babel-loader**, 增加 webpack 配置如下：

```js
module.exports = {
   entry: './src/index.js',
+  module: {
+    rules: [
+     {
+       // 指定使用 babel-loader 解析 .js 文件
+      	test: /\.js$/,
+     	use: 'babel-loader'
+     }
+   ]
+ }
}
```

babel-loader 依赖于 babel， 其对应的配置文件是 : `.babelrc`，安装对应的依赖，在配置文件中增加如下内容

```js
{
  "presets": [
+   "@babel/preset-env"    // 增加ES6的 babel prest配置
+   "@babel/preset-react"  // 增加 React 的 babel preset配置
  ],
  "plugins": [
    "@babel/proposal-class-properities"
  ]
}
```

对于 babel， 一个 "plugins" 对应一个功能，"presets" 是一系列 babel-plugins 的集合。

> 通常框架、组件和 utils 等业务逻辑相关的包依赖放在 dependencies 里面，对于构建、ESlint、单元测试等相关依赖放在 devDependencies 中

**使用操作步骤**：

```shell
# 安装依赖
$ yarn add @babel/core @babel/preset-env babel-loader -D
$ yarn add react react-dom
$ yarn add @babel/preset-react -D

# 根目录新建 .babelrc文件， 添加 "preset"的配置， 参见上述示例

# 修改 webpack.config.js， 在 module.rules 中添加 babel-loader的配置, 参见上述示例

```

### 3.2 解析 css、less 以及 scss

- less-loader 用于将 .less 转换为 .css

- css-loader 用于加载 .css 文件，并且转换为 commonJS 对象

- style-loader 用于将样式通过 `<style>`标签插入到 head 中

修改 webpack 配置文件如下：

```js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
+     {
+       test: /\.css$/,
+       use: ['style-loader', 'css-loader']  // loader是链式调用，从右到左，注意顺序
+     },
+     {
+       test: /\.less$/,
+       use: ['style-loader', 'css-loader', 'less-loader']
+     },
+     {
+       test: /\.scss$/,
+       use: ['style-loader', 'css-loader', 'sass-loader']
+     }
    ]
  }
}
```

**使用操作步骤**

```shell
# 安装依赖
$ yarn add style-loader css-loader -D
$ yarn add less less-loader -D   // less-loader依赖于 less，所以需要一起安装
$ yarn add node-sass sass-loader -D   // sass-loader 依赖于 node-sass

# 在webpack.config.js中添加上述配置
```

### 3.3 解析图片和字体

- file-loader 用于处理文件(图片，字体等)

webpack 配置如下：

```js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
+     {
+       test: /\.(png|jpg|gif|jpeg)$/,
+       use: 'file-loader'
+     },
+     {
+       test: /\.(woff|woff2|eot|ttf|otf)$/,
+       use: 'file-loader'
+     }
    ]
  }
}
```

**使用操作步骤**

```js
// 安装依赖
yarn add file-loader -D

// 修改webpack配置

// 示例：在 css中使用字体
@font-face {
  font-family: 'SourceHanSerifSC-Heavy';
  src: url('./SourceHanSerifSC-Heavy.otf') format('truetype');
}

h1 {
  font-family: 'SourceHanSerifSC-Heavy';
}
```

- **url-loader**，也可以解析图片和字体，此 loader 可以针对较小资源自动做 base64 转换。

```js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
+     {
+       test: /\.(png|jpg|gif|jpeg)$/,
+       use: [{
+        loader: 'url-loader',
+        options: {
+          limit: 10240    // 单位为字节， 即资源小于10k时，webpack打包时会自动 base64，大于10k时会默认自动使用 file-loader
+        }
+      }]
+    }
    ]
  }
}
```

<br><hr><br>

## 4. Webpack 基础用法三： 热更新

### 4.1 webpack 中的文件监听

文件监听是在发现源码发生变化时，自动重新构建出新的输出文件。但是需要需要手动刷新浏览器

Webpack 开启监听模式有两种方式：

- 启动 webpack 命令时，带上 `--watch` 参数 ,
- 在配置 webpack.config.js 中设置`watch: true`

文件监听的原理分析：

轮询判断文件的最后编辑时间是否变化，

某个文件发生了变化，并不会立刻告诉监听者，而是先缓存起来(放在本地磁盘)，等 aggregateTimeout

```js
module.export = {
  // 默认 false, 不开启
  watch: true,
  // 只有开启监听模式时，watchOptions才有意义
  watchOptions: {
    // 不监听的文件或者文件夹，默认为空，支持正则
    ignored: /node_modules/,
    // 监听到变化发生后会等300ms再去执行，默认300ms
    aggregateTimeout: 300,
    // 通过轮询判断文件是否变化，默认1000ms(1s)问一次
    poll: 1000
  }
};
```

### 4.2 webpack 的热更新

使用 **`webpack-dev-server(wds)`**, 主要用于开发环境

- 单独使用 WDS 时会在监听的文件变化时刷新浏览器，配合 HMR 开启热更新则不会
- WDS 不输出文件，而是放在内存中
- 还需要使用 HotModuleReplacementPlugin 插件(此插件是 webpack 内置的)

```js
// 安装依赖
npm i webpack-dev-server -D

// 修改webpack配置
module.exports = {
+  plugins: [
+    new webpack.HotModuleReplacementPlugin()
+  ],
+  // WDS 的配置
+  devServer: {
+    contentBase: './dist',
+    hot: true   // 注意设置为true，则会自动引入HotModuleReplacementPlugin
+  }
}

// 在 index.js 中调用 module.hot.accept() API
```

### 4.3 使用 webpack-dev-middleware

WDM 将 webpack 输出的文件传输给服务器，适用于灵活的定制场景.

### 4.4 热更新的原理分析

- Webpack Compile : 将 JS 编译成 bundle.js
- HMR Server: 将热更新的文件输出给 HMR Runtime
- Bundle server: 是一个服务器，提供文件在浏览器的访问
- HMR Runtime： 会被注入到浏览器，更新文件的变化
- bundle.js: 构建输出的文件

![](./images/webpack热更新原理.png)

- 第一次 build 完成，流程参考 `1---> 2---> A---> B`
- 后续代码发生变化，流程参考 `1---> 2---> 3---> 4`

这里面的热更新有最核心的是 **HMR Server** 和 **HMR runtime**。

HMR Server 是服务端，用来将变化的 js 模块通过 websocket 的消息通知给浏览器端。

HMR Runtime 是浏览器端，用于接受 HMR Server 传递的模块数据，浏览器端可以看到 .hot-update.json 的文件过来。

webpack 构建出来的 bundle.js 本身是不具备热更新的能力的，HotModuleReplacementPlugin 的作用就是将 HMR runtime 注入到 bundle.js，使得 bundle.js 可以和 HMR server 建立 websocket 的通信连接。

<br><hr><br>

## 5. Webpack 基础用法四： 其他

### 5.1 文件指纹

文件指纹指的是： 打包后输出文件名的后缀， 用于做版本管理。

<br>

#### 5.5.1 常见的文件指纹

- **Hash**: 和整个项目的构建有关，只要项目文件有更改，整个项目构建的 hash 值就会更改
- **Chunkhash**： 和 webpack 打包的 chunk(模块) 有关，不同的 entry 会生成不同的 chunkhash 值
- **Contenthash**: 根据文件内容来定义 hash，文件内容不变，则 contenthash 不变，如 css 文件

<br>

#### 5.5.2 JS 文件指纹的设置

- 设置 `webpackOptions.output.filename`，使用`[chunkhash]`

- 设置 `MiniCssExtractPlugin` 的 filename，使用`[contenthash]`

  一般会用 MiniCssExtractPlugin 将 js 文件中的 css 提取为单独的文件，所以会和`style-loader`冲突，应该需要将`style-loader`替换为`MiniCssExtractPlugin.loader`

  ```js
  yarn add mini-css-extract-plugin -D
  ```

- 设置 file-loader 的配置项 name , 使用`[hash]`

```js
  module.exports = {
    entry: {
      app: './src/app.js',
      search: './src/search.js'
    },
    output: {
+     filename: '[name]_[chunkhash:8].js',   // 8代表取hash的前8位
      path: __dirname+'/dist'
    },
    plugins: [
+     new MiniCssExtractPlugin({filename: '[name]_[contenthash:8].css'});
    ],
    module: {
      rules: [
        {
          test: /\.(png|jpg|gif|jpeg)$/,
          use: 'file-loader',
          options: {
+           name: 'static/media/[name].[hash:8].[ext]'
          }
        }
      ]
    }
  }
```

<br>

### 5.2 代码压缩

#### 5.2.1 js 文件的压缩

Webpack4 使用 terser-webpack-plugin 作为依赖，当开启 `mode = production`时会默认使用此插件进行 js 文件的压缩。但是也可以手动安装该插件，设置其他压缩参数，例如并行压缩等。

#### 5.2.2 css 文件的压缩

webpack3 可以使用设置`css-loader`的 minify 来进行 css 的压缩，但是 css-loader-v1.0 后去掉了这个参数，所以需要采用其他方式进行压缩。
webpack4 压缩 `css`则需要使用 `optimize-css-assets-webpack-plugin`插件进行 css 压缩，该插件默认使用 `cssnano`作为预处理器

- 需要注意，使用该插件压缩仅可在 `production` 模式下进行。

```js
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [new TerserPlugin(), new OptimizeCssAssetsPlugin()]
  }
};
```

#### 5.2.3 html 文件的压缩

- 使用 html-webpack-plugin，在实例化时设置压缩参数
  - bundle: 打包最终生成的文件
  - chunk： 每个 chunk 是由多个 module 组成，可以通过代码分割为多个 chunk
  - module: webpack 中的模块(js、css、图片等)

```js
module.exports = {
  plugins: [
    // 一个页面对应一个HtmlWebpackPlugin
+   new HtmlWebpackPlugin({
      //模板所在位置，可使用ejs语法
+     template: path.join(__dirname, 'src/search.html'),
      //打包出来的文件名称
+     filename: 'search.html',
      //生成的html需要的模块
+     chunks:['search'],
      // css和js会被插入 html
+     inject: true,
+     minify:{
+       html5: true,
+       collapseWhitespace:true,
+       preserveLineBreaks: false,
        // 用于压缩一开始就内联在html中的css和js，而不是打包生成的css和js
+       minifyCSS: true,
+       minifyJS: true,
+       removeComments: false
+     }
+   })
  ]
}
```

<br><hr><br>

## 6. Webpack 进阶用法

### 6.1 自动清理构建目录产物

- 通过 npm scripts 清理构建目录

  ```js
  rm -rf ./dist && webpack   // 增减前置操作
  rimraf ./dist && webpack   // 使用 rimraf 库
  ```

* 使用 `clean-webpack-plugin` ，默认会在构建之前先清空 `output.path` 指定的输出目录

  ```js
  module.exports = {
    plugins: [new CleanWebpackPlugin()]
  };
  ```

<br >

### 6.2 CSS 功能增强

#### 6.2.1 css3 的属性为什么需要前缀？

- 以 IE 为代表的使用 **Trident 内核**的浏览器，其 css 需要加上 **-ms**的前缀
- 以 Firefox 为代表的使用 **Geko 内核**，其 css 需要加上**-moz**
- 以 Chrome 为代表使用 **Webkit 内核**，其 css 需要加上 **-webkit**
- 以 Opera 为代表使用 **Presto 内核**，其 css 需要加上 **-o**

#### 6.2.2 使用 PostCSS 插件 autoprefixer 自动补全 css3 的前缀

```js
// 安装依赖
$ yarn add postcss-loader autoprefixer -D

// 修改 webpack.config.js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          'style-loader',
          { loader: 'css-loader', options: { importLoaders: 2 }},
 +        'postcss-loader',
          'sass-loader',
        },
      ]
     }
    ]
  }
}

// 添加 postcss.config.js
module.exports = {
  plugins: [require('autoprefixer')]
}


// package.json 中配置 browserlist 或者 添加 .browserlistrc
{
  "browserlist": [
    "last 2 version",
    ">1%",
    "ios 7"
  ]
}

```

<br>

### 6.3 移动端 px 自动转成 rem

原先采用 CSS 媒体查询实现响应式布局：根据屏幕大小修改样式，需要编写多套适配样式的代码。

css3 提出 rem 的单位，指的是根元素的 font-size 的大小。rem 是一个相对单位，而 px 是一个绝对单位。

- 使用 `px2rem-loader`将 px 转换为 rem
- 在页面渲染时使用手淘的`lib-flexible`库计算根元素的 font-size 的值。

```js
// 安装依赖
yarn add px2rem-loader -D
yarn add lib-flexible

// 修改 webpack.config.js
module.exports = {
  entry: './src/index.js',
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
+         {
+           loader: 'px2rem-loader',
+           options: {
+             remUnit: 75,  // 1rem=75px,适合750的设计稿
+             remPrecision: 8  // px转换为rem时小数的位数
+           }
+         }
        ]
      }
    ]
  }
}


// 内联flexible.js 库，将js和css前置放在html的头部中，不要放到js中去进行打包
// 因为需要页面一打开马上进行计算

```

px2rem-loader 只是以构建的手段将 px 单位转换成了 rem。但是 rem 和 px 的单位计算并不清楚，flexible.js 的作用就是动态的去计算不同设备下的 rem 相对 px 的单位，也就是计算根元素 html 节点的 font-size 大小。

### 6.4 静态资源的内联

资源内联的意义：

- 代码层面
  - 页面框架的初始化脚本
  - 上报相关的点
  - CSS 内联避免页面闪动
- 请求层面： 减少 HTTP 网络请求数
  - 小图片或者字体内联(url-loader)

#### 6.4.1 HTML 和 JS 的内联

```js
// raw-loader 内联 html, 之所以可以使用这种语法，是因为使用了 html-webpack-plugin,支持ejs语法
${require('raw-loader!./meta.html')}

// raw-loader 内联 JS
<script>${require('raw-loader!babel-loader!../../node_modules/lib-flexible/flexible.js')} </script>

```

注： raw-loader 需要使用 0.5.1 版本，最新版本的 raw-loader 使用了导出模块的时候使用了 export default 语法， html 里面用的话有问题。

```js
  yarn add raw-loader@0.5.1 -D
```

#### 6.4.2 CSS 内联

- 使用 style-loader
  ```js
  module.exports = {
    module: {
      rules: [
        {
          test: /\.scss$/,
          use: [
            {
              loader: "style-loader",
              options: {
                insertAt: "top", //样式插入到<head>
                singleton: true //将所有的style标签合并成一个
              }
            },
            "css-loader",
            "sass-loader"
          ]
        }
      ]
    }
  };
  ```
- 使用 html-inline-css-webpack-plugin

### 6.5 多页面应用打包

多页面是每一次页面跳转的时候，后台服务器都会返回一个新的 html 文档，每个页面之间是解耦的，对于 SEO 也更加友好。

#### 6.5.1 多页面打包基本思路

每个页面对应一个`entry`以及一个 `html-webpack-plugin`，这样的话，每次增删页面都需要修改 webpack 配置。

可以动态获取 entry 和设置 html-webpack-plugin 的数量：

- 约定入口文件都为 `src/`目录的直接子目录中的`index.js`，采用 js 脚本去读取
- 使用 [glob](https://www.npmjs.com/package/glob) 库
  ```js
  // webpack.config.js
  entry: glob.sync(path.join(__dirname, "./src/*/index.js"));
  ```

使用示例：

```js
// step1: 将入口文件重新组织，改写为 /src/**subDir**/index.js 的形式

// step2: 安装 glob依赖
yarn add glob -D

// step3 修改 webpack.config.js
const glob = require('glob')
const setMPA = () => {
  const entry = {};
  const htmlWebpackPlugins = [];

  const entryFiles = glob.sync(path.join(__dirname,'./src/*/index.js'))

  Object.keys(entryFiles).map(idx => {
    const entryFile = entryFiles[idx]
    const match = entryFile.match(/src\/(.*)\/index\.js/)
    const pageName = match && match[1]

    entry[pageName] = entryFile;
    htmlWebpackPlugins.push(
      new HtmlWebpackPlugin({
        template: path.join(__dirname, `src/${pageName}/index.html`),
        filename: `${pageName}.html`,
        chunks: [pageName],
        inject: true,
        minify: {
          html5: true,
          collapseWhitespace: true,
          preserveLineBreaks: false,
          minifyCSS: true,
          minifyJS: true,
          removeComments: false
        }
      })
    )
  })

  return {
    entry,
    htmlWebpackPlugins
  }
}

const {entry,htmlWebpackPlugins} = setMPA()

module.exports = {
  entry: entry,
  plugins: [].concat(htmlWebpackPlugins)
}

```

<br>

### 6.6 使用 sourcemap

可以通过 source map 定位到源代码，开发环境时开启，线上环境关闭，线上排查问题的时候可以将 sourcemap 上传到错误监控系统。

#### 6.6.1 source map 关键字

| 关键字     | 描述                                                                                           |
| :--------- | :--------------------------------------------------------------------------------------------- |
| eval       | 打包时使用 eval 包裹模块代码,其后会跟着一个 url，指定代码对应的文件，即 sourcemap 内联在 js 中 |
| source map | 产生.map 文件，sourcemap 与 js 进行了分离                                                      |
| cheap      | 不包含列信息，报错时只定位到行                                                                 |
| inline     | 将.map 作为 DataURI 嵌入，内联到 js 中，不单独生成.map 文件                                    |
| module     | 包含 loader 的 sourcemap                                                                       |

#### 6.6.2 source map 类型

| devtool                        | 首次构建 | 二次构建 | 是否适合生产环境 | 可以定位的代码                       |
| :----------------------------- | :------- | :------- | :--------------- | :----------------------------------- |
| (none)                         | +++      | +++      | yes              | 最终输出的代码                       |
| eval                           | +++      | +++      | no               | webpack 生成的代码(一个个的模块)     |
| cheap-eval-source-map          | +        | ++       | no               | 经过 loader 转换后的代码(只能看到行) |
| cheap-module-eval-source-map   | o        | ++       | no               | 源代码(只能看到行)                   |
| eval-source-map                | --       | +        | no               | 源代码                               |
| cheap-source-map               | +        | o        | yes              | 经过 loader 转换后的代码(只能看到行) |
| cheap-module-source-map        | o        | -        | yes              | 源代码(只能看到行)                   |
| inline-cheap-source-map        | +        | o        | no               | 经过 loader 转换后的代码(只能看到行) |
| inline-cheap-module-source-map | o        | -        | no               | 源代码(只能看到行)                   |
| source-map                     | --       | --       | yes              | 源代码                               |
| inline-source-map              | --       | --       | no               | 源代码                               |
| hidden-source-map              | --       | --       | yes              | 源代码                               |

使用示例：

```js
// webpack.config.js
module.exports = {
  devtool: "source-map" // 可以使用source map其他类型
};
```

<br>

### 6.7 提取页面公共资源

#### 6.7.1 基础库分离

- 思路： 将 react、react-dom 基础包通过 cdn 引入，不打入 bundle 中
- 方法： 使用 html-webpack-externals-plugin

```js
const HtmlWebpackExternalsPlugin = require("html-webpack-externals-plugin");

plugins: [
  new HtmlWebpackExternalsPlugin({
    externals: [
      {
        module: "react",
        entry: "https://11.url.cn/now/lib/16.2.0/react.min.js",
        global: "React"
      },
      {
        module: "react-dom",
        entry: "https://11.url.cn/now/lib/16.2.0/react-dom.min.js",
        global: "ReactDOM"
      }
    ]
  })
];
```

- 利用 SplitChunksPlugin 进行公共脚本分离

chunks 参数说明：

- "async" ---- 仅对异步引入的库进行分离(默认)
- "initial" ---- 仅对同步引入的库进行分离
- "all" ---- 所有引入的库进行分离(推荐)

  ```js
  module.exports = {
    optimization: {
      splitChunks: {
        cacheGroups: {
          commons: {
            test: /(react|react-dom)/,
            name: "vendors", // 提取出来的react 和 react-dom 混合的文件
            chunks: "all"
          }
        }
      }
    }
  };

  // 需要注意的是，需要在 new HtmlWebpackPlugin() 的配置项添加 chunks: ['vendors']
  ```

<br>

- 利用 SplitChunksPlugin 分离页面公共文件

  - minChunks : 设置最小引用次数为 2 次
  - minSize: 分离的包体积的大小

  ```js
  module.exports = {
    optimization: {
      splitChunks: {
        minSize: 0,
        cacheGroups: {
          commons: {
            name: "commons", //
            chunks: "all",
            minChunks: 2
          }
        }
      }
    }
  };
  ```

<br>

### 6.8 Tree-shaking(摇树优化)

一个模块可能有多个方法，只要其中某个方法使用到了，则整个文件都会被打到 bundle 里面去，tree shaking 就是只把用到的方法打入 bundle，没用到的方法会在 uglify 阶段被擦除掉。

使用：webpack 4 默认支持，在.babelrc 里面设置 modules：false 即可。 production mode 的情况下默认开启。

要求： 必须是 ES6 的语法，且代码没有副作用，CommonJS 的方式不支持。

#### 1. DCE(Dead Code Elimination)

代码不会被执行，不可到达；代码执行的结果不会被用到；代码只会影响死变量(只写不读)

#### 2. Tree-shaking 原理

利用 ES6 模块的特点：

- import export 只能作为模块顶层的语句出现
- import 的模块名只能是字符串常量
- import binding 是 immutable 的

代码擦除： 在 uglify 阶段删除无用代码

<br>

### 6.9 Scope Hoisting

未开启 Scope Hoisting 之前，构建后的代码存在大量闭包代码，导致体积增大，运行代码时创建的函数作用域变多，消耗更多内存。

#### 1. webpack 的模块机制

- 打包出来的是一个 IIFE(匿名闭包)
- modules 是一个数组，每一项是一个模块初始化函数
- \_\_webpack_require 用来加载模块，返回 module.exports
- 通过 WEBPACK_REQUIRE_METHOD(0)启动程序

<br>

#### 2. scope hoisting 原理

将所有模块的代码按照引用顺序放在一个函数作用域里，适当的重命名一些变量以防止变量名冲突，通过 scope hoisting 可以减少函数声明代码和内存开销。

webpack3 中需要如下配置：

```js
plugins: [new webpack.optimize.ModuleConcatenationPlugin()];
```

webpack 4 中 mode 为 production 默认开启。仅支持 ES6，不支持 CommonJS.

需要注意的是 scope hoisting 仅对引用一次的代码生效，引用多次的则会被独立的包裹函数所包裹。

<br>

### 6.10 代码分割和动态 import

对于大的 web 应用，将所有的代码都放在一个文件中(bundle.js)是不够有效的，特别是当某些代码块是在某些特殊的时候才会被用到，webpack 有一个公共就是把代码库分割成 chunks，当代码运行到需要他们时再加载。如，先打包出来首屏，当 tab 切换时，再加载其他组件。

适用的场景如： 1) 抽离相同代码到一个共享块， 2) 脚本懒加载，使得初始下载的代码更小

#### 1. 懒加载 JS 脚本的方式

- CommonJS : `require.ensure`
- ES6: 动态 import（目前还没有原生支持，需要 babel 转换）

#### 2. 使用动态 import

- step1. 安装 babel 插件

```js
  yarn add @babel/plugin-syntax-dynamic-import
```

- step2. 添加配置到 .babelrc 配置文件中

```js
plugins: ["@babel/plugin-syntax-dynamic-import"];
```

- step3. 使用示例，点击之后懒加载

```js
handleClick = () => {
  import("./text.js").then(Text => {
    this.setState({ Text: Text.default }); // 设置加载完的组件
  });
};
```

<br>

### 6.11 webpack 和 ESLint 结合

行业里面优秀的 ESLint 规范实践

- Airbnb: eslint-config-airbnb、 eslint-config-airbnb-base
- 腾讯： alloyteam 的 eslint-config-alloy、 ivweb 的 eslint-config-ivweb

#### 1. ESLint 规范： 基于 eslint-recommand 配置改进

| 规范名称         | 错误级别 | 说明                                                    |
| :--------------- | :------- | :------------------------------------------------------ |
| for-direction    | error    | for 循环的方向要求必须正确                              |
| getter-return    | error    | getter 必须有返回值，进制返回值为 undefined，如 return; |
| no-await-in-loop | off      | 允许在循环里面使用 await                                |
| no-console       | off      | 允许在代码里面使用 console                              |

#### 2. ESLint 与 CI/CD 集成

本地开发阶段增加 precommit 钩子

- step1: 安装 husky

```js
 yarn add husky
```

- step2: 增加 scripts, 通过 lint-staged 增量检查修改的文件

```js
  "scripts": {
    "precommit": "lint-staged"
  },
  "lint-staged": {
    "linters":{
      "*.{js.scss}":["eslint --fix","git add"]
    }
  }
```

#### 3. webpack 与 ESLint 集成

使用 eslint-loader

```js
// 修改 webpack.config.js

module: {
  rules: [
    {
      test: /.js$/,
      use: [
        'babel-loader',
+       'eslint-loader'
      ]
    }
  ]
}


// 创建 .eslintrc.js
module.exports = {
  "parser": "babel-eslint",  // 解析器，需要安装
  "extends": "airbnb",  // 继承 airbnb的配置，多个继承时，使用数组
  "env": {            // 指定启用的环境
    "browser": true,
    "node": true
  }
  "rules": {
      "semi": "error",
      "indent": ["error",2] // 错误级别为 error，缩进为2
  }
}
```

`eslint --fix` 脚本可以自动处理空格

<br><hr><br>

## 7.

### 7.1 Git 规范和 Changelog 生成

#### 1. Git 提交格式

- 统一 Git Commit 日志标准
- 使用 angular 的 Git commit 日志作为基本规范
  - 提交类型限制为： feat(新增 feature), fix(修复 bug), docs(仅修改了文档), style(仅修改了格式), refactor(代码重构，未增加新功能或修复 bug), perf(优化相关，如提升性能，体验), test(测试用例), chore(改变构建流程或者新加依赖库、工具), revert(回滚到上一个版本) 等
  - 提交信息分为两部分， 标题(首字母不大写，末尾不要标点)，主体内容（即描述信息）
- 日志提交时友好的类型选择提示，使用 commitize 工具
- 不符合要求格式的日志拒绝提交
  - 使用 commitlint 工具
  - 需要同时在客户端、gitlab server hook 做
- 统一 changelog 文档信息生成： 使用 conventional-changelog-cli 工具

<br><hr><br>

<br><hr><br>

<br><hr><br>
