# 图解HTTP

## <span id="catalog">目录</span>
* [第1章 了解Web及网络技术](#one)
* [第2章 简单的HTTP协议](#two)
* [第3章 HTTP报文的HTTP信息](#three)
* [第4章 HTTP状态码](#four)
* [第5章 Web服务器](#five)
* [第6章 HTTP首部](#six)
* [第7章 HTTPS协议](#seven)
* [第8章 访问用户身份的认证机制](#eight)
* [第9章 基于HTTP新增功能的协议](#nine)
* [第10章 Web攻击](#ten)

<hr>


## <span id="one">第1章 了解Web及网络技术</span>

### 1. TCP/IP 协议族
TCP/IP 分为四层，包含 应用层(FTP、DNS、HTTP等)、传输层(TCP、UDP)、网络层以及链路层。

#### 1.1 IP (Internet Protocol)网际协议

  IP协议主要用于将各种数据包传送给对方，其中需要两个重要的条件，也即 IP地址和MAC地址。IP地址可变换但是MAC地址基本不会更改，可通过ARP协议解析IP地址得到对应的MAC地址。

#### 1.2 确保可靠性的TCP协议
  TCP的三次握手可保证将数据送达目标处，主要使用了 SYN(synchronize)和ACK(acknowledgement)两个标志。
  * 发送 SYN 标志的数据包
  * 回复 SYN/ACK 的数据包
  * 发送 ACK 的数据包

#### 1.3 解析域名的DNS服务
  DNS 协议提供通过域名查找IP地址，或逆向从IP地址反查域名的服务。

### 2. URI(统一资源标识符)与URL(统一资源定位符)

URI 指由某个协议方案表示的资源的定位标识符，即用字符串标识某一互联网资源。而URL标识资源的地点，即URL是URI的子集。
  * 协议方案： http、ftp、file等


<br>
<hr>

## <span id="two">第2章 简单的HTTP协议</span>
基于**HTTP / 1.1**对HTTP协议结构进行讲解。

HTTP的初始版本中，每进行一次HTTP通信就要断开一次TCP连接，现在使用浏览器浏览包含多张图片的HTML页面时，请求页面包含的其他资源则会造成无谓的TCP连接建立和断开，增加通信的开销。

为解决上述TCP连接的问题，HTTP/1.1和一部分的HTTP/1.0提出了持久连接(HTTP keep-alive)的方法，也即只要有任意一端未明确提出断开连接，则保持TCP连接状态。

HTTP/1.1中所有连接默认是持久连接，这样可以减少TCP连接的重复建立和断开带来的通信开销，减轻服务端的负载，同时可更快的响应HTTP请求。

持久连接使得多数请求以管线化方式发送成为可能，此时可以不用等待当前请求的响应直接发送下一个请求。
<br>
<hr>

## <span id="three">第3章 HTTP报文的HTTP信息</span>

HTTP报文本身是由多行数据构成的字符串文本。

HTTP传输可以按照数据原貌直接传输，也可以在传输过程中通过编码(如 gzip、compress、deflate、identity)提升传输速率。

HTTP报文(message)是HTTP通信中的基本单位，由 **`8位组字节流`** 组成，通过HTTP通信传输。

HTTP实体(entity)是请求或响应的有效载荷数据，由实体首部和实体主体组成。


HTTP通信过程中，请求的编码实体资源尚未全部传输完成之前，浏览器无法显示页面，传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面，此功能叫做分块传输编码(Chunked Transfer Coding)。

MIME(Multipurpose Internet Mail Extensions, 多用途因特网邮件扩展)允许邮件可以发送多种类型的文件，同样的，HTTP协议也采纳了多部分对象集合(Multipart)，发送的一份报文主体可以包含多类型实体(使用`Content-type`来指定类型)，常用于图片或文本文件上传。
* multipart/form-data  用于Web表单文件上传
* multipart/byteranges  响应多个范围的内容时使用，状态码对应返回206。
  * Range 可指定资源的byte范围。 
  ```js
   // 取200到1000的范围
   Content-Range: bytes 200-1000/67343
  ```

HTTP的客户端和服务端利用如下字段进行内容协商：
* Accept
* Accept-Charset
* Accept-Encoding
* Accept-Language
* Content-Language

<br>
<hr>


## <span id="four">第4章 HTTP状态码</span>
### 1. 状态码的分类
||类别|原因短语|
|:-|:-|:-|
|1xx|Informational(信息性状态码)|表示接受的请求正在处理|
|2xx|Success(成功状态码)|表示请求正常处理完毕|
|3xx|Redirection(重定向状态码)|表示需要额外的操作完成请求|
|4xx|Client Error(客户端错误状态码)|服务器无法处理请求|
|5xx|Server Error(服务器错误状态码)|服务器处理请求出错|

* `200 OK` 表示请求在服务端被正常处理。
* `204 No Content` 表示没有资源可以返回，响应报文不包含实体的主体。
* `206 Partial Content` 表示客户端是进行了范围请求，只返回指定范围的实体内容。
* `301 Moved Permanently` 永久性重定向，表示请求的资源被分配了新的URI。
* `302 Found` 临时性重定向，表示希望本次能用新的URI进行访问。
* `303 See Other` 表示请求的资源有另一个URI，需要使用**GET方法**请求新的URI获取资源。
* `304 Not Modified` 表示客户端的请求带有条件，虽然服务端允许该请求访问资源，但是资源并未满足条件。
* `307 Temporary Redirect` 临时重定向，与302类似，但是307会严格遵守浏览器标准，不会将POST方法改为GET。
* `400 Bad Request` 表示请求报文存在语法错误。
* `401 Unauthorized` 表示请求必须含通过HTTP认证的认证信息，第一次返回401时浏览器会弹出认证用的对话窗口，第二次则表示用户认证失败。
* `403 Forbidden` 表示服务器拒绝了对请求资源的访问。
* `404 Not Found` 表示服务器上找不到请求的资源。
* `500 Internal Server Error` 表示服务端在执行请求时发生错误
* `502 Bad Gateway` 表示服务器压力大，超时处理请求而抛错。
* `503 Service Unavailable` 表示服务端正在停机维护，无法处理请求。



> 1. 返回301、302、303状态码时，几乎所有的浏览器都会把POST改为GET，删除请求报文的主体，之后请求会自动再次发送。 虽然 301、302标准禁止将POST方法改为GET，但实际使用时大家都会这么做。

<br>
<hr>


## <span id="five">第5章 Web服务器</span>

一台物理主机可以使用虚拟主机的功能搭建多个Web站点，为了便于虚拟主机的区分，客户端在发送请求时需要在首部内指定URI。

**代理**是具有转发功能的应用程序，接收由客户端发送的请求并转发给服务器，同时接收服务器返回的响应转发给客户端。代理不会改变请求的URI，会直接转发请求给持有资源的目标服务器。每次通过代理服务器转发请求或响应时，会追加写入Via首部信息，标记经过的主机信息。
* 缓存代理：代理转发响应时，预先将资源副本进行保存，等客户端再次请求时，则将缓存的资源作为响应返回。
* 透明代理：不对报文内容做任何加工的代理，反之叫做非透明代理。

**网关**是转发其他服务器通信数据的服务器。网关与代理的工作机制类似，但是网关可以使通信线路上的服务器提供非HTTP协议服务。

**隧道**是在相隔甚远的客户端和服务器两者之间进行中转并保持双方通信连接的应用程序。隧道可以按照要求建立一条与其他服务器的通信线路，届时可使用SSL等加密手段进行通信，确保客户端与服务器进行安全的通信，隧道会在通信双方断开连接时结束。

**缓存**不仅可以存在于缓存服务器内，还可以存在客户端浏览器中，浏览器缓存若有效，可再次请求时可直接从本地磁盘内读取。

<br>
<hr>

## <span id="six">第6章 HTTP首部</span>
HTTP协议的请求和响应报文必定包含HTTP首部。**报文首部**提供处理请求和响应所需要的信息，**报文主体**包含所需要的资源信息。

请求报文首部包含 请求行(含方法、URI、HTTP版本)、请求首部字段、通用首部字段、实体首部字段以及其他。

响应报文首部包含 状态行(含HTTP版本、状态码)、响应首部字段、通用首部字段、实体首部字段以及其他。

HTTP首部由字段名和字段值构成，中间用冒号分隔。

#### 通用首部字段
指请求报文和响应报文都会使用的首部。
|首部字段名|说明|
|:-|:-|
|Cache-Control|控制缓存的行为，可设置： `no-cache`，`max-age=秒`(表示响应的最大Age值)等|
|Connection|1. 控制不再转发给代理的首部字段，如设置为`Upgrade`，则代表删除该字段再转发；2. 管理持久连接，如设置为 `Keep-Alive`/`close`|
|Date|表示创建报文的日期和时间|
|Pragma|用于客户端发送的请求中，如设置为`no-cache`表示所有的中间服务器不要返回缓存资源|
|Trailer|用于说明报文主体后记录了哪些首部字段|
|Transfer-Encoding|指定传输报文主体时的编码方式|
|Upgrade|用于检测HTTP协议及其他协议是否可以使用更高版本进行通信，如指定为`TLS/1.0`|
|Via|用于记录代理服务器的相关信息|
|Warning|一般用于通知用户一些与缓存相关的问题的警告|

#### 请求首部字段

|首部字段名|说明|
|:-|:-|
|Accept|用于通知服务器用户代理可处理的媒体类型及相对优先级，一般使用`type/subtype`的形式，如文本文件的`text/html`，应用程序使用的二进制文件`application/octet-stream`|
|Accept-Charset|通知服务器用户代理优先支持的字符集及优先级，如`iso-8859-5`|
|Accept-Encoding|通知服务器用户代理优先支持的内容编码及优先级，如`gzip`|
|Accept-Language|优先的语言，如`zh-cn`|
|Authorization|用于告知服务器用户代理的认证信息(证书值)|
|Except|期待服务器的特定行为，如 `100-continue`|
|From|用户的电子邮箱地址|
|Host|请求资源所在服务器的主机名和端口号，必含！|
|If-Match|条件请求，需要比较是否与实体标记(ETag)一致，一致才会接受请求|
|If-Modified-Since|条件请求，需要比较资源的更新时间，当指定时间之后资源发生了更新，服务器才会接受请求|
|If-None-Match|服务端比对该字段与ETag不一致时接受请求|
|If-Range|资源未更新时发送实体Byte的范围请求|
|If-Unmodified-Since|比较资源的更新时间，在指定时间之后未发生更新，才可处理请求|
|Max-Forwards|最大传输逐跳数，每次转发该数值减1，当服务器收到请求该值为0时，则不再进行转发，而是直接返回响应|
|Proxy-Authorization|代理服务器要求客户端的认证信息|
|Range|对实体的范围请求，单位为字节|
|Referer|发起请求的页面URI|
|TE|传输编码的方式及优先级|
|User-Agent|用户传达浏览器的种类|


#### 响应首部字段 
|首部字段名|说明|
|:-|:-|
|Accept-Ranges|是否接受字节范围请求，`bytes`表示可处理范围请求，`none`表示无法处理|
|Age|推算资源创建到现在的时间，单位为秒|
|ETag|资源的匹配信息，资源更新时，ETag也要更新|
|Location|令客户端重定向至指定的URI|
|Proxy-Authenticate|代理服务器对客户端的认证信息|
|Retry-After|告知客户端应在多久之后再次发起请求，配合503或3xx一起使用|
|Server|HTTP服务器的应用程序的信息|
|Vary|代理服务器缓存的管理信息，当请求中Vary所指定的首部字段的值相同时，则返回缓存，否则从服务器获取资源|
|WWW-Authenticate|服务器对客户端的认证信息|



#### 实体首部字段
针对请求和响应报文的实体部分使用的首部，补充与实体相关的信息。

|首部字段名|说明|
|:-|:-|
|Allow|用于通知客户端其所请求的资源支持的HTTP方法|
|Content-Encoding|用于告知客户端返回的实体主体选用的编码方式|
|Content-Language|用于告知客户端实体主体的自然语言|
|Content-Length|用于告知客户端实体主体的大小(字节)|
|Content-Location|用于给出与报文主体资源对应的URI，如`http://www.aaa.cn/index.html`|
|Content-MD5|实体主体进行MD5(得到128位二进制，无法写入首部)再base64得到的结果，用于客户端校验实体主体是否完整或有误|
|Content-Range|实体主体的位置范围|
|Content-Type|实体主体内对象的媒体类型|
|Expires|实体主体过期的日期时间|
|Last-Modified|资源的最后修改日期时间|


#### Cookie使用的首部字段
|首部字段名|说明|首部类型|
|:-|:-|:-|
|Set-Cookie|用于告知客户端，服务端将要管理的状态|响应首部字段|
|Cookie|发送给服务器的Cookie信息|请求首部字段|

1. `Set-Cookie`字段的值

|属性|说明|
|:-|:-|
|NAME=VALUE|赋予Cookie的名称和值(必需)|
|expires=DATE|Cookie的过期时间，不设置则默认到浏览器关闭为止|
|path=PATH|将服务器上的文件目录作为Cookie的适用对象|
|domain=域名|Cookie使用对象的域名|
|Secure|仅在使用HTTPS时才会发送Cookie|
|HttpOnly|限制不可以使用js脚本访问Cookie|

```js
   Set-Cookie: name=value; HttpOnly
```

<br>
<hr>


## <span id="seven">第7章 HTTPS协议</span>
HTTP本身是明文传输，加密可采取以下几种方式：
* 通信加密，将HTTP和SSL、TLS组合使用，用SSL建议安全通信线路，与SSL组合使用的HTTP称为HTTPS
* 内容加密，此时仍会有内容被篡改的风险

通常HTTP直接和TCP通信，当使用了SSL时，则演变为先和SSL通信，再由SSL和TCP通信。


<br>
<hr>

## <span id="eight">第8章 访问用户身份的认证机制</span>
HTTP/1.1 使用的认证方式如下：
* BASIC 认证
* DIGEST 认证
* SSL 客户端认证
* FormBase 认证

<br>
<hr>

## <span id="nine">第9章 基于HTTP新增功能的协议</span>

### 1. SPDY协议
* SPDY未完全改写HTTP协议，而是在TCP/IP的应用层和传输层直接新加会话层，且规定通信使用SSL。即为 `HTTP(应用层) --- SPDY(会话层) --- SSL(表示层) --- TCP(传输层)`。
* 使用SPDY协议后，HTTP多出如下功能：
  * 多路复用流， 可使用一条TCP连接，无限制处理多个HTTP请求
  * 赋予请求优先级， 给请求逐个分配优先级顺序，解决发送多个请求时，带宽低而导致响应慢的问题
  * 压缩HTTP首部，减少通信产生的数据包数量和发送的字节数
  * 推送功能， 支持服务器主动向客户端推送数据
  * 服务器提示功能，服务器可主动提示客户端请求所需的资源，客户端则可以缓存资源，避免不必要的请求


 ### 2. 使用浏览器进行全双工通信的WebSocket协议
一旦Web 服务器与客户端之间建立起WebSocket协议的通信连接，则可以发送任何格式的数据。

由于ws建立在http协议基础上，所以发起连接的仍然是客户端，一旦确立websocket的通信连接，任意一方都可以直接向对方发送报文。

ws的主要特点：
* 推送功能，服务器可直接发送数据，不必等待客户端的请求
* 减少通信量，在建立http连接后，需要完成一次握手来实现ws通信，此时需要用到HTTP首部的Upgrade字段，告知服务器通信协议发生变化，以达到握手的目的，服务器则会返回`101 Switching Protocols`的响应，成功握手后，则通信时不再使用HTTP的数据帧，而是使用ws独立的数据帧。


ws的API：
```js 
 var ws = new WebSocket(url[, protocols])  // 返回一个ws对象
 ws.onopen = () => {}  // 定义连接成功后的回调函数
 ws.onerror = () => {} // 定义发生错误时执行的回调函数
 ws.onclose = () => {} //  定义连接关闭时的回调函数
 ws.onmessage = e => {}  // 定义从服务器接收到信息的回调函数
```

### 3. HTTP/2.0
主要围绕以下7项技术：
* 压缩
* 多路复用
* TLS 义务化
* 协商
* 客户端拉曳 / 服务端推送
* 流量控制
* WebSocket

<br>
<hr>

## <span id="ten">第10章 Web攻击</span>

### xss 跨站脚本攻击
### cros 跨站请求伪造
### DoS攻击(Denial of Service attack)
* 集中利用访问请求造成资源过载，资源用尽，使服务器呈停止状态
* 通过攻击安全漏洞使服务停止





[返回目录](#catalog)