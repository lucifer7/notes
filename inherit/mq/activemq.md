# Active mq
[TOC]

##  文档
[api官方地址](http://docs.oracle.com/javaee/1.4/api/javax/jms/package-summary.html)

## JMS: Java Message Service
**一、基本概念**:
	1. JMS provider : 实现JMS接口和规范的消息中间件
	2. JMS message:
			a：消息头
			b：消息属性
			c：消息体
	3. JMS producer
	4. JMS consumer
	5. JMS domains:  消息传送域，PTP和pub/sub

**二、一个完整的JMS 消息包含以下部分**
1. **消息头**:包含消息的识别信息和路由信息
	`JMSDestination`
	`JMSDeliveryMode`
	`JMSExpiration`
	`JMSPriority`
	`JMSMessageID`
	`JMSTimestamp`
	`JMSCorrelationID`
	`JMSReplyTo`
	`JMSType`
	`JMSRedelivered`
2. **消息体:** `TextMessage`、`MapMessage`、
`BytesMessage`、`StreamMessage`和`ObjectMessage`
3. **属性**
	- 自定义属性
	- JMS属性
	- 供应商属性

三、HelloWorld
```java
package com.carl.demo.activemq.helloworld_001;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

/**
 * Created by carl on 2016/3/6.
 */
public class Sender {
    public static final String URI = "tcp://192.168.0.135:61616";

    public static void main(String[] args) throws Exception {
        ConnectionFactory connectionFactory = new
                ActiveMQConnectionFactory(URI);
        Connection connection = connectionFactory.createConnection();
        connection.start();
        // Boolean.TRUE indicates whether the session is transacted
        Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
        Destination destination = session.createQueue("carl-queue-001");
        MessageProducer producer = session.createProducer(destination);
        for (int i = 0; i < 3; i++) {
            TextMessage message = session.createTextMessage("message--" + i);
            message.setStringProperty("name", "carl00" + i);
            Thread.sleep(1000);
            //通过消息生产者发出消息
            producer.send(message);
        }
        session.commit();
        session.close();
        connection.close();
    }
}


package com.carl.demo.activemq.helloworld_001;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;
import java.util.Enumeration;

/**
 * Created by carl on 2016/3/6.
 */
public class Receiver {
    public static void main(String[] args) throws Exception {
        ConnectionFactory connectionFactory = new
                ActiveMQConnectionFactory(Sender.URI);
        Connection connection = connectionFactory.createConnection();
        connection.start();
        Enumeration jmsPropertyNames = connection.getMetaData().getJMSXPropertyNames();
        while (jmsPropertyNames.hasMoreElements()) {
            String property = (String) jmsPropertyNames.nextElement();
            System.out.println(property);
        }
        //设定自动签收
        Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
        Destination destination = session.createQueue("carl-queue-001");
        MessageConsumer consumer = session.createConsumer(destination);
        int count = 3;
        for (int i = 0; i < count; i++) {
            TextMessage message = (TextMessage) consumer.receive();
            System.out.println(message.getText() + ",name:" + message.getStringProperty("name"));
        }
        session.commit();
        session.close();
        connection.close();
    }
}

```

## JMS的可靠性机制

**一、消息的接收和确认**
JMS的会话分为事务性会话和非事务性会话，由connection在createSession时候设置，根据api
```java
public Session createSession(boolean transacted,
                             int acknowledgeMode)
                      throws JMSException
Creates a Session object.
Parameters:
transacted - indicates whether the session is transacted
acknowledgeMode - indicates whether the consumer or the client will acknowledge any messages it receives; ignored if the session is transacted. Legal values are Session.AUTO_ACKNOWLEDGE, Session.CLIENT_ACKNOWLEDGE, and Session.DUPS_OK_ACKNOWLEDGE.
Returns:
a newly created session
Throws:
JMSException - if the Connection object fails to create a session due to some internal error or lack of support for the specific transaction and acknowledgement mode
```
 **结论**:
 1. 事务性会话中，session.commit()时确认消费
 2. 非事务性会话中：
	- **Session.AUTO_ACKNOWLEDGE：**当客户成功的从receive方法返回的时候，或者从
MessageListener.onMessage方法成功返回的时候，会话自动确认客户收到的消息。
	- **Session.CLIENT_ACKNOWLEDGE：**客户通过调用消息的acknowledge方法确认消
息。需要注意的是，在这种模式中，确认是在会话层上进行，确认一个被消费的消息
将自动确认所有已被会话消费的消息。例如，如果一个消息消费者消费了10 个消
息，然后确认第5 个消息，那么所有10 个消息都被确认。
	- **Session.DUPS_ACKNOWLEDGE：**该选择只是会话迟钝的确认消息的提交。如果JMS
provider失败，那么可能会导致一些重复的消息。如果是重复的消息，那么JMS
provider 必须把消息头的JMSRedelivered字段设置为true


**二、消息的优先级**
		可以使用消息优先级来指示JMS provider首先提交紧急的消息。优先级分
10个级别，从0（最低）到9（最高）。如果不指定优先级，默认级别是4。需要
注意的是，JMS provider并不一定保证按照优先级的顺序提交消息

**三、消息过期**
	可以设置消息在一定时间后过期，默认是永不过期

**四、消息的临时目的地**
	可以通过会话上的createTemporaryQueue 方法和createTemporaryTopic
方法来创建临时目的地。它们的存在时间只限于创建它们的连接所保持的时间。
只有创建该临时目的地的连接上的消息消费者才能够从临时目的地中提取消息

**五、持久订阅**

**六、事务**
	在一个JMS客户端，可以使用本地事务来组合消息的发送和接收。JMS
Session接口提供了commit和rollback方法。事务提交意味着生产的所有消息被
发送，消费的所有消息被确认；事务回滚意味着生产的所有消息被销毁，消费的
所有消息被恢复并重新提交，除非它们已经过期。
事务性的会话总是牵涉到事务处理中，commit或rollback方法一旦被调
用，一个事务就结束了，而另一个事务被开始。关闭事务性会话将回滚其中的事
务。
需要注意的是，如果使用请求/回复机制，即发送一个消息，同时希望在同
一个事务中等待接收该消息的回复，那么程序将被挂起，因为知道事务提交，发
送操作才会真正执行。
需要注意的还有一个，消息的生产和消费不能包含在同一个事务中，否则会**死锁**。



## JMS模型

**一、PTP**
ptp模型基于队列
1. 如果在Session 关闭时，有一些消息已经被收到，但还没有被签收(acknowledged)，那么，当消费者下次连接到相同的队列时，这些消息还会被再次接收
2. 如果用户在receive 方法中设定了消息选择条件，那么不符合条件的消息会留在队列中，不会被接收到
3. 队列可以长久地保存消息直到消费者收到消息。消费者不需要因为担心消息会丢失而时刻和队列保持激活的连接状态，充分体现了异步传输模式的优势JMS的PTP模型

** 二、Pub/Sub**
1. 消息订阅分为非持久订阅和持久订阅
**非持久订阅**只有当客户端处于激活状态，也就是和JMS Provider保持连接状态才能收到发送到某个主题的消息，而当客户端处于离线状态，这个时间段发到主题的消息将会丢失，永远不会收到。
**持久订阅**时，客户端向JMS 注册一个识别自己身份的ID，当这个客户端处于离线
时，JMS Provider会为这个ID 保存所有发送到主题的消息，当客户再次连接到JMS
Provider时，会根据自己的ID 得到所有当自己处于离线时发送到主题的消息。
2. 如果用户在receive 方法中设定了消息选择条件，那么不符合条件的消息不会被接收
3. 非持久订阅状态下，不能恢复或重新派送一个未签收的消息。只有持久订阅才能恢复或重
新派送一个未签收的消息。
4. 当所有的消息必须被接收，则用持久订阅。当丢失消息能够被容忍，则用非持久订阅