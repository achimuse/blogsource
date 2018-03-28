---
title: http协议小记
date: 2018-03-26 17:03:56
tags:
 - web
---

### URI/URL/URN
![](/assets/blogImgs/url.jpg)
URI是 Uniform Resource Identifiers 的简写，统一资源标识符。  
URL是 Uniform Resource Locators 的简写，统一资源定位符。  
URN是 Uniform Resource Names 的简写，统一资源命名符。  

URL和URN是URI的子集，以下例子都是URI，但不一定都是URL。  

>ftp://ftp.is.co.za/rfc/rfc1808.txt (also a URL because of the protocol)  
>http://www.ietf.org/rfc/rfc2396.txt (also a URL because of the protocol)  
>ldap://[2001:db8::7]/c=GB?objectClass?one (also a URL because of the protocol)
>mailto:John.Doe@example.com (also a URL because of the protocol)
news:comp.infosystems.www.servers.unix (also a URL because of the protocol)  
>tel:+1-816-555-1212  
>telnet://192.0.2.16:80/ (also a URL because of the protocol)  
>urn:oasis:names:specification:docbook:dtd:xml:4.1.2  

### 被动性  
http协议在一条通路上必然有一端是客户端另一端是服务端，而且只能由客户端发出请求，服务端响应该请求。  

### 常用方法
http请求的根本目标是操作资源，所以请求中使用URI定位资源，使用方法表明意图(restful)：  
- GET方法，请求资源，查  
- POST方法，推送资源，增  
- PUT方法，改动资源，改  
- DELETE方法，删除资源，删
- HEAD方法，同GET，响应不返回报文主体，用于确认URI有效性和资源更新时间  

### 持久连接与管道化
持久连接（HTTP Persistent Connections），建立TCP连接后只要任一端没有提出断开连接，则保持连接状态，提高请求相应速度。在同一TCP连接上会有多有HTTP连接，同时线管化（Pipelining）方式使得不用等待上个请求的相应就可以发送下一个请求。  

### 无状态与Cookie
stateless，不会保存请求或相应的状态，优点在于减少服务器负担。如果需要状态则使用Cookie。客户端第一次向服务端发出请求（要求Cookie），服务端生成Cookie并记录下Cookie与客户端的映射，在响应返回Cookie,客户端收到后保存。之后的每个请求都带上Cookie，即保存了状态。Cookie就是一个字符id。  

### 报文结构
http报文实际是字符串文本，用CR+LF空行划分报文的首部和主体。报文分为请求报文和响应报文。请求行包含请求方法、URI、HTTP版本，状态行包含状态吗、原因短语、HTTP版本。  
![](/assets/blogImgs/httpPacket.jpg)  
![](/assets/blogImgs/httpPacketReal.jpg)  

### 传输
http报文在传输过程中，通过内容编码提升传输速率，通过分块实现浏览器逐步显示。  

**报文（message）**  
http通信的基本单位，由octet sequence组成。如一个完整的请求或响应就是一个message。  

**实体（entity）**  
请求或响应中的有效载荷数据，即实体首部和实体主体。  

**内容编码**  
对实体entity进行压缩（gzip/compress/deflate/identity）后传输，类似于发邮件时压缩附件。  

**分块传输编码（chunked transfer coding）**  
将实体主体分成多块，每一块用十六进制标记大小，最后一块使用“0（CR+LF）”标记。  

**多部分对象集合Multipart**  
邮件采用了MIME（multipurpose internet mail extensions）机制使其支持处理文本（文本文件）、图片（二进制文件）等多种数据类型。http采用了其中的multipart方法，用于支持多类型数据。  
* multipart/form-data  
上传web表单文件时使用。
![](/assets/blogImgs/formData.jpg)
以下是一个真实的post请求，使用表单上传.xlxs表格  
![请求头部](/assets/blogImgs/formRealHead.jpg)
![请求主体](/assets/blogImgs/formRealEntity.jpg) 

* multipart/byteranges  
范围请求（Range Request）指定请求实体范围，实现了了下载中断恢复功能。    
在请求首部加上字段`Content-Range: bytes 7000-7999/8000`,返回状态码206（Partial Content）的响应报文。响应报文首部会有字段`Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES`。  
返回的实体都会有`--THIS_STRING_SEPARATES`作为分界线，最后以`--THIS_STRING_SEPARATES--`标记结束。
![响应首部](/assets/blogImgs/byterangesHead.jpg)
![响应主体](/assets/blogImgs/byterangesEntity.jpg)   

