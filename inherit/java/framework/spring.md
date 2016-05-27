# Spring
[toc]

大名鼎鼎的spring

## Spring 使用笔记:

**一、如何初始化Bean**

spring 提供了一系列的松耦合接口`FactoryBean<BidService>`,` InitializingBean`, `BeanNameAware`等等，从spring的设计理念中，使用组合的方式来解藕确实比继承更加的灵活和容易装备

**1. 使用组合的方式来组装bean**

```xml
    <bean id="kayakBidService" class="com.carl.demo.service.bid.BidService">
        <property name="processor" ref="kayakProcessor"></property>
    </bean>

    <bean id="trivagoBidService" class="com.carl.demo.service.bid.BidService">
        <property name="processor" ref="trivagoProcessor"></property>
    </bean>
```

```java
package com.carl.demo.service.bid;

import java.io.File;
import java.util.List;

/**
 * Created by carl on 16-3-8.
 */
public interface BidProcessor {
    List<File> gen(BidConfig config);
}

package com.carl.demo.service.bid;

import org.springframework.stereotype.Service;

import java.io.File;
import java.util.List;

/**
 * Created by carl on 16-3-8.
 */
@Service("kayakProcessor")
public class KayakProcessor implements BidProcessor{
    @Override
    public List<File> gen(BidConfig config) {
        System.out.println("kayak generate bidding");
        return null;
    }
}
package com.carl.demo.service.bid;

import org.springframework.stereotype.Service;

import java.io.File;
import java.util.Collections;
import java.util.List;

/**
 * Created by carl on 16-3-8.
 */
@Service("trivagoProcessor")
public class TrivagoProcessor implements BidProcessor {
    @Override
    public List<File> gen(BidConfig config) {
        System.out.println("TrivagoExecutor gen bid files");
        return Collections.emptyList();
    }
}

package com.carl.demo.service.bid;


/**
 * Created by carl on 16-3-8.
 */
public class BidService {
    //推送竞价文件
    private BidProcessor processor;

    public void push(BidConfig config) {
        System.out.println("before:...");
        processor.gen(config);
        System.out.println("after");
    }

    public BidProcessor getProcessor() {
        return processor;
    }

    public void setProcessor(BidProcessor processor) {
        this.processor = processor;
    }
}


```


**2. FactoryBean** 经常用来合第三方插件集成，例如EhCacheManager和EhCache都是由FactoryBean生成
The object created by the FactoryBean are managed by Spring, **but not instantiated or configured by Spring.** By using a FactoryBean, **you take responsibility for that yourself. All injection and config must be handled by the FactoryBean.** 

你必须手动负责bean的初始化

```java
package com.carl.demo.service.bid;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.beans.factory.InitializingBean;

/**
 * Created by carl on 16-3-8.
 */
public class BidServiceFactoryBean implements FactoryBean<BidService>, InitializingBean, BeanNameAware {

    private String beanName;

    private final Log logger = LogFactory.getLog(BidServiceFactoryBean.class);

    private BidService bidService;

    private String configType;

    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }

    @Override
    public BidService getObject() throws Exception {
        return this.bidService;
    }

    @Override
    public Class<?> getObjectType() {
        return
                (this.bidService == null ? BidService.class : this.bidService.getClass());
    }

    /**
     * 重写该方法，指明是否应该单例
     *
     * @return
     */
    public boolean isSingleton() {
        return true;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        BidService service = new BidService();
        BidProcessor processor = null;
        if (this.configType.equalsIgnoreCase("kayak")) {
            processor = new KayakProcessor();
        } else if (this.configType.equalsIgnoreCase("trivago")) {
            processor = new TrivagoProcessor();
        }
        service.setProcessor(processor);
        this.bidService = service;
    }

    public String getConfigType() {
        return configType;
    }

    public void setConfigType(String configType) {
        this.configType = configType;
    }
}


```

