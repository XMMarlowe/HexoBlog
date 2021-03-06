---
title: HTTP 和 HTTPS
author: Marlowe
tags:
  - HTTP
  - HTTPS
categories: 计算机网络
abbrlink: 49970
date: 2021-02-22 22:11:06
---
<!--more-->

### 什么是HTTP？

超文本传输协议，是一个基于请求与响应，无状态的，应用层的协议，常基于TCP/IP协议传输数据，互联网上应用最为广泛的一种网络协议,所有的WWW文件都必须遵守这个标准。设计HTTP的初衷是为了提供一种发布和接收HTML页面的方法。

#### HTTP的特点
1. **无状态：** 协议对客户端没有状态存储，对事物处理没有“记忆”能力，比如访问一个网站需要反复进行登录操作。
2. **无连接：** HTTP/1.1之前，由于无状态特点，每次请求需要通过TCP三次握手四次挥手，和服务器重新建立连接。比如某个客户机在短时间多次请求同一个资源，服务器并不能区别是否已经响应过用户的请求，所以每次需要重新响应请求，需要耗费不必要的时间和流量。
3. **基于请求和响应：** 基本的特性，由客户端发起请求，服务端响应。
4. 简单快速、灵活
5. **通信使用明文、请求和响应不会对通信方进行确认、无法保护数据的完整性**。

#### HTTP报文格式
<center>

