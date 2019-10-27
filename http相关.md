## 强缓存哪个属性？协商缓存哪个属性？
- 强缓存通过Expires和Cache-Control两种响应头实现
- 协商缓存是利用的是【Last-Modified，If-Modified-Since】和【ETag、If-None-Match】这两对Header来管理的

**强缓存**

***1、Expires***

Expires是http1.0提出的一个表示资源过期时间的header，它描述的是一个绝对时间，由服务器返回。
Expires 受限于本地时间，如果修改了本地时间，可能会造成缓存失效

```Expires: Wed, 11 May 2018 07:20:00 GMT```

***2、Cache-Control***

Cache-Control 出现于 HTTP / 1.1，优先级高于 Expires ,表示的是相对时间

```Cache-Control: max-age=315360000```

- Cache-Control: no-cache不会缓存数据到本地的说法是错误的，详情《HTTP权威指南》P182
0ae591a1-f6fa-4236-8588-9d9e300f8ca7

- Cache-Control: no-store才是真正的不缓存数据到本地
- Cache-Control: public可以被所有用户缓存（多用户共享），包括终端和CDN等中间代理服务器
- Cache-Control: private只能被终端浏览器缓存（而且是私有缓存），不允许中继缓存服务器进行缓存
![image](https://user-images.githubusercontent.com/25027560/38223493-c7ec919e-371d-11e8-8d72-8c6b0e4935a8.png)

**协商缓存**

当浏览器对某个资源的请求没有命中强缓存，就会发一个请求到服务器，验证协商缓存是否命中，如果协商缓存命中，请求响应返回的http状态为304并且会显示一个Not Modified的字符串

1、Last-Modified，If-Modified-Since
Last-Modified 表示本地文件最后修改日期，浏览器会在request header加上If-Modified-Since（上次返回的Last-Modified的值），询问服务器在该日期后资源是否有更新，有更新的话就会将新的资源发送回来

但是如果在本地打开缓存文件，就会造成 Last-Modified 被修改，所以在 HTTP / 1.1 出现了 ETag

2、ETag、If-None-Match
Etag就像一个指纹，资源变化都会导致ETag变化，跟最后修改时间没有关系，ETag可以保证每一个资源是唯一的

If-None-Match的header会将上次返回的Etag发送给服务器，询问该资源的Etag是否有更新，有变动就会发送新的资源回来
070b6284-e835-4470-ac6e-7e1892fab369

ETag的优先级比Last-Modified更高

具体为什么要用ETag，主要出于下面几种情况考虑：

- 一些文件也许会周期性的更改，但是他的内容并不改变(仅仅改变的修改时间)，这个时候我们并不希望客户端认为这个文件被修改了，而重新GET；
- 某些文件修改非常频繁，比如在秒以下的时间内进行修改，(比方说1s内修改了N次)，If-Modified-Since能检查到的粒度是s级的，这种修改无法判断(或者说UNIX记录MTIME只能精确到秒)；
- 某些服务器不能精确的得到文件的最后修改时间。
![images](https://user-images.githubusercontent.com/25027560/38223505-d8ab53da-371d-11e8-9263-79814b6971a5.png)

**如何选择合适的缓存**
大致的顺序

- Cache-Control —— 请求服务器之前
- Expires —— 请求服务器之前
- If-None-Match (Etag) —— 请求服务器
- If-Modified-Since (Last-Modified) —— 请求服务器

协商缓存需要配合强缓存使用，如果不启用强缓存的话，协商缓存根本没有意义

大部分web服务器都默认开启协商缓存，而且是同时启用【Last-Modified，If-Modified-Since】和【ETag、If-None-Match】

但是下面的场景需要注意：

- 分布式系统里多台机器间文件的Last-Modified必须保持一致，以免负载均衡到不同机器导致比对失败；
- 分布式系统尽量关闭掉ETag(每台机器生成的ETag都会不一样）；


## DNS解析过程及原理
DNS=本地域名服务器，当用户访问一个网址，计算机就会提出域名解析请求，并发给本地域名服务器，本地域名服务器收到请求后，查询本地资源，如有记录则返回查询结果，如果资源较少会导致访问网址加载速度变慢，此时可以换一个DNS。

第一步：客户机提出域名解析请求,并将该请求发送给本地的域名服务器.

第二步：当本地的域名服务器收到请求后,就先查询本地的缓存,如果有该纪录项,则本地的域名服务器就直接把查询的结果返回.

第三步：如果本地的缓存中没有该纪录,则本地域名服务器就直接把请求发给根域名服务器,然后根域名服务器再返回给本地域名服务器一个所查询域(根的子域)的主域名服务器的地址.

第四步：本地服务器再向上一步返回的域名服务器发送请求,然后接受请求的服务器查询自己的缓存,如果没有该纪录,则返回相关的下级的域名服务器的地址.

第五步：重复第四步,直到找到正确的纪录.

第六步：本地域名服务器把返回的结果保存到缓存,以备下一次使用,同时还将结果返回给客户机.

让我们举一个例子来详细说明解析域名的过程.假设我们的客户机如果想要访问站点：www.linejet.com此客户本地的域名服务器是dns.company.com , 一个根域名服务器是NS.INTER.NET , 所要访问的网站的域名服务器是dns.linejet.com,域名解析的过程如下所示.

(1)客户机发出请求解析域名www.linejet.com的报文

(2)本地的域名服务器收到请求后, 查询本地缓存, 假设没有该纪录, 则本地域名服务器dns.company.com则向根域名服务器NS.INTER.NET发出请求解析域名www.linejet.com

(3)根域名服务器NS.INTER.NET收到请求后查询本地记录得到如下结果:linejet.com NS dns.linejet.com （表示linejet.com域中的域名服务器为：dns.linejet.com ）, 同时给出dns.linejet.com的地址,并将结果返回给域名服务器dns.company.com

(4)域名服务器dns.company.com 收到回应后,再发出请求解析域名www.linejet.com的报文.

(5)域名服务器 dns.linejet.com收到请求后,开始查询本地的记录，找到如下一条记录:www.linejet.com A 211.120.3.12 （表示linejet.com域中域名服务器dns.linejet.com的IP地址为:211.120.3.12）,并将结果返回给客户本地域名服务器dns.company.com

(6)客户本地域名服务器将返回的结果保存到本地缓存,同时将结果返回给客户机.

这样就完成了一次域名解析过程.


## 一次完整的HTTP请求所经历的7个步骤

HTTP通信机制是在一次完整的HTTP通信过程中，Web浏览器与Web服务器之间将完成下列7个步骤： 

**1. 建立TCP连接**

在HTTP工作开始之前，Web浏览器首先要通过网络与Web服务器建立连接，该连接是通过TCP来完成的，该协议与IP协议共同构建Internet，即著名的TCP/IP协议族，因此Internet又被称作是TCP/IP网络。HTTP是比TCP更高层次的应用层协议，根据规则，只有低层协议建立之后才能，才能进行更层协议的连接，因此，首先要建立TCP连接，一般TCP连接的端口号是80。

**2. Web浏览器向Web服务器发送请求命令**

一旦建立了TCP连接，Web浏览器就会向Web服务器发送请求命令。例如：GET/sample/hello.jsp HTTP/1.1。

**3. Web浏览器发送请求头信息**

浏览器发送其请求命令之后，还要以头信息的形式向Web服务器发送一些别的信息，之后浏览器发送了一空白行来通知服务器，它已经结束了该头信息的发送。 

**4. Web服务器应答**

客户机向服务器发出请求后，服务器会客户机回送应答， HTTP/1.1 200 OK ，应答的第一部分是协议的版本号和应答状态码。

**5. Web服务器发送应答头信息**

正如客户端会随同请求发送关于自身的信息一样，服务器也会随同应答向用户发送关于它自己的数据及被请求的文档。 

**6. Web服务器向浏览器发送数据**

Web服务器向浏览器发送头信息后，它会发送一个空白行来表示头信息的发送到此为结束，接着，它就以Content-Type应答头信息所描述的格式发送用户所请求的实际数据。

**7. Web服务器关闭TCP连接**

一般情况下，一旦Web服务器向浏览器发送了请求数据，它就要关闭TCP连接，然后如果浏览器或者服务器在其头信息加入了这行代码：

Connection:keep-alive

TCP连接在发送后将仍然保持打开状态，于是，浏览器可以继续通过相同的连接发送请求。保持连接节省了为每个请求建立新连接所需的时间，还节约了网络带宽。

## HTTP请求格式
当浏览器向Web服务器发出请求时，它向服务器传递了一个数据块，也就是请求信息，HTTP请求信息由3部分组成：

**（1）请求方法URI协议/版本**

```GET/sample.jsp HTTP/1.1```

“GET”代表请求方法，“/sample.jsp”表示URI，“HTTP/1.1代表协议和协议的版本。

**（2）请求头(Request Header)**

请求头包含许多有关的客户端环境和请求正文的有用信息。例如，请求头可以声明浏览器所用的语言，请求正文的长度等。例如：
```
Accept:image/gif.image/jpeg.*/*
Accept-Language:zh-cn
Connection:Keep-Alive
Host:localhost
User-Agent:Mozila/4.0(compatible:MSIE5.01:Windows NT5.0)
Accept-Encoding:gzip,deflate.
```

**（3）请求正文**

请求头和请求正文之间是一个空行，这个行非常重要，它表示请求头已经结束，接下来的是请求正文。请求正文中可以包含客户提交的查询字符串信息：

```username=joey&password=1234```

在以上的例子的HTTP请求中，请求的正文只有一行内容。当然，在实际应用中，HTTP请求正文可以包含更多的内容。
