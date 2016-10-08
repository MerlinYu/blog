#http
转载文章：http://www.iteye.com/topic/1125771<br>
转载文章：http://blog.csdn.net/hguisu/article/details/8680808<br>
转载文章：http://www.cnblogs.com/phphuaibei/archive/2011/09/27/2192817.html<br>
##简介
互联网的r关键技术就是TCP/IP协议，两台s计sm算机r之间的通信是通过TCP/IP协议在因特网上进行的。实际上是两个协议：
* TCP：Transmission Control Protocol 传输控制协议<br>
* IP: Internet Protocol  网际协议<br>

####IP：计算机之间的通信
IP协议是计算机用来相互识别的通信的一种机制，每台计算机都有一个IP.用来在internet上标识这台计算机。  IP 负责在因特网上发送和接收数据包。
通过 IP，消息（或者其他数据）被分割为小的独立的包，并通过因特网在计算机之间传送。IP 负责将每个包路由至它的目的地。<br>
IP协议仅仅是允许计算机相互发消息，但它并不检查消息是否以发送的次序到达而且没有损坏（只检查关键的头数据）。为了提供消息检验功能，直接在IP协议上设计了传输
控制协议TCP.

####TCP : 应用程序之间的通信
TCP确保数据包以正确的次序到达，并且尝试确认数据包的内容没有改变。TCP在IP地址之上引端口（port），它允许计算机通过网络提供各种服务。一些端口号为不同的服务
保留，而且这些端口号是众所周知。<br>
服务或者守护进程：在提供服务的机器上，有程序监听特定端口上的通信流。例如大多数电子邮件通信流出现在端口25上，用于wwww的HTTP通信流出现在80端口上。
当应用程序希望通过 TCP 与另一个应用程序通信时，它会发送一个通信请求。这个请求必须被送到一个确切的地址。在双方“握手”之后，TCP 将在两个应用程序之间建立一
个全双工 (full-duplex) 的通信，占用两个计算机之间整个的通信线路。TCP 用于从应用程序到网络的数据传输控制。TCP 负责在数据传送之前将它们分割为 IP 包，
然后在它们到达的时候将它们重组。<br>
TCP/IP 就是TCP 和 IP 两个协议在一起协同工作，有上下层次的关系。<br>
TCP 负责应用软件（比如你的浏览器）和网络软件之间的通信。IP 负责计算机之间的通信。TCP 负责将数据分割并装入 IP 包，IP 负责将包发送至接受者，传输过程要经
IP路由器负责根据通信量、网络中的错误或者其他参数来进行正确地寻址，然后在它们到达的时候重新组合它们。
##Http
一次HTTP操作称为一个事务，其工作整个过程如下：<br>
####1. 地址解析<br>
如用客户端浏览器请求这个页面：http://localhost.com:8080/index.htm
从中分解出协议名、主机名、端口、对象路径等部分，对于我们的这个地址，解析得到的结果如下：<br>
协议名：http<br>
主机名：localhost.com<br>
端口：8080<br>
对象路径：/index.htm<br>
在这一步，需要域名系统DNS解析域名localhost.com,得主机的IP地址。<br>

####2. 封装HTTP请求数据包
把以上部分结合本机自己的信息，封装成一个HTTP请求数据包<br>

####3. 封装成TCP包，建立TCP连接（TCP的三次握手）
在HTTP工作开始之前，客户机（Web浏览器）首先要通过网络与服务器建立连接，该连接是通过TCP来完成的，该协议与IP协议共同构建Internet，即著名的TCP/IP协议族，
因此Internet又被称作是TCP/IP网络。<br>
HTTP是比TCP更高层次的应用层协议，根据规则，只有低层协议建立之后才能，才能进行更层协议的连接，因此，首先要建立TCP连接，一般TCP连接的端口号是80。
这里是8080端口

####4. 客户机发送请求命令
建立连接后，客户机发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息包括请求修饰符、客户机信息和可内容。

