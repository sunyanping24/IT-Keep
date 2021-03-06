<!-- TOC -->

- [TCP“两次握手”和“四次挥手”](#tcp两次握手和四次挥手)
  - [TCP报文段结构](#tcp报文段结构)
  - [TCP 三次握手](#tcp-三次握手)
  - [四次挥手](#四次挥手)
  - [为什么要三次握手](#为什么要三次握手)
  - [为什么要四次挥手](#为什么要四次挥手)
- [在浏览器地址栏输入一个地址->显示页面的过程都经历了什么](#在浏览器地址栏输入一个地址-显示页面的过程都经历了什么)
- [如何理解HTTP协议的无状态性](#如何理解http协议的无状态性)
- [`Cookie`和`Session`](#cookie和session)
  - [`Cookie`](#cookie)
  - [`Session`](#session)
  - [`Cookie` 和 `Session`的区别](#cookie-和-session的区别)
- [`URL`和`URI`有什么区别](#url和uri有什么区别)
- [HTTP的长连接和短链接](#http的长连接和短链接)
  - [TCP短连接](#tcp短连接)
  - [TCP长连接](#tcp长连接)
  - [长连接和短连接的优缺点](#长连接和短连接的优缺点)
  - [什么时候选择使用长连接，什么时候选择使用短连接](#什么时候选择使用长连接什么时候选择使用短连接)
- [HTTP和HTTPS的主要区别](#http和https的主要区别)
- [跨域/同源策略](#跨域同源策略)
  - [是否允许跨域的判定](#是否允许跨域的判定)
    - [浏览器判定流程](#浏览器判定流程)
    - [配置服务器实现跨域传输](#配置服务器实现跨域传输)
- [HTTPS](#https)
  - [HTTPS采用的加密方案：对称加密+非对称加密](#https采用的加密方案对称加密非对称加密)
  - [单项认证](#单项认证)
  - [双向认证](#双向认证)
- [Websocket](#websocket)
- [Nginx](#nginx)
  - [nginx作为web服务器和tomcat作为web服务器的区别](#nginx作为web服务器和tomcat作为web服务器的区别)
  - [nginx作为反向代理服务器](#nginx作为反向代理服务器)
  - [nginx的upstream模块](#nginx的upstream模块)
- [Tomcat](#tomcat)
  - [Tomcat的性能调优](#tomcat的性能调优)

<!-- /TOC -->

# TCP“两次握手”和“四次挥手”

 网上发现一篇比较好的文章，大部分是直接贴过来：[https://my.oschina.net/u/4198159/blog/3141874](https://my.oschina.net/u/4198159/blog/3141874)

## TCP报文段结构
一谈到 TCP 协议，大家最先想到的词就是 **「面向连接」** 和 **「可靠」** 。没错，TCP 协议的设计就是为了能够在客户端和服务器之间建立起一个可靠连接。

在讲连接过程之前，我们先来看看 TCP 的报文段结构，通过这个结构，我们可以知道 TCP 能够提供什么信息：

![https://oscimg.oschina.net/oscnet/up-e5800da775bfeadbcd150565f0455802fdf.png](https://oscimg.oschina.net/oscnet/up-e5800da775bfeadbcd150565f0455802fdf.png)

这里有几点是需要注意的：

- TCP 协议需要一个四元组（源IP，源端口，目的IP，目的端口）来确定连接，这要和 UDP 协议区分开。多说一句，IP 地址位于 IP 报文段，TCP 报文段是不含 IP 地址信息的。
- 基本 TCP 头部的长度是 20 字节，但是由于「选项」的长度是不确定的，所以需要「首部长度」字段明确给出头部长度。这里要注意的是，首部长度字段的单位是 32bit，也就是 4 字节，所以该字段的最小值是 5。
- 标橙色的字段（确认序号，接收窗口大小，ECE，ACK）用于「回复」对方，举个例子，服务器收到对方的数据包后，不单独发一个数据包来回应，而是稍微等一下，把确认信息附在下一个发往客户端的数据帧上，也就是捎带技术。
- 窗口大小是一个 16 位无符号数，也就是说窗口被限制在了 65535 字节，也就限制了 TCP 的吞吐量性能，这对一些高速以及高延迟的网络不太友好（可以想想为什么）。所幸 TCP 额外提供了窗口缩放（Window Scale）选项，允许对这个值进行缩放。

下面是 8 个标志位的含义，有的协议比较旧，可能没有前两个标志位：

![https://oscimg.oschina.net/oscnet/up-6dd2e9ca83855e78983a3726d3a1ec94f54.png](https://oscimg.oschina.net/oscnet/up-6dd2e9ca83855e78983a3726d3a1ec94f54.png)

标志位虽然很多，但是如果放到具体场景里来看的话，就很容易理解他们的作用了。

## TCP 三次握手
三次握手就是为了在客户端和服务器间建立连接，这个过程并不复杂，但里面有很多细节需要注意。

![https://oscimg.oschina.net/oscnet/up-6076656ae943ea29f47c7c045700179b93c.png](https://oscimg.oschina.net/oscnet/up-6076656ae943ea29f47c7c045700179b93c.png)

这张图就是握手的过程，可以看到客户端与服务器之间一共传递了三次消息，这三次握手其实就是两台机器之间互相确认状态，我们来一点一点看。

1. 第一次握手：建立连接。客户端发送连接请求报文段，将SYN位置为1，Sequence Number为x；然后，客户端进入SYN_SEND状态，等待服务器的确认；
2. 第二次握手：服务器收到SYN报文段。服务器收到客户端的SYN报文段，需要对这个SYN报文段进行确认，设置Acknowledgment Number为x+1(Sequence Number+1)；同时，自己自己还要发送SYN请求信息，将SYN位置为1，Sequence Number为y；服务器端将上述所有信息放到一个报文段（即SYN+ACK报文段）中，一并发送给客户端，此时服务器进入SYN_RECV状态；
3. 第三次握手：客户端收到服务器的SYN+ACK报文段。然后将Acknowledgment Number设置为y+1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。

完成了三次握手，客户端和服务器端就可以开始传送数据。以上就是TCP三次握手的总体介绍。

## 四次挥手
当客户端和服务器通过三次握手建立了TCP连接以后，当数据传送完毕，肯定是要断开TCP连接的啊。那对于TCP的断开连接，这里就有了神秘的“四次分手”。

![https://oscimg.oschina.net/oscnet/up-37a7a58ee2a13a341095e23e0c005d90553.png](https://oscimg.oschina.net/oscnet/up-37a7a58ee2a13a341095e23e0c005d90553.png)

1. 第一次分手：主机1（可以使客户端，也可以是服务器端），设置Sequence Number和Acknowledgment Number，向主机2发送一个FIN报文段；此时，主机1进入FIN_WAIT_1状态；这表示主机1没有数据要发送给主机2了；
2. 第二次分手：主机2收到了主机1发送的FIN报文段，向主机1回一个ACK报文段，Acknowledgment Number为Sequence Number加1；主机1进入FIN_WAIT_2状态；主机2告诉主机1，我“同意”你的关闭请求；
3. 第三次分手：主机2向主机1发送FIN报文段，请求关闭连接，同时主机2进入LAST_ACK状态；
4. 第四次分手：主机1收到主机2发送的FIN报文段，向主机2发送ACK报文段，然后主机1进入TIME_WAIT状态；主机2收到主机1的ACK报文段以后，就关闭连接；此时，主机1等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，主机1也可以关闭连接了。

## 为什么要三次握手
CP作为一种可靠传输控制协议，其核心思想：**既要保证数据可靠传输，又要提高传输的效率，** 而用三次恰恰可以满足以上两方面的需求！少于3次，数据传输没有保证，多于3次传输数据降低了效率。

- 第一次握手：Client 什么都不能确认；Server 确认了对方发送正常，自己接收正常
- 第二次握手：Client 确认了：自己发送、接收正常，对方发送、接收正常；Server 确认了：对方发送正常，自己接收正常
- 第三次握手：Client 确认了：自己发送、接收正常，对方发送、接收正常；Server 确认了：自己发送、接收正常，对方发送、接收正常

所以三次握手就能确认双发收发功能都正常，缺一不可。

## 为什么要四次挥手
任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接。

举个例子：A 和 B 打电话，通话即将结束后，A 说“我没啥要说的了”，B回答“我知道了”，但是 B 可能还会有要说的话，A 不能要求 B 跟着自己的节奏结束通话，于是 B 可能又巴拉巴拉说了一通，最后 B 说“我说完了”，A 回答“知道了”，这样通话才算结束。

# 在浏览器地址栏输入一个地址->显示页面的过程都经历了什么

1. DNS域名服务器解析域名： 解析域名，找到对应的服务器地址。
2. 客户端与服务器建立TCP连接： 3次握手，建立连接，建立连接成功之后才能收发数据
3. 客户端发送HTTP请求
4. 服务器处理数据：HTTP服务器处理请求（Apache、Nginx等）
5. 返回响应结果
4. 关闭TCP连接： 4次握手
6. 浏览器解析HTML
7. 浏览器布局渲染

> 详细可以参考这篇文章：[https://segmentfault.com/a/1190000006879700](https://segmentfault.com/a/1190000006879700)

# 如何理解HTTP协议的无状态性

HTTP是一种无状态协议，即服务器不保留与客户交易时的任何状态。一次请求，一次应答，连接即就会断开。  
也就是说，上一次的请求对这次的请求没有任何影响，服务端也不会对客户端上一次的请求进行任何记录处理。

**HTTP协议的无状态性有什么问题**

用户登录后，切换到其他界面，进行操作，服务器端是无法判断是哪个用户登录的。 每次进行页面跳转的时候，得重新登录。

**解决问题的办法**

既然HTTP协议是无状态的，不会记录用户信息，那么怎么样才能让HTTP协议记录用户信息呢？换句话说，服务器怎么判断发来HTTP请求的是哪个用户？

于是，两种用于保持HTTP状态的技术就应运而生了，一个是 `Cookie`，而另一个则是 `Session`。

# `Cookie`和`Session`

![Cookie和Session](http://sunyanping.gitee.io/it-keep/ASSET/Cookie和Session.png)

## `Cookie`
Cookie 是客户端的存储空间，由浏览器来维持。具体来说 cookie 机制采用的是在客户端保持状态的方案。

**Cookie 的实现过程：**

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie，当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。

也就是 Cookie 是服务器生成的，但是发送给客户端，并且由客户端来保存。每次请求加上 Cookie就行了。服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。

## `Session`
Session 是另一种记录客户状态的机制，不同的是 Cookie 保存在客户端浏览器中，而 Session 保存在服务器上。

客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上，这就是 Session。客户端浏览器再次访问时，只需要从该 Session 中查找该客户的状态就可以了。

虽然 Session 保存在服务器，对客户端是透明的，它的正常运行仍然需要客户端浏览器的支持。这是因为 Session 需要使用Cookie 作为识别标志。HTTP协议是无状态的，Session 不能依据HTTP连接来判断是否为同一客户，因此服务器向客户端浏览器发送一个名为 JSESSIONID 的 Cookie，它的值为该 Session 的 id（即放在HTTP响应报文头部信息里的Set-Cookie）。Session依据该 Cookie 来识别是否为同一用户。

## `Cookie` 和 `Session`的区别
1. Cookie 数据存放在客户的浏览器上，Session 数据放在服务器上；
2. Cookie 不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗，考虑到安全应当使用 Session ；
3. Session 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能。考虑到减轻服务器性能方面，应当使用COOKIE；
4. 单个Cookie 在客户端的限制是3K，就是说一个站点在客户端存放的COOKIE不能超过3K；

# `URL`和`URI`有什么区别
常见的提问是：`URL`和`URI`有什么区别? 但是其实说这两者的关系时也是避免不了`URN`的。

先来看一下下面这张图，大致就表明了这三者之间的关系：  
![URI、URL、URN之间的关系](http://sunyanping.gitee.io/it-keep/ASSET/URI、URL、URN之间的关系.jpg)

- URI（Uniform Resource Identifier ）：统一资源标识符，是以某种统一的（标准化的）方式标识资源的简单字符串。   

URI一般由三部分组成：
1. 访问资源的命名机制。 
2. 存放资源的主机名。 
3. 资源自身的名称，由路径表示。 

典型情况下，这种字符串以scheme（命名URI的名字空间的标识符——一组相关的名称）开头，语法如下：  `scheme：[// [user：password @] host [：port]] [/] path [？查询] [#片段]`  

URI以scheme和冒号开头。Scheme用大写/小写字母开头，后面为空或者跟着更多的大写/小写字母、数字、加号、减号和点号。冒号把scheme与scheme-specific-part分开了，并且scheme-specific-part的语法和语义（意思）由URI的名字空间决定。如下面的例子：
`http://www.cnn.com`，其中`http`是`scheme`，`//www.cnn.com`是 `scheme-specific-part`，并且它的scheme与scheme-specific-part被冒号分开了。

URI有绝对和相对之分，绝对的URI指以scheme（后面跟着冒号）开头的URI。前面提到的`http://www.cnn.com`就是绝对的URI的一个例子，其它的例子还有`mailto:jeff@javajeff.com`、`news:comp.lang.java.help`和`xyz://whatever`。你可以把绝对的URI看作是以某种方式引用某种资源，而这种方式对标识符出现的环境没有依赖。如果使用文件系统作类比，绝对的URI类似于从根目录开始的某个文件的径。 

与绝对的URI不同的，相对的URI不是以scheme（后面跟着冒号）开始的URI。 它的一个例子是`articles/articles.html`。你可以把相对的URI看作是以某种方式引用某种资源，而这种方式依赖于标识符出现的环境。如果用文件系统作类比，相对的URI类似于从当前目录开始的文件路径。

- URL（Uniform Resource Locator）：统一资源定位符。通俗地说，URL是Internet上用来描述信息资源的字符串，主要用在各种WWW客户程序和服务器程序上，特别是著名的Mosaic。采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服务器的地址和目录等。  

URL的格式一般由下列三部分组成：

1. 协议(或称为服务方式);
2. 存有该资源所在的服务器的名称或IP地址(包括端口号);
3. 主机资源的具体地址。

- URN（Uniform Resource Name）：统一资源名称  

以下是一些例子，作为解释：  

- ftp://ftp.is.co.za/rfc/rfc1808.txt (also a URL because of the protocol)
- http://www.ietf.org/rfc/rfc2396.txt (also a URL because of the protocol)
- ldap://[2001:db8::7]/c=GB?objectClass?one (also a URL because of the protocol)
- mailto:John.Doe@example.com (also a URL because of the protocol)
- news:comp.infosystems.www.servers.unix (also a URL because of the protocol)
- tel:+1-816-555-1212
- telnet://192.0.2.16:80/ (also a URL because of the protocol)
- urn:oasis:names:specification:docbook:dtd:xml:4.1.2

**这些全都是URI, 其中有些是URL。哪些? 就是那些提供了访问机制的。**

**总结： URI可以分为URL,URN或同时具备locators 和names特性的一个东西。URN作用就好像一个人的名字，URL就像一个人的地址。换句话说：URN确定了东西的身份，URL提供了找到它的方式。**

参考文章：  
- [https://blog.csdn.net/readiay/article/details/52862379](https://blog.csdn.net/readiay/article/details/52862379)
- [https://www.cnblogs.com/gaojing/archive/2012/02/04/2413626.html](https://www.cnblogs.com/gaojing/archive/2012/02/04/2413626.html)
- [https://www.php.cn/div-tutorial-413616.html](https://www.php.cn/div-tutorial-413616.html)


# HTTP的长连接和短链接

**常说的HTTP的长连接和短链接实质上时TCP协议的长连接和短链接。** HTTP属于应用层协议，在传输层使用TCP协议，在网络层使用IP协议。 IP协议主要解决网络路由和寻址问题，TCP协议主要解决如何在IP层之上可靠地传递数据包，使得网络上接收端收到发送端所发出的所有包，并且顺序与发送顺序一致。TCP协议是可靠的、面向连接的。

在HTTP/1.0中默认使用短连接。也就是说，客户端和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。当客户端浏览器访问的某个HTML或其他类型的Web页中包含有其他的Web资源（如JavaScript文件、图像文件、CSS文件等），每遇到这样一个Web资源，浏览器就会重新建立一个HTTP会话。

而从HTTP/1.1起，默认使用长连接，用以保持连接特性。使用长连接的HTTP协议，会在响应头加入这行代码：
```
Connection:keep-alive
```
在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。实现长连接需要客户端和服务端都支持长连接。

## TCP短连接

TCP短连接的情况：client向server发起连接请求，server接到请求，然后双方建立连接。client向server发送消息，server回应client，然后一次请求就完成了。这时候双方任意都可以发起close操作，不过一般都是client先发起close操作。上述可知，短连接一般只会在 client/server间传递一次请求操作。
> 建立连接——数据传输——关闭连接...建立连接——数据传输——关闭连接

短连接的优点是：管理起来比较简单，存在的连接都是有用的连接，不需要额外的控制手段。

## TCP长连接

TCP长连接的情况：client向server发起连接，server接受client连接，双方建立连接，client与server完成一次请求后，它们之间的连接并不会主动关闭，后续的读写操作会继续使用这个连接。

> 建立连接——数据传输...（保持连接）...数据传输——关闭连接

TCP的保活功能主要为服务器应用提供。如果客户端已经消失而连接未断开，则会使得服务器上保留一个半开放的连接，而服务器又在等待来自客户端的数据，此时服务器将永远等待客户端的数据。保活功能就是试图在服务端器端检测到这种半开放的连接。

如果一个给定的连接在两小时内没有任何动作，服务器就向客户发送一个探测报文段，根据客户端主机响应探测4个客户端状态：

- 客户主机依然正常运行，且服务器可达。此时客户的TCP响应正常，服务器将保活定时器复位。
- 客户主机已经崩溃，并且关闭或者正在重新启动。上述情况下客户端都不能响应TCP。服务端将无法收到客户端对探测的响应。服务器总共发送10个这样的探测，每个间隔75秒。若服务器没有收到任何一个响应，它就认为客户端已经关闭并终止连接。
- 客户端崩溃并已经重新启动。服务器将收到一个对其保活探测的响应，这个响应是一个复位，使得服务器终止这个连接。
- 客户机正常运行，但是服务器不可达。这种情况与第二种状态类似。

## 长连接和短连接的优缺点

**长连接：** 可以省去较多的TCP建立和关闭的操作，**减少浪费，节约时间**。对于频繁请求资源的客户端适合使用长连接。在长连接的应用场景下，client端一般不会主动关闭连接，当client与server之间的连接一直不关闭，随着**客户端连接越来越多，server会保持过多连接**。这时候server端需要采取一些策略，如关闭一些长时间没有请求发生的连接，这样可以避免一些恶意连接导致server端服务受损；如果条件允许则可以限制每个客户端的最大长连接数，这样可以完全避免恶意的客户端拖垮整体后端服务。

**短连接：** 对于服务器来说**管理较为简单，存在的连接都是有用的连接，不需要额外的控制手段**。但如果客户请求频繁，将在TCP的**建立和关闭操作上浪费较多时间和带宽**。

所以没有十全十美的选择和解决方案，真能针对不同的场景，选择不同的解决方案。

## 什么时候选择使用长连接，什么时候选择使用短连接

长连接多用于操作频繁，点对点的通讯，而且连接数不能太多情况，。每个TCP连接都需要三步握手，这需要时间，如果每个操作都是先连接，再操作的话那么处理速度会降低很多，所以每个操作完后都不断开，次处理时直接发送数据包就OK了，不用建立TCP连接。例如：**数据库的连接用长连接**， 如果用短连接频繁的通信会造成socket错误，而且频繁的socket 创建也是对资源的浪费。 
  
而像WEB网站的http服务一般都用短链接，因为长连接对于服务端来说会耗费一定的资源，而像WEB网站这么频繁的成千上万甚至上亿客户端的连接用短连接会更省一些资源，如果用长连接，而且同时有成千上万的用户，如果每个用户都占用一个连接的话，那可想而知吧。所以并发量大，但每个用户无需频繁操作情况下需用短连好。

 可以参考一篇解释的比较通俗的文章：[https://www.jianshu.com/p/3fc3646fad80](https://www.jianshu.com/p/3fc3646fad80)

# HTTP和HTTPS的主要区别

HTTPS相对于HTTP来说，主要要解决的问题是：在HTTP的基础上，设计出更加安全的HTTP通道。即HTTPS是HTTP的安全版。

Http：超文本传输协议（Http，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。设计Http最初的目的是为了提供一种发布和接收HTML页面的方法。它可以使浏览器更加高效。Http协议是以明文方式发送信息的，如果黑客截取了Web浏览器和服务器之间的传输报文，就可以直接获得其中的信息。

Https：是以安全为目标的Http通道，是Http的安全版。Https的安全基础是SSL。SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。SSL协议可分为两层：SSL记录协议（SSL Record Protocol），它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。SSL握手协议（SSL Handshake Protocol），它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

*这两者具体的设计目标、设计原理、优缺点及工作原理等不同点可以到网上搜。*
当然这篇入门文章也讲得比较详细：[https://blog.csdn.net/qq_38289815/article/details/80969419](https://blog.csdn.net/qq_38289815/article/details/80969419)

# 跨域/同源策略

前端经常遇到的问题`Access-Control-Allow-Origin`，俗称**跨域**。

由于安全问题考虑，浏览器有**同源策略**，对于不同源站点之间的相互请求会做限制，而对于不同源的站点之间相互请求就会出现上述所提的跨域问题。所以首先我们需要明确一点：**跨域问题实际上是由于浏览器的同源策略导致的**。

> 同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。同源策略，它是由Netscape提出的一个著名的安全策略。
现在所有支持JavaScript 的浏览器都会使用这个策略。
所谓同源是指，域名，协议，端口相同。
当一个浏览器的两个tab页中分别打开来 百度和谷歌的页面
当浏览器的百度tab页执行一个脚本的时候会检查这个脚本是属于哪个页面的，
即检查是否同源，只有和百度同源的脚本才会被执行。
如果非同源，那么在请求数据时，浏览器会在控制台中报一个异常，提示拒绝访问。

## 是否允许跨域的判定
同源策略加强了安全的同时，对开发带来了极大的不便利。因此开发者又开发了多挣解决同源问题的办法，来允许数据的跨域传输。有时多个域名可以对应一个服务器，浏览器是没法通过域名来判断是否同源。而是通过服务器响应的header信息来区别是否同源的。当浏览器发现不同源的时候就会拒绝跨域。所以一般情况下我们可以看到很多请求都是有一个预请求。

### 浏览器判定流程
 
- 浏览器先根据同源策略对前端页面和后台交互地址做匹配，若同源，则直接发送数据请求；若不同源，则发送跨域请求。

- 服务器解析程序收到浏览器跨域请求后，根据自身配置返回对应文件头。若未配置过任何允许跨域，则文件头里不包含Access-Control-Allow-origin字段，若配置过域名，则返回Access-Control-Allow-origin+ 对应配置规则里的域名的方式。

- 浏览器根据接受到的http文件头里的Access-Control-Allow-origin字段做匹配，若无该字段，说明不允许跨域；若有该字段，则对字段内容和当前域名做比对，如果同源，则说明可以跨域，浏览器发送该请求；若不同源，则说明该域名不可跨域，不发送请求

### 配置服务器实现跨域传输

- Access-Control-Allow-Origin（必含） – 允许的域名，只能填通配符或者单域名
- Access-Control-Allow-Methods（必含） – 这允许跨域请求的http方法（常见有POST、GET、OPTIONS）

- Access-Control-Allow-Headers（当预请求中包含Access-Control-Request-Headers时必须包含） – 这是对预请求当中Access-Control-Request-Headers的回复，和上面一样是以逗号分隔的列表，可以返回所有支持的头部。

- Access-Control-Allow-Credentials（可选） – 该项标志着请求当中是否包含cookies信息，只有一个可选值：true（必为小写）。如果不包含cookies，请略去该项，而不是填写false。这一项与XmlHttpRequest2对象当中的withCredentials属性应保持一致，即withCredentials为true时该项也为true；withCredentials为false时，省略该项不写。反之则导致请求失败。

- Access-Control-Max-Age（可选） – 以秒为单位的缓存时间。预请求的的发送并非免费午餐，允许时应当尽可能缓存。

# HTTPS

`HTTP`协议：超文本传输协议。内容是明文传输。  
`HTTPS`协议：安全通信的超文本传输协议。内容是密文传输。

这两者都是基于TCP协议的。但是HTTPS协议是基于HTTP上又添加了`TLS`协议,而`TLS`协议的前身又是`SSL`协议。**TLS协议是传输层加密协议**。由于HTTP是明文传输的，所以很容易就可以通过抓包获取到传输的内容，所以就催生了HTTPS的诞生。

HTTPS相对HTTP提供了更安全的数据传输保障，主要体现在三个方面：

1. 内容加密：客户端到服务器的内容都是以加密形式传输，中间者无法直接查看明文内容；

2. 身份认证：通过校验保证客户端访问的是自己的服务器；

3. 数据完整性：防止内容被第三方冒充或者篡改。

## HTTPS采用的加密方案：对称加密+非对称加密
1. 对称加密效率高，并且密文难破解，但是密钥需要在网络中传输给服务器，这样要是被捕获到，就很容易被黑客破解。
2. 非对称加密效率低，但是私钥只有一方拥有，密文只能被私钥解开。安全性很高。

基于以上2点的原因，HTTPS使用两者结合的方式。使用对称加密的方式加密内容，使用非对称加密的方式加密解密的密钥。这样就取得一个平衡点。

## 单项认证

1. 客户端发送TLS版本等信息
2. 服务端给客户端返回TLS版本、随机数等信息，以及服务器公钥。
3. 客户端校验服务端证书是否合法，合法继续，否则告警
4. 客户端发送自己可支持的对称加密方案给服务端供其选择
5. 服务器选择加密程度高的加密方式
6. 服务器将选择好的加密方案以明文的方式方给客户端
7. 客户端收到加密方式后，产生随机码，作为对称加密的密钥，使用对称加密密钥加密后的信息和使用服务端公钥进行加密后的密钥，发送给服务端
8. 服务端使用私钥对对称加密的密钥进行解密，然后再使用对称密钥解密数据内容

## 双向认证

1. 客户端发送TLS版本等信息
2. 服务端给客户端返回TLS版本、随机数等信息，以及服务器公钥。
3. 客户端校验服务端证书是否合法，合法继续，否则告警
4. 客户端将自己的证书及公钥发送给服务端
5. 服务端对客户端的证书进行校验，校验通过后，获得客户端的公钥
6. 客户端发送自己可支持的对称加密方案给服务端供其选择
7. 服务器选择加密程度高的加密方式，使用客户端的公钥加密后发送给客户端
8. 客户端收到加密方式后，产生随机码，作为对称加密的密钥，使用对称加密密钥加密后的信息和使用服务端公钥进行加密后的密钥，发送给服务端
8. 服务端使用私钥对对称加密的密钥进行解密，然后再使用对称密钥解密数据内容

# Websocket

`Websocket`协议是H5出的，是一个持久化的协议。主要是为了解决HTTP是非持久化协议的问题。虽然HTTP协议也可以长连接keep-alive,可以在一次连接中发送多次请求，但是他永远都是一个Request对应一个Response,并且Response都是被动的，不可能主动发起。

# Nginx

## nginx作为web服务器和tomcat作为web服务器的区别

1. nginx只可以处理静态资源，处理静态资源的效率非常高。占用内存少。
2. nginx可以做反向代理服务器
3. nginx模块化配置很灵活
4. tomcat可以处理静态资源也可以处理动态请求资源
5. tomcat可以作为servlet容器，相当于是一个应用服务器

## nginx作为反向代理服务器

**正向代理**：客户端将请求发送给代理服务器，并告知代理服务器需要使用哪个后台服务处理该请求，然后代理服务器，将请求发送给服务器，并将服务器响应的数据返回给客户端。

**反向代理**：客户端将请求发送给代理服务器，由代理服务器自行确定使用哪台后台服务器处理该请求，然后将服务器相应的数据返回给客户端。

**为什么使用反向代理**：  
1. 所有请求都必须经过代理服务器，可以起到保护网站安全的作用。
2. 实现负载均衡。
3. 通过缓存静态资源，加速web请求。

## nginx的upstream模块

nginx的upstream模块是用来做负载均衡的，使用upstream可以定义负载的服务器组。如下：

```
http {

    upstream server-cluster {

        server 192.168.3.84:8088 weight=1 max-fails=3 fail-timeout=20s;
        server 192.168.3.83:8088 weight=2;

        server 192.168.3.82:8088 backup;
    }

    server {
       ... 

       location / {
           proxy_pass http://server-cluster;
           health_checks;
       }
    }
}

```

1. nginx负载均衡的策略

- 轮询： 将请求循环发送到后端服务器
- weight: 通过权重值分配，权重值越大的分配到的请求次数越多
- ip_hash: 按照客户端ip的hash结果分配。相同的客户端被分配的是同一服务。这样还可以保证session的一致性。
- url_hash: 按照url的hash结果分配。这样相同的请求将被分配的是同一服务。
- fair: 根据后台服务的相应时间来分配请求。

# Tomcat

1. tomcat是使用java语言编写的，运行在jvm上。
2. tomcat与web/http请求  
 tomcat的`Connector`组件实现了HTTP请求的解析，Tomcat通过`Connector`组件接收HTTP请求并解析，然后把解析的信息交给Servlet来处理：
（1）对于静态资源，tomcat使用自己默认的servlet应用来处理
（2）对于动态资源，tomcat使用部署的servlet应用来处理
3. tomcat与nginx服务搭配使用

## Tomcat的性能调优

1. 调整`Connector`的连接数，可以适当的增加并发数

server.xml
```
<Connector executor="tomcatThreadPool"
            port="8080" protocol="HTTP/1.1"
            connectionTimeout="20000"
            redirectPort="8443" 
            maxThreads="300"
            minSpaceThread="50"
            acceptCount="250"
            enableLookups="false"
            maxKeepAliveRequests="1"/>
```
2. 增加`Executor`,为tomcat配置线程池，可以减少线程的频繁创建与销毁，提高线程的使用效率。  
server.xml
```
<Executor name="tomcatThreadPool"   
         namePrefix="catalina-exec-"   
         maxThreads="1000"   
         minSpareThreads="100"  
         maxIdleTime="60000"  
         maxQueueSize="Integer.MAX_VALUE"  
         prestartminSpareThreads="false"  
         threadPriority="5"  
         className="org.apache.catalina.core.StandardThreadExecutor"/>
```
3. 调整JVM参数，调整Tomcat运行时所需的虚拟内存。

catalina.sh
```
JAVA_OPTS="$JAVA_OPTS -Xmx512m -Xms512m -Xmn170m -Xss128k --XX:NewRatio=4 -XX:SrrvivorRatio=4"
```
4. 调整垃圾回收策略

catalina.sh
```
JAVA_OPTS="$JAVA_OPTS -Xmx512m -Xms512m -Xmn170m -Xss128k -XX:NewRatio=4 -XX:SrrvivorRatio=4 -XX:+UseParallelGC -XX:ParallelGCThreads=4 -XX:+UserParallelOldGC -XX:MaxGCPauseMillis=100"
```
**串行收集器**
- `-XX:+UseSerialGC`: 代表的使用串行收集器

**并行收集器**
- `-XX:+UseParallelGC`: 代表使用的使并行收集器,仅对新生代有效。
- `-XX:ParallelGCThreads=4`： 并行收集器的线程数量
- `-XX:+UseParallelOldGC`: 老年代垃圾收集方式为并行收集
- `-XX:MaxGCPauseMillis=100`: 每次年轻代垃圾回收的最长时间
- `-XX:+UseAdaptiveSizePolicy`: 表示并行收集器会自动选择新生代区大小和相应的Survivor区比例，以达到目标系统规定的最低响应时间或者收集频率

**并发收集器**
- `-XX:+UseConcMarkSweepGC`: 代表使用的并发收集器
