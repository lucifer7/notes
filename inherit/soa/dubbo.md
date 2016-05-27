# Dubbo
[TOC]

## 帮助文档

[dubbo用户指南](http://dubbo.io/User+Guide-zh.htm#UserGuide-zh-%E8%83%8C%E6%99%AF)
[dubbox-github](https://github.com/shuvigoss/dubbox)
[JBoss easy](http://docs.jboss.org/resteasy/docs/2.2.1.GA/userguide/html/)
[oracle官方指南](http://docs.oracle.com/javaee/7/tutorial/)
[IBM rest文档](http://www.ibm.com/developerworks/cn/java/j-lo-jaxrs/)
[亚马逊微架构SOA强行的6个原则](http://www.infoq.com/cn/articles/micro-soa-1)
[OSGI入门](http://osgi.com.cn/)
## 一、基础概念

### 1.1  Http1.0 Http1.1 tcp socket keepalive

**（1）先解释什么是tcp-（传输层）**

	tcp 是传输层的协议

	a. 三次握手：
	建立一个tcp需要三次握手，为什么要三次？个人理解，是为了server端需要确认能收到clent端的，client端也要确认能收到server端的响应！so：

	第一次握手：client端发送syn到服务器，自己进入SYN_SEND状态，表示自己发送了，要等大哥server你确认一下
	第二次握手：server端收到了，这时候他得告诉client端，兄弟你要知道我能收到你的信息，有空的话也确认下能不能收到我的响应，于是发了2个包，syn+ack
	第三次握手：client收到了syn+ack，知道自己能发送请求给server了，于是他发出了ack包，告诉服务端我也能收到你的响应。此时客户端和服务端进入ESTABLISHED状态，三次握手完成！
	（以上修饰是个人理解）

	b. 三次握手成功后，才能建立tcp连接：
	首先，三次握手中，是不会传递数据的（这里的数据比如是http request中的请求包数据或者http response中的响应包）。 三次握手后，server和client才正式开始传递数据。
	理想状态：（指网络通信良好等等？） tcp是长连接，会一直保持下去。
	直到——》server或者client端二者主动关闭连接，那么问题来了，怎么关闭？据说有4次握手（下次再看，反正感觉就是服务端和客户端交互，最终大家确认断开。。）


**（2） Http**

	a. http 和 tcp
	以前有个概念很容忍搞不清楚:就是为什么Http是无状态的短连接，而TCP是有状态的长连接？Http不是建立在TCP的基础上吗，为什么还能是短连接？

	现在明白了，Http就是在每次请求完成后就把TCP连接关了，所以是短连接。而我们直接通过Socket编程使用TCP协议的时候，因为我们自己可以通过代码区控制什么时候打开连接什么时候关闭连接，只要我们不通过代码把连接关闭，这个连接就会在客户端和服务端的进程中一直存在，相关状态数据会一直保存着。

	b. 什么是无状态？

	所谓的无状态，是指浏览器每次向服务器发起请求的时候，不是通过一个连接，而是每次都建立一个新的连接。如果是一个连接的话，服务器进程中就能保持住这个连接并且在内存中记住一些信息状态。而每次请求结束后，连接就关闭，相关的内容就释放了，所以记不住任何状态，成为无状态连接。

	c. 那么无状态的问题来了

	它也有一个很大的缺点就是，它效率很低，因此Keep-Alive被提出用来解决效率低的问题。

	Keep-Alive功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接，但是会影响了双方的性能，所以加了限制。

	市场上 的大部分Web服务器，包括iPlanet、IIS和Apache，都支持HTTP Keep-Alive。对于提供静态内容的网站来说，这个功能通常很有用。但是，对于负担较重的网站来说，
	这里存在另外一个问题：虽然为客户保留打开的连 接有一定的好处，但它同样影响了服性能，因为在处理暂停期间，本来可以释放的资源仍旧被占用。当Web服务器和应用服务器在同一台机器上运行时，Keep- Alive功能对资源利用的影响尤其突出。 此功能为HTTP 1.1预设的功能，HTTP 1.0加上Keep-Aliveheader也可以提供HTTP的持续作用功能。

	Keep-Alive: timeout=5, max=100 timeout：过期时间5秒（对应httpd.conf里的参数是：KeepAliveTimeout），max是最多一百次请求，强制断掉连接
	就是在timeout时间内又有新的连接过来，同时max会自动减1，直到为0，强制断掉。见下面的四个图，注意看Date的值（前后时间差都是在5秒之内）！


**（3）Http 1.0 / 1.1**

	a HTTP/1.0

	在HTTP/1.0版本中，并没有官方的标准来规定Keep-Alive如何工作，因此实际上它是被附加到HTTP/1.0协议上，如果客户端浏览器支持Keep-Alive，那么就在HTTP请求头中添加一个字段 Connection: Keep-Alive，当服务器收到附带有Connection: Keep-Alive的请求时，它也会在响应头中添加一个同样的字段来使用Keep-Alive。这样一来，客户端和服务器之间的HTTP连接就会被保持，不会断开（超过Keep-Alive规定的时间，意外断电等情况除外），当客户端发送另外一个请求时，就使用这条已经建立的连接

	b HTTP/1.1

	在HTTP/1.1版本中，官方规定的Keep-Alive使用标准和在HTTP/1.0版本中有些不同，默认情况下所在HTTP1.1中所有连接都被保持，除非在请求头或响应头中指明要关闭：Connection: Close  ，这也就是为什么Connection: Keep-Alive字段再没有意义的原因。另外，还添加了一个新的字段Keep-Alive:，因为这个字段并没有详细描述用来做什么，可忽略它

	c Keep-Alive 在 Java实现--客户端

	在客户端，Java抽象了Keep-Alive，和程序员分享离开来，HttpURLConnection类自动实现了Keep-Alive，如果程序员没有介入去操作Keep-Alive，Keep-Alive会通过客户端内部的一个HttpURLConnection类的实例对象来自动实现。也就是说，在java中keep-alive是由一个Java类库来实现的，但在其他类库中不一定可用。



	d Keep-Alive 在Java实现--服务器端

	在服务器端，Java依然是将Keep-Alive抽象出来，HttpServlet、HttpServletRequest、和HttpServletResponse类自动实现 了Keep-Alive。这种情况下一些由第三方控制的操作是可能的，如在KeepAliveServlet中提到的JavaWebServer，Keep-Alive是否启用由两个因素决定，内容长度和输出大小，如果内容长度是响应的一部分（即这段内容长度输出后还有内容需要输出），则Keep-Alive被启用（当然需要客户端支持的情况下）；如果内容长度未设定，则Servlet会试着计算响应缓冲区长度以确定内容长度，在Javasoft实现中，使用一个4KB的缓冲区（相当于上面说的响应）。也就是说如果内容长度未设定，并且返回数据超过4KB，此时相当于内容长度大于响应长度，而不是响应长度一部分，Keep-Alive就不会被启用 。



###  1.2 初始REST以及JAX-RS

	从WIKI上截的一段：
	REST 从资源的角度来观察整个网络，分布在各处的资源由URI 确定，而客户端的应用通过URI来获取资源的表形。获得这些表形致使这些应用程序转变了其状态。随着不断获取资源的表形，客户端应用不断地在转变着其状态，所谓表形化的状态转变（Representational State Transfer）。


**JAX-RS简介：**

JAX-RS是一套用java实现REST服务的规范，提供了一些标注将一个资源类，一个POJOJava类，封装为Web资源。

**1.标注包括**：
**@Path:**		标注资源类或方法的相对路径
**@GET，@PUT，@POST，@DELETE:** 标注方法是用的HTTP请求的类型
**@Produces** 标注返回的MIME媒体类型
**@Consumes** 标注可接受请求的MIME媒体类型

以及@PathParam，@QueryParam，@HeaderParam，@CookieParam，@MatrixParam，@FormParam,分别标注方法的参数来自于HTTP请求的不同位置，例如@PathParam来自于URL的路径，@QueryParam来自于URL的查询参数，@HeaderParam来自于HTTP请求的头信息，@CookieParam来自于HTTP请求的Cookie。

**2.目前JAX-RS的实现包括：**

	Apache CXF，开源的Web服务框架。
	Jersey， 由Sun提供的JAX-RS的参考实现。
	RESTEasy，JBoss的实现。
	Restlet，由Jerome Louvel和Dave Pawson开发，是最早的REST框架，先于JAX-RS出现。
	Apache Wink，一个Apache软件基金会孵化器中的项目，其服务模块实现JAX-RS规范




## 二、使用dubbox


### 2.1 dubbox 支持REST

dubbox 使用Jboss的resteasy作为rest的实现框架，具体细节请参考jboss rest easy 官网

[亚马逊微架构SOA强行的6个原则](http://www.infoq.com/cn/articles/micro-soa-1)


 （1）**支持各种格式xml json以及中文方式和自定义序列化**
**关于xml的序列化**

	由于JAX-RS的实现一般都用标准的JAXB（Java API for XML Binding）来序列化和反序列化XML格式数据，所以我们需要为每一个要用XML传输的对象添加一个类级别的JAXB annotation，否则序列化将报错。例如为getUser()中返回的User添加如下：@XmlRootElement

但是，java中的基础类型(primitive)不能被序列化为xml的格式，so，最好为它们添加一层wrapper对象

```java
@XmlRootElement
public class RegistrationResult implements Serializable {

    private Long id;

    public RegistrationResult() {
    }

    public RegistrationResult(Long id) {
        this.id = id;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
}
```


这种wrapper对象其实利用所谓Data Transfer Object（DTO）模式，采用DTO还能对传输数据做更多有用的定制。


（2） **支持嵌入式server，例如tomcat和jetty等**
（3） **Dubbo的REST也支持JAX-RS标准的Filter和Interceptor**，以方便对REST的请求与响应过程做定制化的拦截处理。其中，Filter主要用于**访问和设置HTTP请求和响应的参数、URI等等**。还有dubbo支持的自己filter，详情见dubbox官方文档


例如，设置HTTP响应的cache header

```java
public class CacheControlFilter implements ContainerResponseFilter {

    public void filter(ContainerRequestContext req, ContainerResponseContext res) {
        if (req.getMethod().equals("GET")) {
            res.getHeaders().add("Cache-Control", "someValue");
        }
    }
}
```

Interceptor主要用于访问和修改输入与输出字节流，例如，手动添加`GZIP`**压缩**：

```java
public class GZIPWriterInterceptor implements WriterInterceptor {

    @Override
    public void aroundWriteTo(WriterInterceptorContext context)
                    throws IOException, WebApplicationException {
        OutputStream outputStream = context.getOutputStream();
        context.setOutputStream(new GZIPOutputStream(outputStream));
        context.proceed();
    }
}
```