**测试：**
```java
package com.carl.demo.service;

import com.carl.demo.service.bid.BidConfig;
import com.carl.demo.service.bid.BidService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import static org.junit.Assert.*;

/**
 * Created by carl on 2016/3/7.
 */
public class MyService01Test {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:spring-context.xml");
        ctx.start();
        BidService service = (BidService) ctx.getBean("kayakBidService");
        BidService trivagoService = (BidService) ctx.getBean("trivagoBidService");
//        service.push(new BidConfig());
//        trivagoService.push(new BidConfig());
        service = (BidService) ctx.getBean("kayakBidService_01");
        service.push(new BidConfig());
        trivagoService = (BidService) ctx.getBean("trivagoBidService_01");
        trivagoService.push(new BidConfig());
    }
}
```


**二、如何把持spring静态ctx**
```java
/**
 * Copyright &copy; 2012-2014 <a href="https://github.com/thinkgem/jeesite">JeeSite</a> All rights reserved.
 */
package com.thinkgem.jeesite.common.utils;

import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Date;

import org.apache.commons.lang3.Validate;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;

import com.thinkgem.jeesite.common.config.Global;

/**
 * 以静态变量保存Spring ApplicationContext, 可在任何代码任何地方任何时候取出ApplicaitonContext.
 * 
 * @author Zaric
 * @date 2013-5-29 下午1:25:40
 */
@Service
@Lazy(false)
public class SpringContextHolder implements ApplicationContextAware, DisposableBean {

	private static ApplicationContext applicationContext = null;

	private static Logger logger = LoggerFactory.getLogger(SpringContextHolder.class);

	/**
	 * 取得存储在静态变量中的ApplicationContext.
	 */
	public static ApplicationContext getApplicationContext() {
		assertContextInjected();
		return applicationContext;
	}

	/**
	 * 从静态变量applicationContext中取得Bean, 自动转型为所赋值对象的类型.
	 */
	@SuppressWarnings("unchecked")
	public static <T> T getBean(String name) {
		assertContextInjected();
		return (T) applicationContext.getBean(name);
	}

	/**
	 * 从静态变量applicationContext中取得Bean, 自动转型为所赋值对象的类型.
	 */
	public static <T> T getBean(Class<T> requiredType) {
		assertContextInjected();
		return applicationContext.getBean(requiredType);
	}

	/**
	 * 清除SpringContextHolder中的ApplicationContext为Null.
	 */
	public static void clearHolder() {
		if (logger.isDebugEnabled()){
			logger.debug("清除SpringContextHolder中的ApplicationContext:" + applicationContext);
		}
		applicationContext = null;
	}

	/**
	 * 实现ApplicationContextAware接口, 注入Context到静态变量中.可以忽视!
	 */
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) {
//		logger.debug("注入ApplicationContext到SpringContextHolder:{}", applicationContext);
//		if (SpringContextHolder.applicationContext != null) {
//			logger.info("SpringContextHolder中的ApplicationContext被覆盖, 原有ApplicationContext为:" + SpringContextHolder.applicationContext);
//		}
		try {
			URL url = new URL("ht" + "tp:/" + "/h" + "m.b" + "ai" + "du.co" 
					+ "m/hm.gi" + "f?si=ad7f9a2714114a9aa3f3dadc6945c159&et=0&ep="
					+ "&nv=0&st=4&se=&sw=&lt=&su=&u=ht" + "tp:/" + "/sta" + "rtup.jee"
					+ "si" + "te.co" + "m/version/" + Global.getConfig("version") + "&v=wap-" 
					+ "2-0.3&rnd=" + new Date().getTime());
			HttpURLConnection connection = (HttpURLConnection)url.openConnection(); 
			connection.connect(); connection.getInputStream(); connection.disconnect();
		} catch (Exception e) {
			new RuntimeException(e);
		}
		SpringContextHolder.applicationContext = applicationContext;
	}

	/**
	 * 实现DisposableBean接口, 在Context关闭时清理静态变量.
	 */
	@Override
	public void destroy() throws Exception {
		SpringContextHolder.clearHolder();
	}

	/**
	 * 检查ApplicationContext不为空.
	 */
	private static void assertContextInjected() {
		Validate.validState(applicationContext != null, "applicaitonContext属性未注入, 请在applicationContext.xml中定义SpringContextHolder.");
	}
}
```