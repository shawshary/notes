---
    title: HTTP Notes
    author: shawshary
    date: 2022-11-13
---

\newpage

# Keywords #

+------+--------------------------------------------------------------------+
| Name | Description                                                        |
+======+====================================================================+
| web  | web browser, web page, web server                                  |
+------+--------------------------------------------------------------------+
| URI  | Uniform Resource Identifier, 由某个协议方案表示的资源的定位标识符. |
+------+--------------------------------------------------------------------+
| URL  | Uniform Resource Locator                                           |
+------+--------------------------------------------------------------------+
| SGML | Standard Generalized Markup Language                               |
+------+--------------------------------------------------------------------+
| HTML | HyperText Markup Language                                          |
+------+--------------------------------------------------------------------+
| http |                                                                    |
+------+--------------------------------------------------------------------+
| CGI  | Common Gateway Interface, 通用网关接口                             |
+------+--------------------------------------------------------------------+
| REST | REpresentational State Transfer, 表征状态转移                      |
+------+--------------------------------------------------------------------+


## URI ##

![绝对URI格式](https://raw.githubusercontent.com/rainvestige/PicGo/master/2022/11/13/eaf0.png)

URI Format: `http://user:pass@www.example.cn:80/dir/index.htm?uid=1#ch1`

+------------+-----------+----------------+------+---------------+-------+------------+
| http:      | user:pass | www.example.cn | 80   | dir/index.htm | uid=1 | ch1        |
+============+===========+================+======+===============+=======+============+
| 协议方案名 | 认证      | 服务器地址     | 端口 | 文件路径      | 参数  | 片段标识符 |
+------------+-----------+----------------+------+---------------+-------+------------+


# HTTP 1.1 #

HTTP协议规定, 请求从客户端发出, 最后服务端响应该请求并返回.


1. 请求结构:

    ```txt
    Method request-URI HTTP-Version
    header-field

    entity-body
    ```

2. 响应结构:

    ```txt
    HTTP-Version status-code reason-phrase
    header-field

    entity-body
    ```

**HTTP是不保存状态的协议**, 即不具备保存之前发送过的请求或响应的功能.
但是有需求需要能保持登录状态, 否则处理业务会棘手. 于是引入了**Cookie**技术.


## 持久连接 ##
只要任意一端没有明确提出断开连接, 则保持TCP连接状态. (keep-alive, connection
reuse, persistent connection) 在HTTP/1.1中, 所有连接默认都是持久连接, 但是在
HTTP/1.0内并未标准化. 客户端也需要支持持久连接.

持久连接使得多数请求以管线化(pipelining)方式发送成为可能. 可以连续发送多个请求,
不需要一个接一个的等该响应.


## Cookie ##
HTTP是无状态协议, 它不对之前发生过的请求和响应的状态进行管理. 也就是说, 无法
根据之前的状态进行本次的请求处理.

如果让服务器管理全部客户端状态则会成为负担. Cookie技术通过在请求和响应报文中
写入Cookie信息来控制客户端状态.

Cookie会根据从 **从服务器端发送的响应报文** 内的一个叫做`Set-Cookie`的首部字段,
通知客户端保存Cookie. 当下次客户端再往服务器发送请求时, 客户端会自动在请求报文
中加入Cookie后发送出去.

服务器端发现客户端发送过来的Cookie后, 会去检查究竟是从哪一个客户端发来的连接请
求, 然后对比服务器上的记录, 最后得到之前的状态信息.


# HTTP报文内的信息 #

首部字段一般有四个: 通用首部, 请求首部, 响应首部和实体首部.


## 报文主体和实体主体的差异 ##
1. 报文(message)是HTTP通信中的基本单位, 由8位组字节流(octet sequence,
   其中octet为8个比特)组成, 通过HTTP通信传输.

2. 实体(entity)作为请求或响应的有效载荷数据(补充项)被传输,
   其内容由实体首部和实体主体组成.

HTTP报文的主体用于传输请求或响应的实体主体. **通常报文主体等于实体主体**.
只有当传输中进行编码操作时, 实体主体的内容发生变化, 才导致它和报文主体产生差异.


## 压缩传输的内容编码 ##
HTTP协议中有一种被称为内容编码的操作, 其指明应用在**实体**内容上的编码格式,
并保持实体信息无损压缩. 内容编码后的实体由客户端接收并负责解码.

常用的内容编码有一下几种:
- gzip (GNU zip)
- compress (UNIX系统的标准压缩)
- deflate (zlib)
- identity (不进行编码)


## 分割发送的分块传输编码 ##
在HTTP通信过程中, 请求的编码实体资源尚未全部传输完成之前, 浏览器无法显示请求页
面. 在传输大容量数据时, 通过把数据分割成多块, 能够让浏览器逐步显示页面.
这种把实体主体分块的功能称为**分块传输编码 (Chunked Transfer Coding)**.

分块传输编码会将实体主体分成多个部分(块), 每一块都会用**16进制**来标记块的大小,
而实体主体的最后一块会使用**0(CR+LF)**来标记.

HTTP/1.1中存在一种称为传输编码(Transfer Coding)的机制, 它可以在通信时按照某种
编码方式传输, 但只定义作用于分块传输编码中.


## 多种数据的多部份对象集合 ##
HTTP协议采纳了多部份对象集合(Multipart)的方法, 来容纳多份不同类型的数据, 发送的
一份报文主体内可含有多类型实体. 通常是在图片或文本文件等上传时使用.

多部份对象集合包含的对象如下:

- multipart/form-data: 在web表单文件上传时使用.
- multipart/byteranges: 状态码206 (Partial Content)响应报文包含了多个范围的内容
  时使用.


## 获取部分内容的范围请求 ##
可以请求指定实体的部分(范围)内容, 指定范围发送的请求叫做**范围请求 (Range
Request)**. 对于一份10000字节大小的资源, 如果使用范围请求, 可以指请求5001-10000
字节内的资源.

```txt
HTTP/1.1 206 Partial Content
Date: ...
Content-Range: bytes -3000, 5000-7000
Content-Length:
Content-Type: image/jepg
```

如果服务器端无法响应范围请求, 则会返回状态码200 OK和完整的实体内容.


## 内容协商 ##
当浏览器的默认语言为英语或中文, 访问相同URI的web页面时, 则会显示对应的英语版或
中文版的web页面. 这样的机制称为内容协商 (Content Negotiation).

内容协商会以响应资源的语言, 字符集, 编码方式等作为判断的基准.



# 状态码 #

状态码的类别

+-----+---------------------------------+----------------------------+
|     | 类别                            | 原因短语                   |
+=====+=================================+============================+
| 1XX | Informational (信息性状态码)    | 接受的请求正在处理         |
+-----+---------------------------------+----------------------------+
| 2XX | Success (成功状态码)            | 请求正常处理完毕           |
+-----+---------------------------------+----------------------------+
| 3XX | Redirection (重定向状态码)      | 需要进行附加操作以完成请求 |
+-----+---------------------------------+----------------------------+
| 4XX | Client Error (客户端错误状态码) | 服务器无法处理请求         |
+-----+---------------------------------+----------------------------+
| 5XX | Server Error (服务器错误状态码) | 服务器处理请求出错         |
+-----+---------------------------------+----------------------------+

+--------+-------------------+------------------------------------------------+
| 状态码 | 原因短句          | 意思                                           |
+========+===================+================================================+
| 200    | OK                | 客户端的请求在服务器端被正常处理了             |
+--------+-------------------+------------------------------------------------+
| 204    | No Content        | 请求成功处理, 但返回的响应报文中不含实体的主体 |
|        |                   | 部分                                           |
+--------+-------------------+------------------------------------------------+
| 206    | Partial Content   | 客户端进行了范围请求, Content-Range            |
+--------+-------------------+------------------------------------------------+
| 301    | Moved Permanently | 永久性重定向. 请求的资源已被分配了新的URI.     |
+--------+-------------------+------------------------------------------------+
| 302    | Found             | 临时性重定向. 请求的资源临时变更, 以后可能还会 |
|        |                   | 发生改变.                                      |
+--------+-------------------+------------------------------------------------+
| 303    | See Other         | 和302相同, 但明确表示客户端需要采用GET方法获取 |
|        |                   | 资源.                                          |
+--------+-------------------+------------------------------------------------+
| 304    | Not Modified      | 当客户端发送附带条件的请求时, 服务器端允许请求 |
|        |                   | 访问资源, 但未满足条件的情况.                  |
+--------+-------------------+------------------------------------------------+
| 307    | Temporary         | 临时性重定向.                                  |
|        | Redirect          |                                                |
+--------+-------------------+------------------------------------------------+
| 400    | Bad Request       | 请求报文中存在语法错误                         |
+--------+-------------------+------------------------------------------------+
| 401    | Unauthorized      | 请求需要有HTTP认证(BASIC认证, DIGEST认证)的认  |
|        |                   | 证信息.                                        |
+--------+-------------------+------------------------------------------------+
| 403    | Forbidden         | 服务器拒绝请求                                 |
+--------+-------------------+------------------------------------------------+
| 404    | Not Found         | 没有请求的资源                                 |
+--------+-------------------+------------------------------------------------+
| 500    | Internal Server   | 服务器端执行请求时发生了错误                   |
|        | Error             |                                                |
+--------+-------------------+------------------------------------------------+
| 503    | Service           | 服务器暂时处于超负荷或正在进行停机维护, 暂时无 |
|        | Unavailable       | 法处理请求                                     |
+--------+-------------------+------------------------------------------------+


# Web Server #


## 单台虚拟主机实现多个域名 ##

在相同的IP地址下, 由于虚拟主机可以寄存多个不同主机名和域名的Web网站,
因此在发送HTTP请求时, 必须在Host首部内完整指定主机名或域名的URI.


## 通信数据转发程序: 代理, 网关, 隧道 ##

通信数据转发程序和服务器可以将请求转发给通信线路上的下一站服务器,
并且能接收从那台服务器发送的响应再转发给客户端.

- 代理
    代理是一种有转发功能的应用程序, 它扮演了位于服务器和客户端"中间人"的角色,
    接收由客户端发送的请求并转发给服务器, 同时也接收服务器返回的响应并转发给客
    户端.

- 网关
    网关是转发其他服务器通信数据的服务器, 接收从客户端发送来的请求时,
    他就像自己拥有资源的源服务器一样对请求进行处理. 有时客户端可能都不会察觉,
    自己的通信目标是一个网关.

- 隧道
    隧道是在**相隔甚远的**客户端和服务器之间进行中转, 并保持双方通信连接的应用
    程序.


### 代理 ###
使用代理的理由有: 利用缓存技术减少网络带宽的流量, 组织内部针对特定网站的访问
控制等等.

![代理追加Via首部](https://raw.githubusercontent.com/rainvestige/PicGo/master/2022/11/17/0aa7.png)

![代理进行访问控制](https://raw.githubusercontent.com/rainvestige/PicGo/master/2022/11/17/add0.png)

代理有多种使用方法, 按两种基准分类. 一种是是否使用缓存, 另一种是是否会修改报文.

- 缓存代理
    代理转发响应时, 缓存代理(Caching Proxy)会预先将资源的副本(缓存)保存再代理
    服务器上.

    当代理再次接收到对同一资源的请求时, 可以直接将之前的缓存资源作为响应返回.

- 透明代理
    转发请求或响应时, **不对报文做任何加工**的代理(Transparent Proxy). 反之,
    做修改的是非透明代理.


### 网关 ###

![网关利用其他协议](https://raw.githubusercontent.com/rainvestige/PicGo/master/2022/11/17/14b4.png)

网关的工作机制和代理十分相似. 而网关能使通信线路上的服务器提供非HTTP协议服务.

利用网关能**提供通信的安全性**, 因为可以在客户端与网关之间的通信线路上加密以
确保连接的安全。例如, 网关可以连接数据库, 使用SQL语句查询数据. 另外, 在Web购物
网站上进行信用卡结算时, 网关可以和信用卡结算系统联动.


### 隧道 ###
隧道使用SSL等加密手段进行通信, 目的时建立安全的通信. 其本身不会解析HTTP请求.
在通信双方断开连接时隧道结束.


## 缓存 ##
缓存是指代理服务器或者客户端本地磁盘内保存的资源副本. 缓存有有效期限. 客户端的
缓存被称为临时网络文件(Temporary Internet File).


# HTTP首部 #
HTTP报文结构由报文首部, 空行, 报文主体构成. HTTP协议的请求和响应报文必定包含
HTTP首部.

- 请求报文首部由请求行(方法 URI HTTP版本), HTTP首部字段(请求首部字段,
  通用首部字段, 实体首部字段)组成.
- 响应报文首部由状态行(HTTP版本 状态码数字 状态码短句), HTTP首部字段(响应首部
  字段, 通用首部字段, 实体首部字段)组成.


## HTTP首部字段结构 ##
> 字段名: 字段值(值可以有多个)

例子,

```txt
    Content-Type: text/html
    Keep-Alive: timeout=15, max=100
```
: 通用首部字段 {#tbl:table1}

| 首部字段名        | 说明                       |
| :------           | :------                    |
| Cache-Control     | 控制缓存的行为             |
| Connection        | 逐跳首部? 连接的管理       |
| Date              | 创建报文的日期             |
| Pragma            | 报文指令                   |
| Trailer           | 报文末端的首部一览         |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade           | 升级为其他协议             |
| Via               | 代理服务器的相关信息       |
| Warning           | 错误通知                   |
