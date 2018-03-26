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
### 简单的理解
http协议在一条通路上必然有一端是客户端另一端是服务端，而且只能由客户端发出请求，服务端响应该请求。  

http请求的根本目标是操作资源，所以请求中使用URI定位资源，使用方法表明意图(restful)：  
- GET方法，请求资源，查  
- POST方法，推送资源，增  
- PUT方法，改动资源，改  
- DELETE方法，删除资源，删
- HEAD方法，同GET，响应不返回报文主体，用于确认URI有效性和资源更新时间  

持久连接（HTTP Persistent Connections），建立TCP连接后只要任一端没有提出断开连接，则保持连接状态，提高请求相应速度。在同一TCP连接上会有多有HTTP连接，同时线管化（Pipelining）方式使得不用等待上个请求的相应就可以发送下一个请求。  

stateless，不会保存请求或相应的状态，优点在于减少服务器负担。如果需要状态则使用Cookie。客户端第一次向服务端发出请求（要求Cookie），服务端生成Cookie并记录下Cookie与客户端的映射，在响应返回Cookie,客户端收到后保存。之后的每个请求都带上Cookie，即保存了状态。Cookie就是一个字符id。  

