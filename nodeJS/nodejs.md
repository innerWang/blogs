# 零基础学node.js

### 1. http模块

```js

var http = require('http');

http.createServer(function(req,res){
	res.writeHead(200,{"Content-Type":"text/html"});
	res.write('hello nodejs');
	res.end(); /*结束响应*/

}).listen(8888)

```


### 2. url模块

1. `url.parse()`方法可以对url进行解析，当第二个参数为true时，query字段（对应get传值）会被转化为对象

```js
var url = require('url');

var result = url.parse( req.url , true );
 
console.log(result.query)

```


### 3. supervisor工具

修改代码后会自动重启web服务

```
npm -g install supervisor
supervisor app.js
```


### 4. CommonJS 

CommonJS是模块化的标准，nodejs是CommonJS的实现。

node提供两类模块，一类是包含 HTTP模块、URL模块、FS模块等内置的核心模块，可以直接引入使用，另一类是用户编写的模块，称为文件模块，在运行时会动态加载，需要完整的路径分析、文件定位、编译执行过程。

自定义模块是一个单独的js文件，模块内部的方法或属性需要通过exports 或 module.exports 导出，然后在需要使用的地方通过require引入，引入时需要注意路径，可省略后缀名。

若引入的某个模块在目录下没有，node会从`node_modules` 中寻找这个模块。若此时require的路径不定位到具体的js文件，则会找该文件目录下的package.json(`npm init -y`可生成)中的入口文件，即“main”的值。


```js
/*
  --demo/
    ----node_modules/
        ----nav/
            ---nav.js
            ---package.json
        ----bar/
            ---bar.js
        ----foo.js
    ----app.js

 */


// app.js
var nav = require('nav')
var bar = require('bar/bar.js')
var foo = require('foo')

console.log(nav.str)
console.log(bar.str)
console.log(foo.str)

```

### 5. fs文件系统模块

##### 5.1 `fs.stat` 检测是文件还是目录
需要注意的是fs.stat是一个异步方法。

```js
fs.stat('views',(err,stats)=>{
    if(err){
        console.log(err)
        return false
    }

    console.log('文件： '+stats.isFile())
    console.log('目录： '+stats.isDirectory())
})

```

##### 5.2 `fs.mkdir` 创建目录
可接受三个参数：
1. path  : 创建的目录路径
2. mode  : 权限，默认为777(可读可写可执行)
3. callback(Function) : 执行出错时的回调

##### 5.3 `fs.writeFile` 写入文件
当文件不存在时，会创建，当文件存在时，会**覆盖**文件中的内容。


##### 5.4 `fs.appendFile` 追加文件
当文件不存在时，会创建，当文件存在时，会**追加**文件中的内容。


##### 5.5 `fs.readFile` 读取文件

fs.readFile是异步地读取文件的全部内容。，若想要同步读取，可使用fs.readFileSync

```js
fs.readFile('a.txt',(err,data)=>{
  if(err){
    console.log(err)
    return false
  }
  console.log(data.toString()) 
})
```
##### 5.6 `fs.readdir`读取目录
获取目录下面的文件以及文件夹


##### 5.7 `fs.rename`重命名
可以实现改名以及剪切文件
```js
fs.rename('html/style.css','html/css/basic.css',err=>{
    if(err){
        console.log(err)
        return false
    }
    console.log('剪切成功')
})
```

##### 5.8 `fs.rmdir`删除目录
此方法只能删除目录。

##### 5.9 `fs.unlink`删除文件


##### 5.10 `fs.createReadStream`读取文件

调用`fs.createReadStream`创建一个`fs.ReadStream` 对象， 所有 `fs.ReadStream` 对象都是[可读流(`stream.Readable`)](http://nodejs.cn/api/stream.html#stream_class_stream_readable)。

文件比较大时，采用文件流的方式进行读取，不会卡死。
```js
var fs = require('fs')

var readStream = fs.createReadStream('a.txt')
var str=''
var count = 0
readStream.on('data',function(chunk){
    str += chunk;
    count++;
})

readStream.on('end',function(){
    console.log(str)
    console.log(count)
})

readStream.on('error',function(){
    console.log('读取失败')
})

```


##### 5.11 `fs.createWriteStream` 写文件

```js
var fs = require('fs')
var data="我是数据，需要保存\n";

//创建一个写入流，写内容到output.txt中
var writeStream = fs.createWriteStream('output.txt')
writeStream.write(data,'utf8');

//标记写入完成
writeStream.end();

writeStream.on('finish',function(){
    console.log('写入完成')
})

writeStream.on('error',function(){
    console.log('写入失败')
})
```

##### 5.12 管道读写

```js
var fs = require('fs')
var readStream = fs.createReadStream('a.txt')
var writeStream = fs.createWriteStream('output.txt')

//管道读写操作，读取a.txt中的内容，写入到output.txt中

readStream.pipe(writeStream)

```

### 6. path模块

path 模块提供用于处理文件路径和目录路径的实用工具。

##### 6.1 `path.extname(path)`

返回传入参数path的扩展名，包含`.`，若字符串path中没有`.`或者第一个字符是`.`，返回空字符串。



### 7. nodejs回调和事件驱动

nodejs中大多API是异步的，如下，想要获取文件中的内容，会得到undefined。
```js
var fs = require('fs')
function getMime(){
    fs.readFile('mime.json',function(err,data){
        return data.toString()
    })
}

console.log(getMime());  //undefined

```
我们可以使用两种方式来获取异步数据，回调函数和事件驱动。

##### 7.1 回调函数获取异步数据
```js
var fs = require('fs')
function getMime(callback){
    fs.readFile('mime.json',function(err,data){
        callback && callback(data.toString())
    })
}

getMime(function(result){
    console.log(result)
});

```


##### 7.2 事件驱动的方式获取异步数据
事件驱动的方式主要是使用node的内置模块events。

```js
var fs = require('fs')
var events = require('events')

var EventEmitter = new events.EventEmitter()

EventEmitter.on('mime_data',function(data){
    console.log('接收到mime数据,data='+data)
})


function getMime(){
    fs.readFile('mime.json',function(err,data){
        EventEmitter.emit('mime_data',data.toString())
    })
}

getMime()
```

### 8. node.js路由，get和post提交数据

使用[http.createServer](http://nodejs.cn/api/http.html#http_http_createserver_options_requestlistener)创建一个[http.Server](http://nodejs.cn/api/http.html#http_class_http_server)实例:
```js
    http.createServer([options][, requestlistener])
```
其中的 requestlistener 是一个自动添加到 [request](http://nodejs.cn/api/http.html#http_event_request) 事件的函数, 其两个参数`req`,`res`分别为[`http.IncomingMessage 类`](http://nodejs.cn/api/http.html#http_class_http_incomingmessage)和[`http.ServerResponse 类`](http://nodejs.cn/api/http.html#http_class_http_serverresponse)的实例。

所谓路由也就是根据不同的url以及get/post的参数执行对应的代码，获取不同的资源。

* 当使用get方式进行请求时，可通过`url.parse(req.url,true).query`获取get传值
* 当使用post方式进行请求时,可通过监听事件(参考**可读流**的事件)得到数据
```js
    var postData = '';
    req.on('data',function(chunk){
        postData +=chunk;
    })
    req.on('end',function(){
        console.log(postData)
    })

```