####5. 服务器响应
服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息包括服务器信息、实体信息和可能的内容。<br>
实体消息是服务器向浏览器发送头信息后，它会发送一个空白行来表示头信息的发送到此为结束，接着，它就以Content-Type应答头信息所描述的格式发送用户所请求的实际数据.

####6. 服务器关闭TCP连接
一般情况下，一旦Web服务器向浏览器发送了请求数据，它就要关闭TCP连接，然后如果浏览器或者服务器在其头信息加入了这行代码<br>
```java
    Connection:keep-alive
```
TCP连接在发送后将仍然保持打开状态，于是，浏览器可以继续通过相同的连接发送请求。保持连接节省了为每个请求建立新连接所需的时间，还节约了网络带宽。<br>
客户机发起一次请求的时候：<br>
客户机会将请求封装成http数据包-->封装成Tcp数据包-->封装成Ip数据包--->封装成数据帧--->硬件将帧数据转换成bit流（二进制数据）-->最后通过物理硬件（网卡芯片）发送到指定地点。<br>
服务器硬件首先收到bit流....... 然后转换成ip数据包。于是通过ip协议解析Ip数据包，然后又发现里面是tcp数据包，就通过tcp协议解析Tcp数据包，
接着发现是http数据包通过http协议再解析http数据包得到数据。
##Https
HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安
全基础是SSL。其所用的端口号是443。<br>
SSL：安全套接层，是netscape公司设计的主要用于web的安全传输协议。这种协议在WEB上获得了广泛的应用。通过证书认证来确保客户端和网站服务器之间的通信数据是加密安全的。<br>
有两种基本的加解密算法类型：<br>
####1. 对称加密（symmetrcic encryption）
密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的对称加密算法有DES、AES，RC5，3DES等；<br>
对称加密主要问题是共享秘钥，除你的计算机（客户端）知道另外一台计算机（服务器）的私钥秘钥，否则无法对通信流进行加密解密。解决这个问题的方案非对称秘钥。
####2. 非对称加密
使用两个秘钥：公共秘钥和私有秘钥。私有秘钥由一方密码保存（一般是服务器保存），另一方任何人都可以获得公共秘钥。<br>
这种密钥成对出现（且根据公钥无法推知私钥，根据私钥也无法推知公钥），加密解密使用不同密钥（公钥加密需要私钥解密，私钥加密需要公钥解密），相对对称加密速度较慢，
典型的非对称加密算法有RSA、DSA等。<br>
####https的通信过程：<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/https.jpg)<br>
####https通信的优点：<br>
1）客户端产生的密钥只有客户端和服务器端能得到；<br>
2）加密的数据只有客户端和服务器端才能得到明文；<br>
3）客户端到服务端的通信是安全的。<br>

##http请求格式
http的请求头由一个域名，冒号（:）和域值三部分组成。域名是大小写无关的，域值前可以添加任何数量的空格符，头域可以被扩展为多行，在每行开始处，
使用至少一个空格或制表符。<br>
HTTP最常见的请求头如下：<br>
####1. Transport 头域
* Connection：作用：表示是否需要持久连接。<br>
如果服务器看到这里的值为“Keep-Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接）,它就可以利用持久连接的优点，当页面包含多个元素时（例如Applet，图片），显著地减少下载所需要的时间。要实现这一点，服务器需要在应答中发送一个Content-Length头，最简单的实现方法是：先把内容写入 ByteArrayOutputStream，然后在正式写出内容之前计算它的大小；

例如：　Connection: keep-alive   当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的  网页，会继续使用这一条已经建立的连接

例如：  Connection: close  代表一个Request完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭，  当客户端再次发送Request，需要重新建立TCP连接。

* Host（发送请求时，该报头域是必需的）<br>
Host请求报头域主要用于指定被请求资源的Internet主机和端口号，它通常从HTTP URL中提取出来的。
eg：http://；localhost/index.html
浏览器发送的请求消息中，就会包含Host请求报头域，如下：
Host：localhost
此处使用缺省端口号80，若指定了端口号8080，则变成：Host：localhost:8080