### 内容协商（Content Negotiation）
客户端会与服务端协商请求最适合的资源，主要在与资源的语言、字符集、编码方式。故请求的首部字段会有`Accept`/`Accept-Charset`/`Accept-Encoding`/`Accept-Language`/`Content_Language`等协商字段。  

### 状态码
![响应状态码](/assets/blogImgs/statusCode.jpg) 
**2XX 成功**  
`200 OK`：请求处理正常   
`204 No Content`：请求处理正常，相应报文中无主体  
`206 Partial Content`：范围请求  

**3XX 重定向**  
表明浏览器需要执行响应中的处理请求  
`301 Moved Permanently`：永久重定向，资源分配到了新的URI，如果资源已被保存书签，浏览器会更新响应报文中`Location`字段的URI。  
`302 Found`：临时重定向，希望浏览器请求响应报文中`Location`字段中的URI，方法未指定，通常是GET。  
`303 See Other`：与302相同，但明确了浏览器使用GET方法。  

**4XX 客户端错误**  
表明请求中存在错误
`401 Unauthorized`：用户需要认证或者认证失败，浏览器会弹出认证窗口。  
`403 Forbidden`：拒绝请求访问。  
`404 Not Found`：服务端无请求的资源。  

**5XX 服务端错误**  
`500 Internet Server Error`：执行请求出错，服务端内部错误。  
`503 Service Unavailable`：服务端出于停机维护状态。  

### 通信数据转发
HTTP实际通信中，除了客户端和服务端，还有其他通信数据转发程序。 

**代理**  
中间人程序，接收客户端请求转发给服务端，接收服务端响应转发给客户端。代理有两种基本分类，一种是是否使用缓存，另一种是否修改报文。  
缓存代理（Caching Proxy），预先保留资源缓存，接收到相同资源请求时直接响应返回资源缓存。  
透明代理（Transparent Proxy），转发请求或响应时不改动报文。反之为非透明代理。  

**网关**  
类似于代理，但网关能使通信线路上的服务器提供非http协议服务（比如微服务架构的kong网关）。  
![](/assets/blogImgs/gateway.jpg)  

**隧道**  
隧道建立安全的通信线路，不会解析http报文，使用SSL加密通信。  

### 缓存
缓存代理服务器可以避免多次从源服务器转发资源，客户端可更快地获取资源，也减轻服务端处理负担。  

**TTL（Time to Live）**  
由于源服务器资源更新时，缓存无法保证其有效性。缓存服务器会依据包含`no-cache`字段的请求、缓存有效期等因素，向源服务器确认资源的有效性，判断失效则会拉取新的资源。  

**客户端缓存**  
浏览器也可以缓存一定的资源内容。  

## HTTP首部字段
首部字段是报文的要素，表明报文主体大小、使用的语言、编码方式、认证信息等等。
### 首部字段结构
由首部字段名和字段值组成，用:分隔。例如Content-Type字段，表明报文主体的对象类型。
![](/assets/blogImgs/headerField.jpg)
单个首部字段可以有多个字段值。
![](/assets/blogImgs/headerFields.jpg)
### 首部字段类型
#### 通用首部字段（General Header Fields）
请求报文和响应报文两方都会使用的首部。  
![](/assets/blogImgs/generalHeader.jpg)  
**Connection**  
 转发过程中删除某些字段，例如`Connection: Hop-by-hop`在代理转发给服务端前会删除Hop-by-hop字段。  
 管理持久连接，例如`Connection: close`表明是断开tcp连接；客户端请求`Connection: Keep-Alive`，服务端响应`Keep-Alive: time-out=10, max=500`+`Connection: Keep-Alive`表明是建立持久连接。  

**Transfer-Encoding**  
规定传输报文主体编码方式，实际上HTTP/1.1中只对分块编码有效，其他的编码方式由内容编码字段`Content-Encoding`指定。  
以下响应报文，采用了分块编码，被分成3312字节和914字节大小的分块，最后以0字节表示块结束。   
![](/assets/blogImgs/transferCoding.jpg) 