![HTTP报文格式](https://img-blog.csdnimg.cn/2019080311162578.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx,size_16,color_FFFFFF,t_70)
</center>

1. **请求方法:** GET和POST是最常见的HTTP方法,初次以外还包括 DELETE、HEAD、OPTIONS、PUT、TRACE，不过现在大部分的浏览器只支持GET和POST。

2. **请求对应的URL地址：** 他和报文头的Host属性,组合起来是一个完整的请求URL。

3. **协议名称和版本号**。

4. **报文头：** 有若干个属性,形式为key:val,服务端据此获取客户端信息。

5. **是报文体：** 它将一个页面表单中的组件值通过param1=val1&parma=2的键值对形式编码成一个格式化串,它承载多个请求参数的数据,不但报文头可以传递请求参数,URL也可以通过/chapter15/user.html? param1=value1&param2=value2”的方式传递数值。


#### HTTP响应报文
1. **报文协议及版本**；

2. **状态码及状态描述**；

3. **响应报文头:** 也是由多个属性组成；

4. **响应报文体:** 即我们要的数据。


#### HTTP通信传输

![20210415105419](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210415105419.png)

客户端输入URL回车，DNS解析域名得到服务器的IP地址，服务器在80端口监听客户端请求，端口通过TCP/IP协议（可以通过Socket实现）建立连接。HTTP属于TCP/IP模型中的运用层协议，所以通信的过程其实是对应数据的入栈和出栈。

![20210415105502](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210415105502.png)
报文从运用层传送到运输层，运输层通过TCP三次握手和服务器建立连接，四次挥手释放连接。


#### HTTP的性能优化

![20210508153730](http://marlowe.oss-cn-beijing.aliyuncs.com/img/20210508153730.png)

通过以上图，可以从三个方面来优化HTTP的性能。

##### 服务器

衡量服务器性能的指标，主要有以下几个：

1. 吞吐量（或TPS、RPS、QPS）

2. 并发数

3. 响应时间

4. 资源利用率（CPU、内存、硬盘、网络）

* 提高吞吐量，吞吐量越高，服务器的性能越好！

* 提高并发数，支持的并发数越大，服务器的性能越好！

* 降低响应时间，响应时间越短，服务器的性能越好！

* 合理利用服务器资源，过高肯定是不行，过低也有可能是存在问题的！

* Linux服务器的监控工具主要有top、sar、glances等

##### 客户端

因为数据都要通过网络从服务器获取，所以它最基本的性能指标就是：**延迟**。

所谓的“延迟”其实就是“等待”，等待数据到达客户端时所花费的时间。

**延迟的原因，有几点：**

1. 距离：由于地理距离导致的延迟，是无法客服的，比如访问数千公里外的网站。

2. 带宽

3. DNS查询（如果域名在本地没有缓存的话）

4. TCP握手（必须要经过 SYN、SYN/ACK、ACK 三个包之后才能建立连接）

* 对于 HTTP 性能优化，有一个专门的测试网站：WebPageTest，或使用浏览器的开发者工具

* 一次 HTTP“请求 - 响应”的过程中延迟的时间是非常大的，有可能会占到９０％以上

* 所以，客户端优化的关键，降低延迟

##### 传输链路（客户端和服务器之间的传输链路）

使用CDN等技术，总之，要增加带宽，降低延迟，优化传输速度。

**具体的优化手段：**

主要是优化服务端的性能。

1. **前端：** 可以利用PageSpeed等工具进行检测并根据提示进行优化。

2. **后端：** 主要有以下几方面

**a.硬件、软件或服务**

比如更换强劲的CPU、内存、磁盘、带宽等，比如使用CDN

**b.服务器选择、参数调优**

选用高性能的服务器，比如Nginx，它强大的反向代理能力实现“动静分离”，动态页面交给Tomcat等，静态资源交给Nginx

另外，Nginx自身也有可以调优的参数，比如说禁用负载均衡锁、增大连接池，绑定 CPU 等

对于 HTTP 协议一定要启用长连接，因为TCP 和 SSL 建立新连接的成本非常高，可能会占到客户端总延迟的一半以上

TCP 的新特性“TCP Fast Open“，类似 TLS 的“False Start”，可以在初次握手的时候就传输数据，尽可能在操作系统和 Nginx 里开启这个特性，减少外网和内网里的握手延迟。

下面这个Nginx 配置，启用了长连接等优化参数，实现了动静分离

server {
  listen 80 deferred reuseport backlog=4096 fastopen=1024; 


  keepalive_timeout  60;
  keepalive_requests 10000;
  
  location ~* \.(png)$ {
    root /var/images/png/;
  }
  
  location ~* \.(php)$ {
    proxy_pass http://php_back_end;
  }
}
 

使用HTTP协议内置的“数据压缩”编码，可以选择标准的 gzip，也可以尝试新的压缩算法 br

不过在数据压缩的时候应当注意选择适当的压缩率，不是压缩的越厉害越好

**c.缓存**

网站系统内部，可以使用 Memcache、Redis等专门的缓存服务，把计算的中间结果和资源存储在内存或者硬盘里

Web 服务器首先检查缓存系统，如果有数据就立即返回给客户端

另外，CDN 的网络加速功能就是建立在缓存的基础之上的，可以这么说，如果没有缓存，那就没有 CDN。

利用好缓存功能的关键是理解它的工作原理，为每个资源都添加 ETag 和 Last-modified字段，再用 Cache-Control、Expires 设置好缓存控制属性。

其中最基本的是 max-age 有效期，标记资源可缓存的时间。对于图片、CSS 等静态资源可以设置较长的时间，比如一天或者

一个月，对于动态资源，除非是实时性非常高，也可以设置一个较短的时间，比如 1 秒或者 5 秒。这样一旦资源到达客户端，就

会被缓存起来，在有效期内都不会再向服务器发送请求。


### 什么是HTTPS？

HTTPS是身披SSL外壳的HTTP。HTTPS是一种通过计算机网络进行安全通信的传输协议，经由HTTP进行通信，利用SSL/TLS建立全信道，加密数据包。HTTPS使用的主要目的是提供对网站服务器的身份认证，同时保护交换数据的隐私与完整性。

PS:TLS是传输层加密协议，前身是SSL协议，由网景公司1995年发布，有时候两者不区分。


#### HTTPS的特点
1. **内容加密：** 采用混合加密技术，中间者无法直接查看明文内容。
2. **验证身份：** 通过证书认证客户端访问的是自己的服务器。
3. **保护数据完整性：** 防止传输的内容被中间人冒充或者篡改。


#### HTTPS的缺点

1. HTTPS的握手协议比较费时，所以会影响服务的响应速度以及吞吐量。
2. HTTPS也并不是完全安全的。他的证书体系其实并不是完全安全的。并且HTTPS在面对DDOS这样的攻击时，几乎起不
到任何作用。
3. 证书需要费钱，并且功能越强大的证书费用越高。


### HTTP和HTTPS的区别
1. 端口 ：HTTP的URL由“http://”起始且默认使用**端口80**，而HTTPS的URL由“https://”起始且默认使用**端口443**。
2. 安全性和资源消耗： **HTTP协议**运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。**HTTPS**是运行在SSL/TLS之上的HTTP协议，SSL/TLS 运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。**所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源。**
  * 对称加密：密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的对称加密算法有DES、AES等；
  * 非对称加密：密钥成对出现（且根据公钥无法推知私钥，根据私钥也无法推知公钥），加密解密使用不同密钥（公钥加密需要私钥解密，私钥加密需要公钥解密），相对对称加密速度较慢，典型的非对称加密算法有RSA、DSA等。

### 如何保证HTTP传输安全性？

目前大多数网站和app的接口都是采用http协议，但是http协议很容易就通过抓包工具监听到内容，甚至可以篡改内容，为了保证数据不被别人看到和修改，可以通过以下几个方面避免。

**重要的数据，要加密：** 比如用户名密码，我们需要加密，这样即使被抓包监听，他们也不知道原始数据是什么（如果简单的md5，是可以暴力破解），所以加密方法越复杂越安全，根据需要，常见的是 md5(不可逆)，aes（可逆），自由组合吧,你还可以加一些特殊字符啊，没有做不到只有想不到， 举例：username = aes(username), pwd = MD5(pwd + username);。。。。。

**非重要数据，要签名：** 签名的目的是为了防止篡改，比如http://www.xxx.com/getnews?id=1，获取id为1的新闻，如果不签名那么通过id=2,就可以获取2的内容等等。怎样签名呢？通常使用sign，比如原链接请求的时候加一个sign参数，sign=md5(id=1)，服务器接受到请求，验证sign是否等于md5(id=1)，如果等于说明正常请求。这会有个弊端，假如规则被发现，那么就会被伪造，所以适当复杂一些，还是能够提高安全性的。

**登录态怎么做：** http是无状态的，也就是服务器没法自己判断两个请求是否有联系，那么登录之后，以后的接口怎么判定是否登录呢，简单的做法，在数据库中存一个token字段（名字随意），当用户调用登陆接口成功的时候，就将该字段设一个值，（比如aes(过期时间)），同时返回给前端，以后每次前端请求带上该值，服务器首先校验是否过期，其次校验是否正确，不通过就让其登陆。（redis 做这个很方便哦，key有过期时间）。

### HTTPS为什么安全？

因为HTTPS保证了传输安全，防止传输过程被监听、防止数据被窃取，可以确认网站的真实性。

###  HTTPS的传输过程是怎样的?

客户端发起HTTPS请求，服务端返回证书，客户端对证书进行验证，验证通过后本地生成用于改造对称加密算法的随机数，通过证书中的公钥对随机数进行加密传输到服务端，服务端接收后通过私钥解密得到随机数，之后的数据交互通过对称加密算法进行加解密。

1. 客户使用https的URL访问Web服务器，要求与Web服务器建立SSL连接。
2. Web服务器收到客户端请求后，会将网站的证书信息（证书中包含公钥）传送一份给客户端。
3. 客户端拿到证书，证书校验通过后，用系统内置的CA证书，进行对证书解密，拿到公钥。
4. 客户端的浏览器与Web服务器开始协商SSL/TLS连接的安全等级，也就是信息加密的等级。
5. 客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站。
6. Web服务器利用自己的私钥解密出会话密钥。
7. Web服务器利用会话密钥对数据进行对称加密，再与客户端进程通讯。


<center>

![HTTPS传输过程](https://gitee.com/unclezs/image-blog/raw/master/blog/20200917102430.png)
</center>



### HTTP长连接、短连接

在**HTTP/1.0**中**默认使用短连接。** 也就是说，客户端和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。当客户端浏览器访问的某个HTML或其他类型的Web页中包含有其他的Web资源（如JavaScript文件、图像文件、CSS文件等），每遇到这样一个Web资源，浏览器就会重新建立一个HTTP会话。

而从**HTTP/1.1**起，**默认使用长连接，** 用以保持连接特性。使用长连接的HTTP协议，会在响应头加入这行代码：
```java
Connection:keep-alive
```
在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。**Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。** 实现长连接需要客户端和服务端都支持长连接。

**HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接。**

### CA证书的作用
CA证书就是服务端自己有一份公钥私钥，把公钥给CA证书，获得一份数字证书，当客户端来请求时，就拿这个数字证书给客户端，客户端再通过CA认证算法拿到公钥，再进行后续操作。



### 为什么需要证书?

防止”中间人“攻击，同时可以为网站提供身份证明。

### 验证证书安全性过程

1. 当客户端收到这个证书之后，使用本地配置的权威机构的公钥对证书进行解密得到服务端的公钥和证书的数字签名，数字签名经过CA公钥解密得到证书信息摘要。
2. 然后证书签名的方法计算一下当前证书的信息摘要，与收到的信息摘要作对比，如果一样，表示证书一定是服务器下发的，没有被中间人篡改过。因为中间人虽然有权威机构的公钥，能够解析证书内容并篡改，但是篡改完成之后中间人需要将证书重新加密，但是中间人没有权威机构的私钥，无法加密，强行加密只会导致客户端无法解密，如果中间人强行乱修改证书，就会导致证书内容和证书签名不匹配。

### 使用HTTPS会被抓包吗？
会被抓包，HTTPS只防止用户在不知情的情况下通信被监听，如果用户主动授信，是可以构建
“中间人”网络，代理软件可以对传输内容进行解密。


### SSL/TLS协议的基本过程

#### 客户端发出请求（ClientHello）
1. 支持的协议版本，比如TLS 1.0版。
2. 一个客户端生成的随机数，稍后用于生成”对话密钥”。
3. 支持的加密方法，比如RSA公钥加密。
4. 支持的压缩方法。
#### 服务器回应（SeverHello）
1. 确认使用的加密通信协议版本，比如TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。
2. 一个服务器生成的随机数，稍后用于生成”对话密钥”。
3. 确认使用的加密方法，比如RSA公钥加密，此时带有公钥信息。
4. 服务器证书。
#### 客户端回应
1. 一个随机数pre-master key。该随机数用服务器公钥加密，防止被窃听。
2. 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。
3. 客户端握手结束通知，表示客户端的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供服务器校验。

上面客户端回应中第一项的随机数，是整个握手阶段出现的第三个随机数，又称”pre-master key”。有了它以后，客户端和服务器就同时有了三个随机数，接着双方就用事先商定的加密方法，各自生成本次会话所用的同一把”会话密钥”。


### SSL建立连接过程
![SSL建立连接过程](https://img-blog.csdnimg.cn/20190803111825690.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9taW5nMTAwMDAx,size_16,color_FFFFFF,t_70)

1. client向server发送请求 **https://baidu.com，** 然后连接到server的443端口，发送的信息主要是随机值1和客户端支持的加密算法。
2. server接收到信息之后给予client响应握手信息，包括随机值2和匹配好的协商加密算法，这个加密算法一定是client发送给server加密算法的子集。
3. 随即server给client发送第二个响应报文是数字证书。服务端必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面，这套证书其实就是一对公钥和私钥。传送证书，这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间、服务端的公钥，第三方证书认证机构(CA)的签名，服务端的域名信息等内容。
4. 客户端解析证书，这部分工作是由客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值（预主秘钥）。
5. 客户端认证证书通过之后，接下来是通过随机值1、随机值2和预主秘钥组装会话秘钥。然后通过证书的公钥加密会话秘钥。
6. 传送加密信息，这部分传送的是用证书加密后的会话秘钥，目的就是让服务端使用秘钥解密得到随机值1、随机值2和预主秘钥。
7. 服务端解密得到随机值1、随机值2和预主秘钥，然后组装会话秘钥，跟客户端会话秘钥相同。
8. 客户端通过会话秘钥加密一条消息发送给服务端，主要验证服务端是否正常接受客户端加密的消息。
9. 同样服务端也会通过会话秘钥加密一条消息回传给客户端，如果客户端能够正常接受的话表明SSL层连接建立完成了。


### 一些问题

#### 那么为什么一定要用三个随机数，来生成”会话密钥”呢？
为了保证绝对随机，不相信服务器或者客户端的随机数，而是才用三个随机数再进行一定算法计算出真正的会话密钥(对称加密)。
#### 为什么不都使用对称加密而是才有非对称与对称加密组合？
因为非对称加密比较耗费性能，比对称加密慢了几倍甚至几百倍，所以才有了对称加密进行最终的数据加密。


### 参考
[HTTP与HTTPS详解](https://blog.unclezs.com/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/HTTP%E4%B8%8EHTTPS%E8%AF%A6%E8%A7%A3.html)

[HTTP和HTTPS协议，看一篇就够了](https://blog.csdn.net/xiaoming100001/article/details/81109617)

[HTTP的性能优化](https://blog.csdn.net/fengchuanyun/article/details/102463146)