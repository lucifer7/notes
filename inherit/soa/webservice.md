# WebService


[toc]
## 一、初识webservice

### 1.基础概念
**优势**:
*   **平台无关**。不管你使用什么平台，都可以使用Web service。
*　**编程语言无关**。只要遵守相关协议，就可以使用任意编程语言，向其他网站要求Web service。这大大增加了web service的适用性，降低了对程序员的要求。
*　对于Web service提供者来说，部署、升级和维护Web service都非常单纯，**不需要考虑客户端兼容问题**，而且一次性就能完成。
*　对于Web service使用者来说，可以轻易实现多种数据、多种服务的聚合（mashup），因此能够做出一些以前基础根本无法想像的事情。



###2.第一个程序
**趋势**:
*　在使用方式上，RPC和soap的使用在减少，Restful架构占到了主导地位。
*　在数据格式上，XML格式的使用在减少，json等轻量级格式的使用在增多。
*　在设计架构上，越来越多的第三方软件让用户在客户端（即浏览器），直接与云端对话，不再使用第三方的服务器进行中转或处理数据。


```java
	//定义接口
@WebService
public interface MyService {
    int add(int a, int b);
}

//定义接口实现
@WebService(endpointInterface = "com.carl.demo.webservice.start_01.server.MyService")
public class MyServiceImpl implements MyService {
    public int add(int a, int b) {
        return a+b;
    }
}

//服务端
public class MyServer {
    public static void main(String[] args) {
        String address = "http://localhost:8888/ns";
        Endpoint.publish(address, new MyServiceImpl());
    }
}

//访问http://localhost:8888/ns?wsdl 查看属性

//客户端
public class MyClient {
    public static void main(String[] args) throws Exception {
        URL url = new URL("http://localhost:8888/ns?wsdl");
        //qName 使用的属性targetNamespace和name 创建接口
        QName qName = new QName("http://server.start_01.webservice.demo.carl.com/", "MyServiceImplService");
        Service service = Service.create(url, qName);
        MyService myService = service.getPort(MyService.class);
        int sum = myService.add(1, 2);
        System.out.println(sum);
    }
}
```

>发现以上`client`和`sever`在同一个应用中，实际中，不可能让客户端拥有服务端的接口代码
>因此，需要能根据wsdl直接生成依赖代码的工具

###3.使用wsimport生成接口代码

使用以下命令：
`wsimport -d f://webservice/01/ -keep -verbose http://localhost:8888/ns?wsdl`
具体信息，输入wsimport 可以查看：
1. -keep 是否保留.java文件
2. -verbose 详细信息
3. http://localhost:8888/ns?wsdl 具体的wsdl地址
4. f:/webservice/01/  目录


```java
public class MyClient2 {
    public static void main(String[] args) throws Exception {
        //在MyServiceImplService拥有返回service的方法，具体查看源码
        MyServiceImplService myServiceImplService = new MyServiceImplService();
        MyService myService = myServiceImplService.getMyServiceImplPort();
        int sum = myService.add(1,2);
        System.out.println(sum);
    }
}
```

>这样可以在任何地方使用该接口，不需要服务端的接口代码


----------


### 4.SOAP: Simple object access protocol

**soap协议使用xml作为传输格式，包含以下内容：**
- **types**: 定义的类型
```
<types>
<xsd:schema>
<xsd:import namespace="http://server.start_01.webservice.demo.carl.com/" schemaLocation="http://localhost:8888/ns?xsd=1"/>
</xsd:schema>
</types>
```
- **message**：SOAP 传递的消息
```
<message name="add">
	<part name="parameters" element="tns:add"/>
</message>
<message name="addResponse">
	<part name="parameters" element="tns:addResponse"/>
</message>
```

- **portType**：服务器的接口，通过operation绑定相应的in和out
```
<portType name="MyService">
	<operation name="add">
	<input wsam:Action="http://server.start_01.webservice.demo.carl.com/MyService/addRequest" message="tns:add"/>
	<output wsam:Action="http://server.start_01.webservice.demo.carl.com/MyService/addResponse" message="tns:addResponse"/>
	</operation>
</portType>
```

- **binding：**消息格式，现在多用`literal`
```
<binding name="MyServiceImplPortBinding" type="tns:MyService">
	<soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
		<operation name="add">
	<soap:operation soapAction=""/>
	<input>
		<soap:body use="literal"/>
	</input>
	<output>
	<soap:body use="literal"/>
		</output>
	</operation>
</binding>
```
- **service：指定服务发布的信息**
```
<service name="MyServiceImplService">
	<port name="MyServiceImplPort" binding="tns:MyServiceImplPortBinding">
		<soap:address location="http://localhost:8888/ns"/>
	</port>
</service>
```


----------
### 5. 使用tcpmon工具以及自定义Result和Param标签名字

>可以下载`tcpmon` 工具监听

**?自定义返回值标签名称以及参数名称**
```java
@WebService
public interface MyService {
    @WebResult(name = "addResult") //自定义返回值标签名称以及参数名称
    int add(@WebParam(name = "a") int a, @WebParam(name = "b") int b);
}

```


### 6. 什么是soa？

契约优先还是代码优先？
	异构系统由于有大多的不确定性，无法使用代码优先的设计。

服务的类型：
	基于实体的服务
	基于功能的服务
	基于流程的服务 (业务流程ERP) 工作流程(JBPM)


## 二、xml基础

### 2.1 DTD