#### 请求首部字段（Request Header Fields）
从客户端向服务器端发送请求报文时使用的首部。补充了请求的附加
内容、客户端信息、响应内容相关优先级等信息。  
![](/assets/blogImgs/requestHeader.jpg)
**Accept**  
通知服务端用户代理支持的媒体类型及其优先级，媒体类型有：
- 文本文件如text/html, text/plain, text/css, application/xhtml+xml, application/xml.  
- 图片文件（二进制文件）如image/jpeg, image/gif, image/png.  
- 视频文件（二进制文件）如video/maped, video/quicktime.  
- 应用程序使用的二进制文件如application/octet-stream, application/zip.  
指定优先级可在字段值后加上q=，分号;分隔，范围0.000--1.000，不指定默认q=1.0。  
`Accept: text/html,application/xhtml+xml,application/xml;q=0.7`  

**Accep-Charset**  
来通知服务器用户代理支持的字符集及字符集的相对优先顺序，优先级同上。  
`Accept-Charset: iso-8859-5, unicode-1-1;q=0.8`  

**Accept-Encoding**  
通知服务器用户代理支持的内容编码（使用星号\*表示任意格式）及内容编码的优先级顺序，优先级同上。  
`Accept-Encoding: gzip, deflate, compress, identity`  
同样的Accept-Language字段表明支持的自然语言集。  
`Accept-Language: zh-cn,zh;q=0.7,en-us,en;q=0.3`  

**Authorization**  
用户代理接收到响应状态码为401 Unauthorized时，会把首部字段Authorization加入请求，包含着认证信息。  
`Authorization: Basic dWVub3NlbjpwYXNzd29yZA==`  

**Host**  
多个虚拟主机在同一个IP上，使用Host字段可以加以区分,若服务端未设主机名直接发送空值即可。  
`Host: www.hackr.jp`    

**User-Agent**  
将创建请求的浏览器和用户代理名称等信息传达给服务器。如果经过代理也可能加上代理服务器名称。    
`User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:13.0)`

**Max-Forwards**  
报文途经多个代理服务器有可能在代理间变成死循环，Max-Forwards的值是一个数字，每经过一个服务器就会减1，为0之后服务器不再转发并直接返回响应。（实际上tcp/ip的路由算法里也有类似机制）  

**Range**  
对于只需获取部分资源的范围请求，包含首部字段 Range 即可告知服
务器资源的指定范围。  
接收到附带 Range 首部字段请求的服务器，会在处理请求之后返回状
态码为 206 Partial Content 响应。无法处理该范围请求时，则会返
回状态码 200 OK 的响应及全部资源。  
`Range: bytes=5001-10000`   

#### 响应首部字段（Response Header Fields）
从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加
内容，也会要求客户端附加额外的内容信息。  
![](/assets/blogImgs/responseHeader.jpg)
**Accept-Ranges**  
告知客户端服务端能否处理范围请求Range，可处理范围请求时指定其为 bytes，反之则指定其为 none。  
`Accept-Ranges: bytes`  
`Accept-Ranges: none`  

**Age**  
缓存服务器用于告知客户端，该响应是源服务器在多久前创建的，单位sec。  
`Age: 600`  

**Location**  
配合 3xx ：Redirection 的响应，提供重定向的URI。浏览器接收响应后会对Location的URI进行访问。  
`location: https://account.aliyun.com/login/login.htm?oauth_callback=https%6A%2Fecs.console.aliyun.com%7F`

**Server**  
告知客户端当前服务器上安装的 HTTP 服务器应用程序的信息。标出服务器上的软件应用名称，可能包括版
本号和安装启用的可选项，比如以下Bing的Server信息。  
`server: Microsoft-IIS/10.0`  

**WWW-Authenticate**  
用于 HTTP 访问认证。告知客户端适用于访问请求 URI 指定资源的认证方案（Basic 或是Digest）和带参数提示的质询（challenge）。状态码 401 Unauthorized 响应中，一定带有首部字段 WWW-Authenticate。  
`WWW-Authenticate: Basic realm="Usagidesign Auth"`  

#### 实体首部字段（Entity Header Fields）
针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更
新时间等与实体有关的信息。  
![](/assets/blogImgs/entityHeader.jpg)
**Content-Encoding**  
告知客户端实体的主体部分选用的内容编码方式（就是压缩方式）。  
`Content-Encoding: gzip`  

**Content-Language**  
告知客户端实体主体使用的自然语言。  
`Content-Language: zh-CN`  