####2. Client 头域
* Accept：
作用：浏览器可以接受的媒体类型（MIME类型）,
例如：  Accept: text/html  代表浏览器可以接受服务器回发的类型为 text/html  也就是我们常说的html文档, 如果服务器无法返回text/html类型的数据，服务器
应该返回一个406错误(non acceptable)。<br>
通配符 * 代表任意类型。例如  Accept: */*  代表浏览器可以处理所有类型，(一般浏览器发给服务器都是发这个)<br>
* Accept-Encoding：
作用： 浏览器申明自己接收的编码方法，通常指定压缩方法，是否支持压缩，支持什么压缩方法（gzip，deflate），（注意：这不是只字符编码）;<br>

例如： Accept-Encoding: gzip, deflate。Server能够向支持gzip/deflate的浏览器返回经gzip或者deflate编码的HTML页面。 许多情形下这可以减少5到10倍的下载时间，也节省带宽。

* Accept-Language：作用： 浏览器申明自己接收的语言。<br>
语言跟字符集的区别：中文是语言，中文有多种字符集，比如big5，gb2312，gbk等等；<br>
例如： Accept-Language:zh-cn 。如果请求消息中没有设置这个报头域，服务器假定客户端对各种语言都可以接受。<br>

* User-Agent：作用：告诉HTTP服务器， 客户端使用的操作系统和浏览器的名称和版本.<br>
我们上网登陆论坛的时候，往往会看到一些欢迎信息，其中列出了你的操作系统的名称和版本，你所使用的浏览器的名称和版本，这往往让很多人感到很神奇，实际上，
服务器应用程序就是从User-Agent这个请求报头域中获取到这些信息User-Agent请求报头域允许客户端将它的操作系统、浏览器和其它属性告诉服务器。<br>
例如： User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; CIBA; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; .NET4.0C; InfoPath.2; .NET4.0E)

* Accept-Charset：作用：浏览器申明自己接收的字符集。<br>
这就是本文前面介绍的各种字符集和字符编码，如gb2312，utf-8（通常我们说Charset包括了相应的字符编码方案）；<br>
例如：Accept-Charset:iso-8859-1,gb2312.如果在请求消息中没有设置这个域，缺省是任何字符集都可以接受。<br>
Authorization：授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中；<br>
Authorization请求报头域主要用于证明客户端有权查看某个资源。当浏览器访问一个页面时，如果收到服务器的响应代码为401（未授权），可以发送一个包含Authorization请求报头域的请求，要求服务器对其进行验证。

####3. Cookie/Login 头域
* Cookie:作用： 最重要的header, 将cookie的值发送给HTTP 服务器

####4. Entity头域
* Content-Length作用：发送给HTTP服务器数据的长度。即请求消息正文的长度；
例如： Content-Length: 38<br>
* Content-Type：作用:接收类型<br>
例如：Content-Type: application/x-www-form-urlencoded<br>

####5. Miscellaneous 头域
* Referer:
作用： 提供了Request的上下文信息的服务器，告诉服务器我是从哪个链接过来的，比如从我主页上链接到一个朋友那里， 他的服务器就能够从HTTP Referer中统计出每天有多少用户点击我主页上的链接访问    他的网站。
例如: Referer:http://translate.google.cn/?hl=zh-cn&tab=wT<br>

####6. Cache 头域
* If-Modified-Since：
作用： 把浏览器端缓存页面的最后修改时间发送到服务器去，服务器会把这个时间与服务器上实际文件的最后修改时间进行对比。如果时间一致，那么返回304，客户端就直接使用本地缓存文件。如果时间不一致，就会返回200和新的文件内容。客户端接到之后，会丢弃旧文件，把新文件缓存起来，并显示在浏览器中。
例如：If-Modified-Since: Thu, 09 Feb 2012 09:07:57 GMT。
* If-None-Match：
作用: If-None-Match和ETag一起工作，工作原理是在HTTP Response中添加ETag信息。 当用户再次请求该资源时，将在HTTP Request 中加入If-None-Match信息(ETag的值)。如果服务器验证资源的ETag没有改变（该资源没有更新），将返回一个304状态告诉客户端使用本地缓存文件。否则将返回200状态和新的资源和Etag.  
使用这样的机制将提高网站的性能
例如: If-None-Match: "03f2b33c0bfcc1:0"
* Pragma：
作用： 防止页面被缓存， 在HTTP/1.1版本中，它和Cache-Control:no-cache作用一模一样Pargma只有一个用法， 例如： Pragma: no-cache
注意: 在HTTP/1.0版本中，只实现了Pragema:no-cache, 没有实现Cache-Control
* Cache-Control：作用: 这个是非常重要的规则。 这个用来指定Response-Request遵循的缓存机制。各个指令含义如下
* Cache-Control:Public   可以被任何缓存所缓存（）
* Cache-Control:Private     内容只缓存到私有缓存中
* Cache-Control:no-cache  所有内容都不会被缓存

###https响应格式

在接收和解释请求消息后，服务器会返回一个 HTTP 响应消息。与 HTTP 请求类似，HTTP 响应也是由三个部分组成，分别是：状态行、消息报头和响应正文。如：<br>
```bash
HTTP/1.1 200 OK
Date: Sun, 17 Mar 2013 08:12:54 GMT
Server: Apache/2.2.8 (Win32) PHP/5.2.5
X-Powered-By: PHP/5.2.5
Set-Cookie: PHPSESSID=c0huq7pdkmm5gg6osoe3mgjmm3; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Content-Length: 4393
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=utf-8
<html>
<head>
<title>HTTP响应示例<title>
</head>
<body>
Hello HTTP!
</body>
</html>
```
####1. 状态行
状态行由协议版本、数字形式的状态代码，及相应的状态描述组成，各元素之间以空格分隔，结尾时回车换行符，格式如下：<br>

HTTP-Version Status-Code Reason-Phrase CRLF

HTTP-Version 表示服务器 HTTP 协议的版本，Status-Code 表示服务器发回的响应代码，Reason-Phrase 表示状态代码的文本描述，CRLF 表示回车换行。例如：

HTTP/1.1 200 OK (CRLF)

      状态代码与状态描述

      状态代码由 3 位数字组成， 表示请求是否被理解或被满足，状态描述给出了关于状态码的简短的文字描述。状态码的第一个数字定义了响应类别，后面两位数字没有具体分类。第一个数字有 5 种取值，如下所示。

- 1xx：指示信息——表示请求已经接受，继续处理
- 2xx：成功——表示请求已经被成功接收、理解、接受。
- 3xx：重定向——要完成请求必须进行更进一步的操作
- 4xx：客户端错误——客户端请求有错误或请求无法实现
- 5xx：服务器端错误——服务器未能实现合法的请求。

常见状态代码、状态描述、说明：
200 OK      //客户端请求成功

303：重定向，即从原url重定向到新的url。 例如php 的hear函数header（"localtion:/index.php"）

400 Bad Request  //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用
403 Forbidden  //服务器收到请求，但是拒绝提供服务，一般是服务器路径没有权限或者是其他权限相关问题
404 Not Found  //请求资源不存在，eg：输入了错误的URL
500 Internal Server Error //服务器发生不可预期的错误：一般来说，这个问题都会在服务器端的源代码出现错误时出现，比如出现死循环。

502 Bad Gateway//作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。比如LNMP ,php-fpm没有启动就会报502错误。

503 Server Unavailable  //服务器当前不能处理客户端的请求，一段时间后可能恢复正常

504 Gateway Time-out：作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP、FTP、LDAP）或者辅助服务器（例如DNS）收到响应，比如nginx和php-fpm， php设置sleep（200），就会收到504 Gateway Time-out。注意：某些代理服务器在DNS查询超时时会返回400或者500错误

####2. 响应正文

响应正文就是服务器返回的资源的内容，响应头和正文之间也必须用空行分隔。如：
```bash
[html] view plain copy

 print?

- <html>
- <head>
- <title>HTTP响应示例<title>
- </head>
- <body>
- Hello HTTP!
- </body>
- </html>
```
####3. 响应头信息
HTTP最常见的响应头如下所示：<br>
#####1. Cache头域
* Date：作用：生成消息的具体时间和日期，即当前的GMT时间。
例如：　Date: Sun, 17 Mar 2013 08:12:54 GMT
* Expires：作用: 浏览器会在指定过期时间内使用本地缓存，指明应该在什么时候认为文档已经过期，从而不再缓存它。
例如: Expires: Thu, 19 Nov 1981 08:52:00 GMT
* Vary例如: Vary: Accept-Encoding

#####2. Cookie/Login 头域

* P3P 作用: 用于跨域设置Cookie, 这样可以解决iframe跨域访问cookie的问题
例如: P3P: CP=CURa ADMa DEVa PSAo PSDo OUR BUS UNI PUR INT DEM STA PRE COM NAV OTC NOI DSP COR
* Set-Cookie 作用： 非常重要的header, 用于把cookie 发送到客户端浏览器， 每一个写入cookie都会生成一个Set-Cookie.
例如: Set-Cookie: PHPSESSID=c0huq7pdkmm5gg6osoe3mgjmm3; path=/
#####3. Entity实体头域：
实体内容的属性，包括实体信息类型，长度，压缩方法，最后一次修改时间，数据有效性等。<br>
* ETag：作用:  和If-None-Match 配合使用。 （实例请看上节中If-None-Match的实例）
例如: ETag: "03f2b33c0bfcc1:0"
* Last-Modified：作用： 用于指示资源的最后修改日期和时间。（实例请看上节的If-Modified-Since的实例）
例如: Last-Modified: Wed, 21 Dec 2011 09:09:10 GMT
* Content-Type：
作用：WEB服务器告诉浏览器自己响应的对象的类型和字符集,
例如:
```bash
    Content-Type: text/html; charset=utf-8

　　Content-Type:text/html;charset=GB2312

　　Content-Type: image/jpeg
```
* Content-Length：
指明实体正文的长度，以字节方式存储的十进制数字来表示。在数据下行的过程中，Content-Length的方式要预先在服务器中缓存所有数据，然后所有数据再一股脑儿地发给客户端。
例如: Content-Length: 19847
* Content-Encoding：作用：文档的编码（Encode）方法。一般是压缩方式。
WEB服务器表明自己使用了什么压缩方法（gzip，deflate）压缩响应中的对象。利用gzip压缩文档能够显著地减少HTML文档的下载时间。
例如：Content-Encoding：gzip
* Content-Language：作用： WEB服务器告诉浏览器自己响应的对象的语言者.例如： Content-Language:da

#####4. Miscellaneous 头域

* Server：作用：指明HTTP服务器的软件信息
例如:Apache/2.2.8 (Win32) PHP/5.2.5
* X-Powered-By：作用：表示网站是用什么技术开发的。例如： X-Powered-By: PHP/5.2.5

#####5. Transport头域
* Connection：
例如：　Connection: keep-alive   当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接
例如：  Connection: close  代表一个Request完成后，客户端和服务器之间用于传输HTTP数据的TCP连接会关闭， 当客户端再次发送Request，需要重新建立TCP连接。

#####6. Location头域
* Location：作用： 用于重定向一个新的位置， 包含新的URL地址.实例请看304状态实例<br>
HTTP协议是无状态的和Connection: keep-alive的区别<br>
无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。从另一方面讲，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。
HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（无连接）。<br>
从HTTP/1.1起，默认都开启了Keep-Alive，保持连接特性，简单地说，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接。
Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。<br>