[w3c dtd知识](http://www.w3school.com.cn/dtd/dtd_building.asp)

使用idea工具tools->xml action->generate dtd from this xml生成文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE classroom SYSTEM "classroom.dtd">
<classroom id="c1">
    <grade>2010</grade>
    <className>java</className>
    <students>
        <student>
            <id>s1</id>
            <name>aa</name>
            <age>12</age>
        </student>

        <student>
            <id>s2</id>
            <name>bb</name>
            <age>12</age>
        </student>
    </students>
</classroom>
```

### 2.2 Schema

1. 基于xml的语法
2. 可以使用命名空间的方式来解决名称冲突问题

**关键在于理解命名空间:**


自定义xsd
```
<?xml version="1.0" encoding="UTF-8"?>
<xld:schema
        xmlns:ns="http://www.carl.service/01"
        elementFormDefault="qualified" xmlns:xld="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://www.carl.service/01">
    <xld:element name="users">
        <xld:complexType>
            <xld:sequence>
                <xld:element name="user" type="ns:user"></xld:element>
            </xld:sequence>
        </xld:complexType>
    </xld:element>

    <xld:complexType name="user">
        <xld:sequence>
            <xld:element name="id" type="xld:int"></xld:element>
            <xld:element name="username" type="xld:string"></xld:element>
            <xld:element name="birth" type="xld:date"></xld:element>
        </xld:sequence>
    </xld:complexType>

</xld:schema>
```

1. `xmlns:xld="http://www.w3.org/2001/XMLSchema"` 必须有，表示依赖于w3c定义的基础schema
其中，`xmlns:xld` 表示引用这个需要加上**xld****前缀**

2.  `targetNamespace="http://www.carl.service/01"` 表示其他xml或者schema文件引用必须使用的`schemaLocation` 实例如下: `xsi:schemaLocation="http://www.carl.service/01 classroom.xsd"` 其中一个是targetNamespace，一个是实际位置

3. `xmlns:ns="http://www.carl.service/01" ，表示引用的是自身，由于加了前缀，导致，引用自身类型也需要**加上前缀**

4. 编写原则：一个schema就表示一个类别

下面是实例xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<users xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.carl.service/01"
       xsi:schemaLocation="http://www.carl.service/01
        classroom.xsd
       "
>
    <user>
        <id>1</id>
        <username>carl</username>
        <birth>2015-01-01</birth>
    </user>
</users>
```


##  三、 Java与xml

### 3.1 使用JAXB完成对象和xml之间的转换

1. SAX
2. DOM4J
3. alibaba fastxml
4. Xstream --> Stax
5. google xml ...

Hello World：序列化和反序列化
```
 @Test
    public void testMarshaller() throws Exception {
        JAXBContext ctx = JAXBContext.newInstance(User.class);
        Marshaller ms = ctx.createMarshaller();
        User user = new User();
        user.setBirth(new Date());
        user.setInterests(new String[]{"foot", "basket"});
        user.setList(Lists.newArrayList("1", "2", "3"));
        Map<String, String> map = new HashMap<>();
        map.put("1", "a");
        map.put("2", "b");
        user.setMap(map);
        user.setUsername("carl001");
        user.setStudents(Lists.newArrayList(new Student("ccc", 100), new Student("aaa", 200)));
        OutputStream out = new FileOutputStream("user.xml");
        ms.marshal(user, out);
        out.close();
    }

    @Test
    public void testUnMarshaller() throws Exception {
        JAXBContext ctx = JAXBContext.newInstance(User.class);
        Unmarshaller unmarshaller = ctx.createUnmarshaller();
        InputStream in = new FileInputStream("user.xml");
        User u = (User) unmarshaller.unmarshal(in);
        System.out.println(ToStringBuilder.reflectionToString(u));
        in.close();
    }
```

 实体测试类:
```java
package com.carl.demo.webservice;

import javax.xml.bind.annotation.XmlRootElement;
import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * Created by carl on 2016/3/19.
 */
@XmlRootElement
public class User {
    private String username;
    private String[] interests;
    private Date birth;
    private Boolean sex;
    private List<String> list;
    private Map<String, String> map;
    private List<Student> students;

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String[] getInterests() {
        return interests;
    }

    public void setInterests(String[] interests) {
        this.interests = interests;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Boolean getSex() {
        return sex;
    }

    public void setSex(Boolean sex) {
        this.sex = sex;
    }

    public List<String> getList() {
        return list;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    public Map<String, String> getMap() {
        return map;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }
}


```


### 3.2 使用stax 来阅读xml

**一、从w3c 中下载测试文件**
```
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

    <book category="COOKING">
        <title lang="en">Everyday Italian</title>
        <author>Giada De Laurentiis</author>
        <year>2005</year>
        <price>30.00</price>
    </book>

    <book category="CHILDREN">
        <title lang="en">Harry Potter</title>
        <author>J K. Rowling</author>
        <year>2005</year>
        <price>29.99</price>
    </book>

    <book category="WEB">
        <title lang="en">XQuery Kick Start</title>
        <author>James McGovern</author>
        <author>Per Bothner</author>
        <author>Kurt Cagle</author>
        <author>James Linn</author>
        <author>Vaidyanathan Nagarajan</author>
        <year>2003</year>
        <price>49.99</price>
    </book>

    <book category="WEB">
        <title lang="en">Learning XML</title>
        <author>Erik T. Ray</author>
        <year>2003</year>
        <price>39.95</price>
    </book>

</bookstore>
```


**二、使用XMLStreamReader来读取XML**
```java
public void testStreamReader() throws Exception {
        XMLInputFactory factory = XMLInputFactory.newInstance();
        InputStream in = null;
        in = XMLReaderTest.class.getClassLoader().getResourceAsStream("books.xml");
        XMLStreamReader reader = factory.createXMLStreamReader(in);
        while (reader.hasNext()) {
            int type = reader.next();
            String name;
            if (type == XMLStreamConstants.START_ELEMENT) {
                //1. 获取所有开始节点
                //1.1 获取节点名称
                System.out.println("这是开始节点:" + reader.getName());
                name = reader.getName().toString();
                //1.2 获取节点属性
                if (name.equals("book")) {
                    System.out.println("this is book");
                    int count = reader.getAttributeCount();
                    for (int i = 0; i < count; i++) {
                        System.out.println(reader.getAttributeName(i) + ":" + reader.getAttributeValue(i));
                    }
                }
                //1.3 获取节点内容
                if (name.equals("price")) {
                    System.out.println("my price is :" + reader.getElementText());
                }
            } else if (type == XMLStreamConstants.CHARACTERS) {
                //2. 获取所有文本
                System.out.println("这是文本:" + reader.getText());
            } else if (type == XMLStreamConstants.END_ELEMENT) {
                //3. 获取结束节点
                System.out.println("这是结束节点：" + reader.getName());
            }
        }
        in.close();
    }
```

**三、使用XMLEventReader来读取XML**
```java
  @Test
    public void testEventReader() throws Exception {
        XMLInputFactory factory = XMLInputFactory.newInstance();
        InputStream in = null;
        in = XMLReaderTest.class.getClassLoader().getResourceAsStream("books.xml");
        XMLEventReader reader = factory.createXMLEventReader(in);
        while (reader.hasNext()) {
            XMLEvent event = reader.nextEvent();
            //1. 如果是开始节点
            if (event.isStartElement()) {
                StartElement start = event.asStartElement();
                //1.1 获取节点名称
                String nodeName = start.getName().toString();
                if (nodeName.equals("title")) {
                    //1.2 获取节点内容
                    System.out.println(reader.getElementText());
                    //1.3 获取节点属性
                    System.out.println(start.getAttributeByName(new QName("lang")));
                }

                start.getAttributeByName(new QName(""));
            }
        }
        in.close();
    }
```

> 如果该标签没有文本，会出现异常`Message: elementGetText() function expects text only elment but START_ELEMENT was encountered.`

**四、使用 Filter Reader来读取XML**
**五、使用XPath来读取XML**



## 四、SOAP

### 4.1 什么是SOAP协议
Simple Object Access Protocol
简单的对象传输协议，直接看helloworld中的实例(由tcp monitor监听)

**Request Header**
```
POST /ns HTTP/1.1
Accept: text/xml, multipart/related
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://server.start_01.webservice.demo.carl.com/MyService/addRequest"
User-Agent: JAX-WS RI 2.2.9-b130926.1035 svn-revision#5f6196f2b90e9460065a4c2f4e30e065b245e51e
Host: 127.0.0.1:7777
Connection: keep-alive
Content-Length: 212
<?xml version="1.0" ?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Body>
        <ns2:add xmlns:ns2="http://server.start_01.webservice.demo.carl.com/">
            <a>1</a>
            <b>2</b>
        </ns2:add>
    </S:Body>
</S:Envelope>
```

>1. 可以发现soap也是基于http协议，只是消息体是以xml的格式发送(为了方便查看，上述xml都被格式化过)
>2. 以固定的基本格式传输，S:Envelope中Body，还可能有Header等

**Response**
```
HTTP/1.1 200 OK
Date: Mon, 21 Mar 2016 05:03:03 GMT
Transfer-encoding: chunked
Content-type: text/xml; charset=utf-8
ec
<?xml version="1.0" ?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Body>
        <ns2:addResponse xmlns:ns2="http://server.start_01.webservice.demo.carl.com/">
            <addResult>3</addResult>
        </ns2:addResponse>
    </S:Body>
</S:Envelope>
0
```

>跟平时的网页相应类似，只是：Content-type: text/xml; charset=utf-8`



### 4.2 使用SOAP Message底层直接发送请求

**注意**：如果修改targetNameSpace 等信息，则接口和实现类都要修改，深坑勿入

Java官方api支持两种mode发送：`MESSAGE`和`SOURCE`，前一种封装过，后一种直接以原始字符串的方式发送

- **1.以MESSAGE形式发送**
```java
 @Test
    public void testSendSoapMsg() throws Exception {
        // 1. 创建服务
        // 1.1 指定url targetNameSpace
        String targetNameSpace = "http://server.start_01.webservice.demo.carl.com/";
        URL url = new URL("http://localhost:7777/ns?wsdl");
        QName serviceQName = new QName(targetNameSpace, "MyServiceImplService");
        Service service = Service.create(url, serviceQName);


        //2. 创建Dispatch
        Dispatch<SOAPMessage> dispatch = service.createDispatch(
                new QName(targetNameSpace, "MyServiceImplPort"),
                SOAPMessage.class,
                Service.Mode.MESSAGE
        );

        //3. 创建SOAP msg
        MessageFactory fac = MessageFactory.newInstance();
        SOAPMessage message = fac.createMessage();
        SOAPPart part = message.getSOAPPart();
        SOAPEnvelope envelope = part.getEnvelope();
        SOAPBody body = envelope.getBody();
        QName qName = new QName("http://server.start_01.webservice.demo.carl.com/", "add", "ns2");
        SOAPBodyElement ele = body.addBodyElement(qName);
        ele.addChildElement("a").setValue("4");
        ele.addChildElement("b").setValue("2");
        SOAPMessage response = dispatch.invoke(message);
        //4. 将响应解析为文档化对象
        Document document = response.getSOAPBody().extractContentAsDocument();
        NodeList list  = document.getElementsByTagName("addResult");
        System.out.println(list.item(0).getTextContent());
    }
```


**2.以SOURCE形式发送**

```java
 public void testSoapSource() throws Exception {
        // 1. 创建服务
        // 1.1 指定url targetNameSpace
        String targetNameSpace = "http://server.start_01.webservice.demo.carl.com/";
        URL url = new URL("http://localhost:7777/ns?wsdl");
        QName serviceQName = new QName(targetNameSpace, "MyServiceImplService");
        Service service = Service.create(url, serviceQName);


        //2. 创建Dispatch
        Dispatch<Source> dispatch = service.createDispatch(
                new QName(targetNameSpace, "MyServiceImplPort"),
                Source.class,
                Service.Mode.PAYLOAD
        );

        //3. 创建SOAP msg
        String request = "<ns:add xmlns:ns=\"http://server.start_01.webservice.demo.carl.com/\"><a xmlns=\"\">4</a><b xmlns=\"\">2</b></ns:add>";
        StAXSource response = (StAXSource) dispatch.invoke(new StreamSource(new StringReader(request)));
        Transformer transformer = TransformerFactory.newInstance().newTransformer();
        DOMResult domResult = new DOMResult();
        transformer.transform(response, domResult);
        XPath xPath = XPathFactory.newInstance().newXPath();
        NodeList nodeList = (NodeList) xPath.evaluate("//addResult", domResult.getNode(), XPathConstants.NODESET);
        System.out.println(nodeList.item(0).getTextContent());
    }
```


### 4.3 显示传递Header的方式

### 4.4 异常的处理
注意此时在接口中定义了异常UserException
```java
@WebService
public interface MyService {
    @WebResult(name = "addResult")
    int add(@WebParam(name = "a") int a, @WebParam(name = "b") int b);

    @WebResult(name = "users")
    List<User> findAll(@WebParam(name = "authCode") String authCode) throws UserException;
}

@WebService(endpointInterface = "com.carl.demo.webservice.start_01.server.MyService")
public class MyServiceImpl implements MyService {
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public List<User> findAll(String authCode) throws UserException {
        if (authCode.equals("carl")) {
            return Lists.newArrayList(
                    new User("aa", "18", new Date(), "this is for test"),
                    new User("bb", "18", new Date(), "this is for test")
            );
        }else if(authCode.equals("111")){
            throw new RuntimeException("111");
        }
        throw new UserException("some thing happen");
    }
}
```

测试结果**一般异常**如下：
```
<?xml version="1.0" ?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Body>
        <S:Fault xmlns:ns4="http://www.w3.org/2003/05/soap-envelope" xmlns="">
            <faultcode>S:Server</faultcode>
            <faultstring>some thing happen</faultstring>
            <detail>
                <ns2:UserException xmlns:ns2="http://server.start_01.webservice.demo.carl.com/">
                    <message>some thing happen</message>
                </ns2:UserException>
            </detail>
        </S:Fault>
    </S:Body>
</S:Envelope>

```

> 可以看出是通过一个`S:Fault`的标签提供的
**如果是RuntimeException**即是没有捕获到的异常
```
<?xml version="1.0" ?>
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Body>
        <S:Fault xmlns:ns4="http://www.w3.org/2003/05/soap-envelope" xmlns="">
            <faultcode>S:Server</faultcode>
            <faultstring>111</faultstring>
        </S:Fault>
    </S:Body>
</S:Envelope>
```

客户端会抛出`ServerSOAPFaultException`异常



### 4.5 Handler介绍

分为2种，LogicalHandler 和 SOAPHandler，前者只能获取SOAPBody的信息，后者可以获取SOAPMessage的信息，所以往往使用后者,下面**通过handler来增加并且获取SOAPHeader**

**一、首先在客户端使用**

```java
package com.carl.demo.webservice.start_01.server;

import javax.xml.namespace.QName;
import javax.xml.soap.SOAPEnvelope;
import javax.xml.soap.SOAPHeader;
import javax.xml.soap.SOAPMessage;
import javax.xml.ws.handler.MessageContext;
import javax.xml.ws.handler.soap.SOAPHandler;
import javax.xml.ws.handler.soap.SOAPMessageContext;
import java.util.Set;

/**
 * Created by carl on 2016/3/21.
 */
public class HeadHandler implements SOAPHandler<SOAPMessageContext> {
    @Override
    public Set<QName> getHeaders() {
        return null;
    }

    @Override
    public boolean handleMessage(SOAPMessageContext context) {
        Boolean out = (Boolean) context.get(SOAPMessageContext.MESSAGE_OUTBOUND_PROPERTY);
        if (out) {
            try {
                SOAPMessage message = context.getMessage();
                SOAPEnvelope envelope = message.getSOAPPart().getEnvelope();
                SOAPHeader header = envelope.getHeader();
                if (header == null)
                    header = envelope.addHeader();
                QName qName = new QName("http://server.start_01.webservice.demo.carl.com/", "myHeader","ns");
                header.addHeaderElement(qName).setValue("hello myHeader");
            } catch (Exception e) {
            }
        }
        return true;
    }

    @Override
    public boolean handleFault(SOAPMessageContext context) {
        return false;
    }

    @Override
    public void close(MessageContext context) {

    }
}



@WebServiceClient(name = "MyServiceImplService", targetNamespace = "http://server.start_01.webservice.demo.carl.com/", wsdlLocation = "http://localhost:8888/ns?wsdl")
@HandlerChain(file = "handler-chain.xml")
public class MyServiceImplService
        extends Service {
	...
}
```

**二、添加客户端配置文件handler-chain.xml**

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<javaee:handler-chains
        xmlns:javaee="http://java.sun.com/xml/ns/javaee"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <javaee:handler-chain>
        <javaee:handler>
            <javaee:handler-class>com.carl.demo.webservice.start_01.server.HeadHandler</javaee:handler-class>
        </javaee:handler>
    </javaee:handler-chain>
</javaee:handler-chains>
```

**三、服务端增加handler**
```java
package com.carl.demo.webservice.start_01.server;

import com.google.common.collect.Lists;

import javax.jws.HandlerChain;
import javax.jws.WebService;
import java.util.Date;
import java.util.List;

/**
 * Created by carl on 2016/3/17.
 */
//定义接口实现
@WebService(endpointInterface = "com.carl.demo.webservice.start_01.server.MyService")
@HandlerChain(file = "handler-chain.xml")
public class MyServiceImpl implements MyService {
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public List<User> findAll(String authCode) throws UserException {
        System.out.println("进入findAll...方法");
        if (authCode.equals("carl")) {
            return Lists.newArrayList(
                    new User("aa", "18", new Date(), "this is for test"),
                    new User("bb", "18", new Date(), "this is for test")
            );
        }else if(authCode.equals("111")){
            throw new RuntimeException("111");
        }
        throw new UserException("some thing happen");
    }
}


package com.carl.demo.webservice.start_01.server;

import javax.xml.namespace.QName;
import javax.xml.soap.SOAPEnvelope;
import javax.xml.soap.SOAPFault;
import javax.xml.soap.SOAPHeader;
import javax.xml.soap.SOAPMessage;
import javax.xml.ws.handler.MessageContext;
import javax.xml.ws.handler.soap.SOAPHandler;
import javax.xml.ws.handler.soap.SOAPMessageContext;
import javax.xml.ws.soap.SOAPFaultException;
import java.util.Set;

/**
 * Created by carl on 2016/3/21.
 */
public class LicenseHandler implements SOAPHandler<SOAPMessageContext> {
    @Override
    public Set<QName> getHeaders() {
        return null;
    }

    @Override
    public boolean handleMessage(SOAPMessageContext context) {
        // 该参数是服务器端接收客户端请求的操作
        Boolean out = (Boolean) context.get(SOAPMessageContext.MESSAGE_OUTBOUND_PROPERTY);
        if (!out) {
            try {
                SOAPMessage message = context.getMessage();
                SOAPEnvelope envelope = message.getSOAPPart().getEnvelope();
                SOAPHeader header = envelope.getHeader();
                try {
                    String str = header.getElementsByTagName("ns:myHeader").item(0).getTextContent();
                    System.out.println(str);
                } catch (Exception e) {
                    SOAPFault fault = message.getSOAPBody().addFault();
                    fault.setFaultString("头部信息不能为空");
                    throw new SOAPFaultException(fault);
                }
            } catch (Exception e) {
                e.printStackTrace();
                if (e instanceof SOAPFaultException)
                    throw (SOAPFaultException) e;
            }
        }
        return true;
    }

    @Override
    public boolean handleFault(SOAPMessageContext context) {
        return false;
    }

    @Override
    public void close(MessageContext context) {

    }
}

```

**四、服务端增加配置文件**
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<javaee:handler-chains
        xmlns:javaee="http://java.sun.com/xml/ns/javaee"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <javaee:handler-chain>
        <javaee:handler>
            <javaee:handler-class>com.carl.demo.webservice.start_01.server.LicenseHandler</javaee:handler-class>
        </javaee:handler>
    </javaee:handler-chain>
</javaee:handler-chains>
```

### 4.6 契约优先
下面写一个测试 `mathService`来说明这个问题，遇到很多麻烦:

- 没有在自定义实现类`MathServiceImpl`中写清楚所有的包括`serviceName`,`portName `等等

- 没有完全拷贝生成代码的包路径到client下，导致错误信息`两个类具有相同的 XML 类型名称`

一般的开发流程如以下步骤所示：
**一、先写schema或者wsdl文件**:
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<wsdl:definitions
        xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
        xmlns:tns="http://service.carl.math.cn"
        xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        name="MathServiceImpl"
        targetNamespace="http://service.carl.math.cn">

    <!-- wrappered推荐使用的方式 -->
    <!--1.定义信息中引用的元素类型-->
    <wsdl:types>
        <xsd:schema targetNamespace="http://service.carl.math.cn">
            <!--1.1 定义元素的名字，和包装类型-->
            <xsd:element name="add" type="tns:add"/>
            <xsd:element name="addResponse" type="tns:addResponse"/>
            <xsd:element name="minus" type="tns:minus"></xsd:element>
            <xsd:element name="minusResponse" type="tns:minusResponse"></xsd:element>

            <!--1.2 定义被元素引用的包装类型-->
            <xsd:complexType name="add">
                <xsd:sequence>
                    <!--定义每个参数的名字和类型-->
                    <xsd:element name="a" type="xsd:int"></xsd:element>
                    <xsd:element name="b" type="xsd:int"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="addResponse">
                <xsd:sequence>
                    <!--定义每个参数的名字和类型-->
                    <xsd:element name="addResult" type="xsd:int"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="minus">
                <xsd:sequence>
                    <!--定义每个参数的名字和类型-->
                    <xsd:element name="a" type="xsd:int"></xsd:element>
                    <xsd:element name="b" type="xsd:int"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="minusResponse">
                <xsd:sequence>
                    <!--定义每个参数的名字和类型-->
                    <xsd:element name="minusResult" type="xsd:int"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>
        </xsd:schema>
    </wsdl:types>

    <!--2.编写消息-->
    <!--2.1 add方法要输入的消息-->
    <wsdl:message name="add">
        <wsdl:part name="parameters" element="tns:add"/>
    </wsdl:message>
    <!--2.2 add方法要输出的消息-->
    <wsdl:message name="addResponse">
        <wsdl:part name="parameters" element="tns:addResponse"/>
    </wsdl:message>
    <wsdl:message name="minus">
        <wsdl:part name="parameters" element="tns:minus"/>
    </wsdl:message>
    <wsdl:message name="minusResponse">
        <wsdl:part name="parameters" element="tns:minusResponse"/>
    </wsdl:message>


    <!--3.编写服务器要接口方法-->
    <wsdl:portType name="MathService">
        <!--3.1 add方法-->
        <wsdl:operation name="add">
            <wsdl:input message="tns:add"></wsdl:input>
            <wsdl:output message="tns:addResponse"></wsdl:output>
        </wsdl:operation>

        <wsdl:operation name="minus">
            <wsdl:input message="tns:minus"></wsdl:input>
            <wsdl:output message="tns:minusResponse"></wsdl:output>
        </wsdl:operation>
    </wsdl:portType>

    <!--4.编写服务器消息格式-->
    <wsdl:binding name="MathServiceImplPortBinding" type="tns:MathService">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
        <wsdl:operation name="add">
            <wsdl:input>
                <soap:body use="literal"/>
            </wsdl:input>
            <wsdl:output>
                <soap:body use="literal"/>
            </wsdl:output>
        </wsdl:operation>
        <wsdl:operation name="minus">
            <wsdl:input>
                <soap:body use="literal"/>
            </wsdl:input>
            <wsdl:output>
                <soap:body use="literal"/>
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>

    <!--5.编写服务器接口实现类方法-->
    <wsdl:service name="MathServiceImpl">
        <wsdl:port name="MyServiceImplPort" binding="tns:MathServiceImplPortBinding">
            <soap:address location="http://localhost:8888/ns"/>
        </wsdl:port>
    </wsdl:service>

</wsdl:definitions>

```

**二、** 根据文件生成客户端代码

**三、** 编写实现类
```java
package com.carl.demo.webservice;

import javax.jws.WebService;

/**
 * Created by carl on 2016/3/21.
 * 下面标注中的所有内容必须跟wsdl文件上描述的一致
 */
@WebService(
        serviceName = "MathServiceImpl",
        endpointInterface = "com.carl.demo.webservice.MathService",
        targetNamespace = "http://service.carl.math.cn",
        portName = "MyServiceImplPort",
        wsdlLocation = "math.wsdl"
)
public class MathServiceImpl implements MathService {
    @Override
    public int add(int a, int b) {
        System.out.println("进入add方法....");
        return a + b;
    }

    @Override
    public int minus(int a, int b) {
        System.out.println("进入minus方法....");
        return a - b;
    }
}

```

**四、** 发布服务到tomcat



### 4.7 如何与tomcat 集成
**一、配置依赖jar包**
依赖jar包
```
<dependency>
            <groupId>com.sun.xml.ws</groupId>
            <artifactId>jaxws-rt</artifactId>
            <version>2.2.6</version>
        </dependency>
        <dependency>
            <groupId>org.jvnet.jax-ws-commons.spring</groupId>
            <artifactId>jaxws-spring</artifactId>
            <version>1.8</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

**二、WEB-INF下配置sun-jaxws.xml文件**

```
<?xml version="1.0" encoding="UTF-8"?>
<endpoints xmlns="http://java.sun.com/xml/ns/jax-ws/ri/runtime" version="2.0">
  <endpoint name="MathServiceImpl"
       implementation="com.carl.demo.math.MathServiceImpl" url-pattern="/ms"/>
</endpoints>
```


**三、配置web.xml文件**
```
  <listener>
        <listener-class>com.sun.xml.ws.transport.http.servlet.WSServletContextListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>MathServiceImpl</servlet-name>
        <servlet-class>com.sun.xml.ws.transport.http.servlet.WSServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>MathServiceImpl</servlet-name>
        <url-pattern>/ms</url-pattern>
    </servlet-mapping>
```


**四、wsdl**
	在WEB-INF下建立wsdl文件夹，将wsdl文件放入，并且配置地址
	必须有wsdl文件夹

```java
@WebService(targetNamespace = "http://math.demo.carl.com",
        serviceName = "MathServiceImpl",
        portName = "MyServicePort",
        endpointInterface = "com.carl.demo.math.MathService",
//        wsdlLocation = "WEB-INF/wsdl/math.wsdl"
        wsdlLocation = "math.wsdl"
)
public class MathServiceImpl implements MathService {
    @Override
    public int add(int a, int b) throws UserException {
        if (a < 0 && b < 0) {
            throw new UserException("test UserException");
        }
        return a + b;
    }

    @Override
    public int minus(int a, int b) {
        return a - b;
    }
}
```


### 4.8 如何与 spring 集成

**一、配置spring配置文件**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:ws="http://jax-ws.dev.java.net/spring/core"
       xmlns:wss="http://jax-ws.dev.java.net/spring/servlet"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.0.xsd
           http://jax-ws.dev.java.net/spring/core http://jax-ws.dev.java.net/spring/core.xsd
    	   http://jax-ws.dev.java.net/spring/servlet http://jax-ws.dev.java.net/spring/servlet.xsd">
    <!-- <bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
        <property name="baseAddress" value="http://localhost:8888/ss/"/>
    </bean> -->
    <!-- 通过以下代码可以有效的让studentWsService被spring说管理 -->

    <bean class="com.carl.demo.math.MathServiceImpl" name="mathService"></bean>
    <wss:binding url="/ms">
        <wss:service>
            <!-- bean中表示了webservice的注入对象，注意在这个里面需要在开头增加一个# -->
            <ws:service bean="#mathService">
                <!-- 把wsdl中的外部文件加入 -->
                <!--<ws:metadata>
                    <value>/WEB-INF/wsdl/student.xsd</value>
                </ws:metadata>-->
            </ws:service>
        </wss:service>
    </wss:binding>

</beans>
```


**二、配置web.xml文件**

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         id="WebApp_ID" version="2.5">
  <!--  <listener>
        <listener-class>com.sun.xml.ws.transport.http.servlet.WSServletContextListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>MathServiceImpl</servlet-name>
        <servlet-class>com.sun.xml.ws.transport.http.servlet.WSServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>MathServiceImpl</servlet-name>
        <url-pattern>/ms</url-pattern>
    </servlet-mapping>-->


    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-context.xml</param-value>
    </context-param>


    <servlet>
        <servlet-name>MathServiceImpl</servlet-name>
        <servlet-class>com.sun.xml.ws.transport.http.servlet.WSSpringServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>MathServiceImpl</servlet-name>
        <url-pattern>/ms</url-pattern>
    </servlet-mapping>


    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>


</web-app>
```

**注意事项：**
**1.unqualified 表示返回response不加前缀**

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<definitions
        xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
        xmlns:tns="http://math.demo.carl.com"
        xmlns="http://schemas.xmlsoap.org/wsdl/"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        name="MathServiceImpl"
        targetNamespace="http://math.demo.carl.com">
    <!--1.types-->
    <types>
	    <!--unqualified 表示返回response不加前缀-->
        <xsd:schema targetNamespace="http://math.demo.carl.com" elementFormDefault="unqualified">
            <xsd:element name="add" type="tns:add"></xsd:element>
            <xsd:element name="addResponse" type="tns:addResponse"></xsd:element>
            <xsd:element name="hello" type="tns:hello"></xsd:element>
            <xsd:element name="helloResponse" type="tns:helloResponse"></xsd:element>
            <xsd:element name="auth" type="xsd:string"></xsd:element>
            <xsd:element name="UserException" type="tns:UserException"></xsd:element>
            <xsd:element name="findAll" type="tns:findAll"></xsd:element>
            <xsd:element name="findAllResponse" type="tns:findAllResponse"></xsd:element>


            <xsd:complexType name="findAll">
                <xsd:sequence/>
            </xsd:complexType>

            <xsd:complexType name="findAllResponse">
                <xsd:sequence>
                    <xsd:element name="users" type="tns:user" maxOccurs="unbounded"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <!--定义User对象-->
            <xsd:complexType name="user">
                <xsd:sequence>
                    <xsd:element name="age" type="xsd:string"/>
                    <xsd:element name="createTime" type="xsd:dateTime"/>
                    <xsd:element name="info" type="xsd:string"/>
                    <xsd:element name="username" type="xsd:string"/>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="UserException">
                <xsd:sequence>
                    <xsd:element type="xsd:string" name="message"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>
            <xsd:complexType name="add">
                <xsd:sequence>
                    <xsd:element name="a" type="xsd:int"></xsd:element>
                    <xsd:element name="b" type="xsd:int"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="addResponse">
                <xsd:sequence>
                    <xsd:element name="addResult" type="xsd:int"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="hello">
                <xsd:sequence>
                    <xsd:element name="username" type="xsd:string"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="helloResponse">
                <xsd:sequence/>
            </xsd:complexType>
        </xsd:schema>
    </types>

    <!--2. messages-->
    <message name="UserException">
        <part name="fault" element="tns:UserException"></part>
    </message>
    <message name="findAllResponse">
        <part name="parameters" element="tns:findAllResponse"></part>
    </message>
    <message name="findAll">
        <part name="parameters" element="tns:findAll"></part>
    </message>
    <message name="add">
        <part name="parameters" element="tns:add"/>
    </message>
    <message name="addResponse">
        <part name="parameters" element="tns:addResponse"/>
    </message>
    <message name="hello">
        <part name="parameters" element="tns:hello"/>
    </message>
    <message name="helloResponse">
        <part name="parameters" element="tns:helloResponse"/>
    </message>
    <message name="auth">
        <part name="auth" element="tns:auth"></part>
    </message>

    <!--3. port-->
    <portType name="MathService">
        <operation name="add">
            <input message="tns:add"></input>
            <output message="tns:addResponse"></output>
            <fault name="UserException" message="tns:UserException"></fault>
        </operation>
        <operation name="hello">
            <input message="tns:hello"></input>
            <output message="tns:helloResponse"></output>
        </operation>
        <operation name="findAll">
            <input message="tns:findAll"></input>
            <output message="tns:findAllResponse"></output>
        </operation>
    </portType>

    <!--4.binding-->
    <binding name="MathServiceImplPortBinding" type="tns:MathService">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>

        <operation name="add">
            <input>
                <soap:body use="literal"/>
                <soap:header message="tns:auth" part="auth" use="literal"></soap:header>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
            <fault name="UserException">
                <soap:fault name="UserException" use="literal"></soap:fault>
            </fault>
        </operation>

        <operation name="hello">
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>

        <operation name="findAll">
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
    </binding>

    <!--5.-->
    <service name="MathServiceImpl">
        <port name="MathServicePort" binding="tns:MathServiceImplPortBinding">
            <soap:address location="http://localhost:9080/soa/ms"/>
        </port>
    </service>
</definitions>

```


**2.若要使用/WEB-INF下的配置文件， 在根目录前加/**
```
@WebService(
        endpointInterface = "com.carl.demo.math.MathService",
        wsdlLocation = "/WEB-INF/wsdl/math.wsdl",
        serviceName = "MathServiceImpl",
        portName = "MathServicePort",
        targetNamespace = "http://math.demo.carl.com"

)
```


**3.如何在client端和spring整合**

```
   <bean id="mathService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
        <property name="serviceInterface" value="com.carl.demo.math.MathService"/>
        <property name="wsdlDocumentUrl" value="http://localhost:9080/soa/ms?wsdl"/>
        <property name="namespaceUri" value="http://math.demo.carl.com"/>
        <property name="serviceName" value="MathServiceImpl"/>
        <property name="portName" value="MathServicePort"/>
    </bean>
```


**4.spring 在客户端和handler 整合**

```java
	package com.carl.demo.math.handler;

import javax.xml.namespace.QName;
import javax.xml.soap.SOAPBody;
import javax.xml.soap.SOAPEnvelope;
import javax.xml.soap.SOAPHeader;
import javax.xml.ws.handler.MessageContext;
import javax.xml.ws.handler.soap.SOAPHandler;
import javax.xml.ws.handler.soap.SOAPMessageContext;
import java.util.Set;

/**
 * Created by carl on 2016/3/24.
 */
public class MyHandler implements SOAPHandler<SOAPMessageContext> {
    @Override
    public Set<QName> getHeaders() {
        return null;
    }

    @Override
    public boolean handleMessage(SOAPMessageContext ctx) {
        Boolean out = (Boolean) ctx.get(SOAPMessageContext.MESSAGE_OUTBOUND_PROPERTY);
        try {
            if (!out) {
                String targetNameSpace = "http://math.demo.carl.com";
                SOAPEnvelope enve = ctx.getMessage().getSOAPPart().getEnvelope();
                SOAPBody body = enve.getBody();
                String name = body.getFirstChild().getLocalName();
                System.out.println("name:" + name);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
        return true;
    }

    @Override
    public boolean handleFault(SOAPMessageContext context) {
        return false;
    }

    @Override
    public void close(MessageContext context) {

    }
}


package com.carl.demo.math.handler;

import javax.xml.ws.handler.Handler;
import javax.xml.ws.handler.HandlerResolver;
import javax.xml.ws.handler.PortInfo;
import java.util.ArrayList;
import java.util.List;

/**
 * Created by carl on 2016/3/24.
 */
public class MySOAPHandlerResover implements HandlerResolver {

    private List<Handler> handlers;

    public MySOAPHandlerResover() {
        handlers = new ArrayList<Handler>();
        handlers.add(new MyHandler());
    }

    public List<Handler> getHandlerChain(PortInfo arg0) {
        return handlers;
    }
}

```

xml 注入依赖bean
```
	 <bean id="mySOAPHandlerResover" class="com.carl.demo.math.handler.MySOAPHandlerResover"></bean>
    <bean id="mathService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
        <property name="serviceInterface" value="com.carl.demo.math.MathService"/>
        <property name="wsdlDocumentUrl" value="http://localhost:9080/soa/ms?wsdl"/>
        <property name="namespaceUri" value="http://math.demo.carl.com"/>
        <property name="serviceName" value="MathServiceImpl"/>
        <property name="portName" value="MathServicePort"/>
        <property name="handlerResolver" ref="mySOAPHandlerResover" />
    </bean>
```



## 五、CXF
Apache 的一个实现



### 5.1 依赖包及插件

首先下载依赖包
```
   <cxf.version>2.6.1</cxf.version>
    <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-frontend-jaxws</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http</artifactId>
            <version>${cxf.version}</version>
        </dependency>

        <!-- Jetty is needed if you're are not using the CXFServlet -->
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http-jetty</artifactId>
            <version>${cxf.version}</version>
        </dependency>
```


CXF 有一个类似于java `wsimport`的插件，功能上应该更加强大
```
  <plugin>
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-codegen-plugin</artifactId>
                <version>${cxf.version}</version>
                <executions>
                    <execution>
                        <id>generate-sources</id>
                        <phase>generate-sources</phase>
                        <configuration>
                            <sourceRoot>${project.build.directory}/cxf</sourceRoot>
                            <wsdlOptions>
                                <wsdlOption>
                                    <wsdl>http://localhost:8878/ms?wsdl</wsdl>
                                    <extraargs>
                                        <extraarg>-impl</extraarg>
                                        <extraarg>-verbose</extraarg>
                                    </extraargs>
                                </wsdlOption>
                            </wsdlOptions>
                        </configuration>
                        <goals>
                            <goal>wsdl2java</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

### 5.2 开发契约

契约优先，开发xml

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<definitions
        xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
        xmlns:tns="http://cxf.demo.carl.com"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns="http://schemas.xmlsoap.org/wsdl/"
        targetNamespace="http://cxf.demo.carl.com"
        name="UserRemoteServiceImpl">
    <!--1.types-->
    <types>
        <xsd:schema targetNamespace="http://cxf.demo.carl.com" elementFormDefault="unqualified">
            <xsd:element name="findAll" type="tns:findAll"/>
            <xsd:element name="findAllResponse" type="tns:findAllResponse"/>
            <xsd:element name="add" type="tns:add"/>
            <xsd:element name="addResponse" type="tns:addResponse"/>
            <!--自定义头部全局验证信息-->
            <xsd:element name="auth" type="tns:auth"></xsd:element>
            <!--自定义异常-->
            <xsd:element name="UserException" type="tns:UserException"></xsd:element>
            <xsd:element name="del" type="tns:del"></xsd:element>
            <xsd:element name="delResponse" type="tns:delResponse"></xsd:element>

            <xsd:complexType name="del">
                <xsd:sequence>
                    <xsd:element name="username" type="xsd:string"></xsd:element>
                    <xsd:element name="pwd" type="xsd:string"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>
            <xsd:complexType name="delResponse">
                <xsd:sequence/>
            </xsd:complexType>

            <xsd:complexType name="UserException">
                <xsd:sequence>
                    <xsd:element name="message" type="xsd:string"/>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="add">
                <xsd:sequence>
                    <xsd:element name="user" type="tns:user"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>
            <xsd:complexType name="addResponse">
                <xsd:sequence/>
            </xsd:complexType>


            <xsd:complexType name="auth">
                <xsd:sequence>
                    <xsd:element name="accountName" type="xsd:string"></xsd:element>
                    <xsd:element name="accountPwd" type="xsd:string"></xsd:element>
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="findAll">
                <xsd:sequence/>
            </xsd:complexType>

            <xsd:complexType name="findAllResponse">
                <xsd:sequence>
                    <xsd:element name="users" type="tns:user" minOccurs="0" maxOccurs="unbounded"/>
                </xsd:sequence>
            </xsd:complexType>
            <xsd:complexType name="user">
                <xsd:sequence>
                    <xsd:element name="id" type="xsd:int" minOccurs="0"/>
                    <xsd:element name="info" type="xsd:string" minOccurs="0"/>
                    <xsd:element name="password" type="xsd:string" minOccurs="0"/>
                    <xsd:element name="username" type="xsd:string" minOccurs="0"/>
                </xsd:sequence>
            </xsd:complexType>
        </xsd:schema>
    </types>

    <!--2.message-->
    <message name="findAllResponse">
        <part name="parameters" element="tns:findAllResponse"></part>
    </message>
    <message name="findAll">
        <part name="parameters" element="tns:findAll"></part>
    </message>
    <message name="add">
        <part name="parameters" element="tns:add"></part>
    </message>
    <message name="addResponse">
        <part name="parameters" element="tns:addResponse"></part>
    </message>
    <message name="del">
        <part name="parameters" element="tns:del"></part>
    </message>
    <message name="delResponse">
        <part name="parameters" element="tns:delResponse"></part>
    </message>

    <message name="auth">
        <part name="auth" element="tns:auth"></part>
    </message>
    <message name="UserException">
        <part name="UserException" element="tns:UserException"></part>
    </message>

    <!--3.port-->
    <portType name="UserRemoteService">
        <operation name="findAll">
            <input message="tns:findAll"></input>
            <output message="tns:findAllResponse"></output>
        </operation>
        <operation name="add">
            <input message="tns:add"></input>
            <output message="tns:addResponse"></output>
        </operation>
        <operation name="del">
            <input message="tns:del"></input>
            <output message="tns:delResponse"></output>
            <fault name="UserException" message="tns:UserException"></fault>
        </operation>
    </portType>

    <!--4.binding-->
    <binding name="UserRemoteServicePortBinding" type="tns:UserRemoteService">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
        <operation name="findAll">
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
        <operation name="add">
            <input>
                <soap:body use="literal"/>
                <soap:header message="tns:auth" part="auth" use="literal"></soap:header>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
        <operation name="del">
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
            <fault name="UserException">
                <soap:fault name="UserException" use="literal"></soap:fault>
            </fault>
        </operation>
    </binding>

    <!--5.service-->
    <service name="UserRemoteServiceImpl">
        <port name="UserRemoteServicePort" binding="tns:UserRemoteServicePortBinding">
            <soap:address location="http://localhost:9080/soa/us"/>
        </port>
    </service>

</definitions>
```

### 5.3 根据契约开发实现类:

```
 <plugin>
                  <groupId>org.apache.cxf</groupId>
                  <artifactId>cxf-codegen-plugin</artifactId>
                  <version>${cxf.version}</version>
                  <executions>
                      <execution>
                          <id>generate-sources</id>
                          <phase>compile</phase>
                          <configuration>
                              <sourceRoot>${project.build.directory}/cxf</sourceRoot>
                              <wsdlOptions>
                                  <wsdlOption>
                                      <wsdl>src/main/resources/wsdl/user.wsdl</wsdl>
                                      <extraargs>
                                          <extraarg>-verbose</extraarg>
                                      </extraargs>
                                  </wsdlOption>
                              </wsdlOptions>
                          </configuration>
                          <goals>
                              <goal>wsdl2java</goal>
                          </goals>
                      </execution>
                  </executions>
              </plugin>
```


删除className等等信息:
```java
package com.carl.demo.cxf;

import com.carl.demo.vo.User;

import javax.jws.WebMethod;
import javax.jws.WebParam;
import javax.jws.WebResult;
import javax.jws.WebService;
import javax.xml.ws.RequestWrapper;
import javax.xml.ws.ResponseWrapper;

/**
 * This class was generated by Apache CXF 2.6.1
 * 2016-03-27T23:39:21.605+08:00
 * Generated source version: 2.6.1
 */
@WebService(targetNamespace = "http://cxf.demo.carl.com", name = "UserRemoteService")
public interface UserRemoteService {

    @WebMethod
    @RequestWrapper(localName = "del", targetNamespace = "http://cxf.demo.carl.com")
    @ResponseWrapper(localName = "delResponse", targetNamespace = "http://cxf.demo.carl.com")
    public void del(
            @WebParam(name = "username", targetNamespace = "")
            java.lang.String username,
            @WebParam(name = "pwd", targetNamespace = "")
            java.lang.String pwd
    ) throws UserException;

    @WebMethod
    @RequestWrapper(localName = "add", targetNamespace = "http://cxf.demo.carl.com")
    @ResponseWrapper(localName = "addResponse", targetNamespace = "http://cxf.demo.carl.com")
    public void add(
            @WebParam(name = "user", targetNamespace = "")
            User user
    );

    @WebMethod
    @RequestWrapper(localName = "findAll", targetNamespace = "http://cxf.demo.carl.com")
    @ResponseWrapper(localName = "findAllResponse", targetNamespace = "http://cxf.demo.carl.com")
    @WebResult(name = "users", targetNamespace = "")
    public java.util.List<User> findAll();
}


// 实现类如下----------------------------------------
package com.carl.demo.cxf;

import com.carl.demo.service.UserService;
import com.carl.demo.vo.User;
import org.apache.commons.lang.builder.ToStringBuilder;
import org.springframework.beans.factory.annotation.Autowired;

import javax.jws.WebService;
import java.util.List;

/**
 * Created by carl on 2016/3/27.
 */
@WebService
        (
                endpointInterface = "com.carl.demo.cxf.UserRemoteService",
                targetNamespace = "http://cxf.demo.carl.com",
                portName = "UserRemoteServicePortBinding",
                serviceName = "UserRemoteServiceImpl",
                wsdlLocation = "/WEB-INF/wsdl/user.wsdl"
        )
public class UserRemoteServiceImpl implements UserRemoteService {
    @Autowired
    private UserService userService;

    @Override
    public List<User> findAll() {
        return userService.findAll();
    }

    @Override
    public void del(String username, String pwd) throws UserException {
        User u = userService.findOne(username);
        if (u == null)
            throw new UserException("用户名不存在");
        if (!u.getPassword().equals(pwd))
            throw new UserException("密码不存在");
        System.out.println("删除成功");
    }

    @Override
    public void add(User user) {
        System.out.println("要增加的用户：" + ToStringBuilder.reflectionToString(user));
    }
}

```


### 5.4 使用spring配置到web容器中

**web.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         id="WebApp_ID" version="2.5">

    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

    <servlet>
        <servlet-name>CXFServlet</servlet-name>
        <servlet-class>
            org.apache.cxf.transport.servlet.CXFServlet
        </servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>CXFServlet</servlet-name>
        <url-pattern>/services/*</url-pattern>
    </servlet-mapping>

    <!-- Spring配置文件开始  -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:spring-context.xml
        </param-value>
    </context-param>



    <!-- Spring配置文件结束 -->

    <!-- 设置servlet编码开始 -->
    <filter>
        <filter-name>Set Character Encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>Set Character Encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

**spring配置文件**
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.0.xsd
    	   http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
    <import resource="classpath:META-INF/cxf/cxf.xml"/>
    <import resource="classpath:META-INF/cxf/cxf-extension-soap.xml"/>
    <import resource="classpath:META-INF/cxf/cxf-servlet.xml"/>


    <bean class="com.carl.demo.cxf.UserRemoteServiceImpl" id="userRemoteService"></bean>

    <!--定义interceptors-->
    <bean id="authInterceptor" class="com.carl.demo.cxf.interceptor.AuthInterceptor"></bean>

    <!--JAXWS 的方式-->
    <!-- <jaxws:endpoint id="sws"
                     implementor="#userRemoteService"
                     address="/us"/>-->

    <jaxws:server serviceClass="com.carl.demo.cxf.UserRemoteService" address="/us">
        <jaxws:serviceBean>
            <ref bean="userRemoteService"></ref>
        </jaxws:serviceBean>

        <jaxws:inInterceptors>
            <bean class="org.apache.cxf.interceptor.LoggingInInterceptor"/>
            <ref bean="authInterceptor"></ref>
        </jaxws:inInterceptors>

        <jaxws:outInterceptors>
            <bean class="org.apache.cxf.interceptor.LoggingOutInterceptor"/>
        </jaxws:outInterceptors>
    </jaxws:server>

</beans>
```



### 5.6 开发client端和spring整合


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xmlns:cxf="http://cxf.apache.org/core"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.0.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
           http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd
           http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">


    <jaxws:client id="userRemoteService"
                  serviceClass="com.carl.demo.cxf.UserRemoteService"
                  address="http://localhost:9080/soa/services/us?wsdl">
        <jaxws:outInterceptors>
            <bean class="com.carl.demo.cxf.handler.AuthOutInterceptor"></bean>
        </jaxws:outInterceptors>
    </jaxws:client>
</beans>
```


### 5.7 整合时用到的Interceptor

```
package com.carl.demo.cxf.handler;

import com.carl.demo.cxf.Auth;
import org.apache.cxf.binding.soap.SoapMessage;
import org.apache.cxf.binding.soap.interceptor.AbstractSoapInterceptor;
import org.apache.cxf.databinding.DataBinding;
import org.apache.cxf.headers.Header;
import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.jaxb.JAXBDataBinding;
import org.apache.cxf.phase.Phase;

import javax.xml.bind.JAXBException;
import javax.xml.namespace.QName;
import java.util.List;

/**
 * Created by carl on 2016/3/27.
 */
public class AuthOutInterceptor extends AbstractSoapInterceptor {
    public AuthOutInterceptor() {
        super(Phase.WRITE);
    }


    @Override
    public void handleMessage(SoapMessage message) throws Fault {
        List<Header> headers = message.getHeaders();
        QName qn = new QName(CxfProperties.TARGET_NAME_SPACE, "auth", CxfProperties.QNAME_PREFIX);
        try {
            DataBinding binding = new JAXBDataBinding(Auth.class);
            Auth auth = new Auth();
            auth.setAccountName("root");
            auth.setAccountPwd("123456");
            Header header = new Header(qn, auth, binding);
            headers.add(header);
        } catch (JAXBException e) {
            e.printStackTrace();
        }
    }
}

```


```
package com.carl.demo.cxf.handler;

import com.carl.demo.cxf.Auth;
import org.apache.cxf.binding.soap.SoapMessage;
import org.apache.cxf.binding.soap.interceptor.AbstractSoapInterceptor;
import org.apache.cxf.databinding.DataBinding;
import org.apache.cxf.headers.Header;
import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.jaxb.JAXBDataBinding;
import org.apache.cxf.phase.Phase;

import javax.xml.bind.JAXBException;
import javax.xml.namespace.QName;
import java.util.List;

/**
 * Created by carl on 2016/3/27.
 */
public class AuthOutInterceptor extends AbstractSoapInterceptor {
    public AuthOutInterceptor() {
        super(Phase.WRITE);
    }


    @Override
    public void handleMessage(SoapMessage message) throws Fault {
        List<Header> headers = message.getHeaders();
        QName qn = new QName(CxfProperties.TARGET_NAME_SPACE, "auth", CxfProperties.QNAME_PREFIX);
        try {
            DataBinding binding = new JAXBDataBinding(Auth.class);
            Auth auth = new Auth();
            auth.setAccountName("root");
            auth.setAccountPwd("123456");
            Header header = new Header(qn, auth, binding);
            headers.add(header);
        } catch (JAXBException e) {
            e.printStackTrace();
        }
    }
}

```