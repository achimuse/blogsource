---
title: http协议小记
date: 2018-03-26 17:03:56
tags:
	- web
---
### 概述
当前http协议主流版本是HTTP/1.1, 预计不久将来会使用HTTP/2.0  
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
### 基本概念
#### 被动性  
http协议在一条通路上必然有一端是客户端另一端是服务端，而且只能由客户端发出请求，服务端响应该请求。  
#### 常用方法
http请求的根本目标是操作资源，所以请求中使用URI定位资源，使用方法表明意图(restful)：  
- GET方法，请求资源，查  
- POST方法，推送资源，增  
- PUT方法，改动资源，改  
- DELETE方法，删除资源，删
- HEAD方法，同GET，响应不返回报文主体，用于确认URI有效性和资源更新时间  
#### 持久连接与管道化
持久连接（HTTP Persistent Connections），建立TCP连接后只要任一端没有提出断开连接，则保持连接状态，提高请求相应速度。在同一TCP连接上会有多有HTTP连接，同时线管化（Pipelining）方式使得不用等待上个请求的相应就可以发送下一个请求。  
#### 无状态与Cookie
stateless，不会保存请求或相应的状态，优点在于减少服务器负担。如果需要状态则使用Cookie。客户端第一次向服务端发出请求（要求Cookie），服务端生成Cookie并记录下Cookie与客户端的映射，在响应返回Cookie,客户端收到后保存。之后的每个请求都带上Cookie，即保存了状态。Cookie就是一个字符id。  
#### 报文结构
http报文实际是字符串文本，用CR+LF空行划分报文的首部和主体。报文分为请求报文和响应报文。请求行包含请求方法、URI、HTTP版本，状态行包含状态吗、原因短语、HTTP版本。  
![](/assets/blogImgs/httpPacket.jpg)  
![](/assets/blogImgs/httpPacketReal.jpg)  
#### 传输
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
以下是一个真实的post请求，使用了表单上传.xlxs表格  
![请求头部](/assets/blogImgs/formRealHead.jpg)
![请求主体](/assets/blogImgs/formRealEntity.jpg) 

* multipart/byteranges  
范围请求（Range Request）指定请求实体范围，实现了了下载中断恢复功能。    
在请求首部加上字段`Content-Range: bytes 7000-7999/8000`,返回状态码206（Partial Content）的响应报文。响应报文首部会有字段`Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES`。  
返回的实体都会有`--THIS_STRING_SEPARATES`作为分界线，最后以`--THIS_STRING_SEPARATES--`标记结束。
![响应首部](/assets/blogImgs/byterangesHead.jpg)
![响应主体](/assets/blogImgs/byterangesEntity.jpg) 
#### 内容协商（Content Negotiation）
客户端会与服务端协商请求最适合的资源，主要在与资源的语言、字符集、编码方式。故请求的首部字段会有`Accept`/`Accept-Charset`/`Accept-Encoding`/`Accept-Language`/`Content_Language`等协商字段。
#### 状态码
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
`403 Forbidden`：拒绝该请求访问。  
`404 Not Found`：服务端无请求的资源。    
**5XX 服务端错误**  
`500 Internet Server Error`：执行请求出错，服务端内部错误。  
`503 Service Unavailable`：服务端出于停机维护状态。  

