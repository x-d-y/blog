---
title: http原理解析
date: 2019-12-03 11:12:03
tags:
---

&#160;&#160;&#160;&#160;&#160;&#160;HTTP 协议（HyperText Transfer Protocol，超文本传输协议）是因特网上应用最为广泛的一种网络传输协议，所有的 WWW 文件都必须遵守这个标准。HTTP 是一个基于 TCP/IP 通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。

### HTTP 注意事项

- HTTP 是无连接，即只有当发送请求时客户端才会和服务端进行连接，当服务端处理完请求后返回完成断开连接。

- HTTP 是媒体独立的，任何类型的数据都可以通过 HTTP 协议进行传输，只要双方约定好如何处理该类型的数据即可正常工作

- HTTP 是无装态的，如果一次请求要用的上一次请求的数据，只能重新传输上一次的数据，本次请求不记录上次请求的状态
  <!--more-->

### HTTP 协议是基于 TCP/IP 模型

&#160;&#160;&#160;&#160;&#160;&#160;HTTP 协议是基于 TCP/IP 协议，并且是属于应用层的协议，其默认端口是 80。
（图）

### HTTP 消息组成

#### 客户端

&#160;&#160;&#160;&#160;&#160;&#160;客户端发送的请求中，消息被分为四部分即：请求行、请求头部、空行、请求数据
（图）

#### 服务端

&#160;&#160;&#160;&#160;&#160;&#160;服务端在接收到请求后，进行处理，处理完成后按照如下格式进行响应:状态行、消息报头、空行、响应正文
（图）

### HTTP 请求方法

HTTP 协议共支持九种请求方法，除了开发时常用的 post、put、get、delete 外还有 head、connect、options、trace、patch

- GET ｜ 获取指定的页面的信息
- HEAD ｜和 GET 一样，只不过请求的内容中没有返回数据，只有报头
- POST ｜ 发送上传信息的请求，信息内容不是写在 url 中，而是在单独的请求数据模块中
- PUT ｜ 发送上传信息请求，指定需要被上传信息替换的更改数据，完成该数据的信息更改
- DELETE ｜ 指定需要删除的数据完成该数据的删除
- CONNECT ｜ 预留
- OPTIONS ｜ 允许客户端查看服务器的性能
- TRACE ｜ 显示服务器收到的请求，一般用于问题诊断
- PATCH ｜ 和 PUT 作用一样，只不过是对局部信息更改，并不是对全部信息进行更改

### HTTP 头响应信息

|      应答头      |                                       说明                                       |
| :--------------: | :------------------------------------------------------------------------------: |
|      Allow       |                               服务器支持的请求方法                               |
| Content-Encoding | 文档的编码方法，需要按照编码的方法解码后才能获得 content-type 内指定的类型的信息 |
|  Content-Length  |                                  表示内容的长度                                  |
|       Date       |                                     当前时间                                     |
|     Expires      |                         文档缓存时间，超过时间将不再缓存                         |
|  Last-Modified   |                                 文档最后改动时间                                 |
|     Location     |                                   文档提取位置                                   |
|     Refresh      |                                   射中刷新时间                                   |
|      Server      |                                   服务器的名字                                   |
|    Set-Cookie    |                              设置页面关联的 Cookie                               |
| WWW-Authenticate |                               授权方式以及授权信息                               |

### HTTP 请求响应状态码

| 分类  |                    分类描述                    |
| :---: | :--------------------------------------------: |
| 1\*\* | 信息，服务器接收到请求，需要请求者继续执行请求 |
| 2\*\* |                      成功                      |
| 3\*\* |        重定向，需要进一步的操作完成请求        |
| 4\*\* |                   客户端错误                   |
| 5\*\* |                   服务器错误                   |

### HTTP coontent-type

Content-type 标头告诉客户端实际返回的内容以及内容类型，常见的内容如下：

- text/html ： HTML 格式
- text/plain ：纯文本格式
- text/xml ： XML 格式
- image/gif ：gif 图片格式
- image/jpeg ：jpg 图片格式
- image/png：png 图片格式
- application/xhtml+xml ：XHTML 格式
- application/xml： XML 数据格式
- application/atom+xml ：Atom XML 聚合格式
- application/json： JSON 数据格式
- application/pdf：pdf 格式
- application/msword ： Word 文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： <form encType=””>中默认的 encType，form 表单数据被编码为 key/value 格式发送到服务器（表单默认的提交数据的格式）
- multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式