**Content-Length**  
告知客户端实体主体部分的大小（单位是字节）。  
`Content-Length: 15000` 

**Content--Type**  
告知客户端实体主体内对象的媒体类型, 和首部字段Accept一样，字段值用 type/subtype 形式赋值。  
`Content-Type: text/html; charset=UTF-8`  

**Content--Range**  
针对范围请求，返回响应时使用的首部字段Content-Range，能告知客户端作为响应返回的实体的哪个部分符合范围请求。字段值以字节为单位，表示当前发送部分及整个实体大小。  
![](/assets/blogImgs/contentRange.jpg)

**Content-Location**  
Content-Location 表示的是报文主体返回资源对应的URI。  

**Expires**  
将资源失效的日期告知客户端。缓存服务器在接收到含有首部字段Expires的响应后，会以缓存来应答请求，在Expires字段值指定的时间之前，响应的副本会一直被保存。当超过指定的时间后，缓存服务器在请求发送过来时，会转向源服务器请求资源。  
源服务器不希望缓存服务器对资源缓存时，最好在 Expires 字段内写入与首部字段 Date 相同的时间值。  
当首部字段Cache-Control有指定 max-age 指令时，比起首部字段 Expires，会优先处理 max-age 指令。  
`Expires: Wed, 04 Jul 2012 08:26:05 GMT`   

#### Cookie首部字段（RCookie Fields）  
**Set-Cookie**  
当服务器准备开始管理客户端的状态时，会事先告知各种信息。以下是Set-Cookie 的字段值。
![](/assets/blogImgs/setCookie.jpg)

**Cookie**  
首部字段 Cookie 会告知服务器，当客户端想获得 HTTP 状态管理支
持时，就会在请求中包含从服务器接收到的 Cookie。接收到多个
Cookie 时，同样可以以多个 Cookie 形式发送。

### 认证
HTTP/1.1使用的认证方式有：
* BASIC 认证（基本认证）  
* DIGEST 认证（摘要认证）  
* SSL 客户端认证  
* FormBase 认证（基于表单认证）  

#### Basic认证  
basic认证就是客户端把`username:passwd`(用户ID和密码用:分隔)字符串，使用Base64编码，写入到请求首部字段Authorization。服务端验证通过返回包含Request-URI资源的响应。  
缺点是用户名密码都是明文，且无法实现注销操作，基本无人使用。  
![](/assets/blogImgs/basic.jpg)

#### Digest认证
digest认证和basic的区别就是不直接明文传输密码。而是依据401响应首部中WWW-Authenticate字段值nounce随机数（经过Basic64编码的十六进制数），和密码放在一起做MD5运算生成摘要。把摘要和用户名以明文方式发送请求给客户端。  
虽然避免密码泄漏，但是摘要容易被中间人劫持冒充，很少使用。    
![](/assets/blogImgs/digest.jpg)

#### SSL客户端认证
需要使用证书，且一般结合表单认证使用。证书用于认证客户端（保护通信过程），用户名密码用于认证用户本人。浏览器一般都预先放置了各大网站（bing、ali等等）的证书，所以对于别的网站如网银、12306都要求用户在使用前安装相关证书。  

#### FormBase认证（使用最多）
与Basic的区别就是用户名密码在传输过程中有HTTPS加密，且使用了Session ID作为Cookie保存了用户状态。  
步骤 1：客户端把用户ID和密码等登录信息放入报文的实体部分，以POST方法把请求发送给服务器。使用HTTPS通信来进行HTML表单画面的显示和用户输入数据的发送。  
步骤 2：服务器发放识别用户的SessionID（SessionID与用户存在一一映射）。通过验证从客户端发送过来的登录信息进行身份认证，然后把用户的认证状态与Session ID 绑定后记录在服务器端。向客户端返回响应时，会在首部字段 Set-Cookie 内写入 SessionID（如 PHPSESSID=028a8c…）。Session ID是服务端识别用户认证状态的关键，应使用难以推测的字符串，服务器端也需要进行有效期的管理，并在Cookie内加上 httponly 属性。  
步骤 3： 客户端接收到从服务器端发来的 Session ID 后，会将其作为Cookie保存在本地。下次向服务器发送请求时，浏览器会自动发送Cookie，Session ID 也随之发送到服务器。服务器端验证Session ID识别用户和其认证状态。  
![](/assets/blogImgs/formBase.jpg)