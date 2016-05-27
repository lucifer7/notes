# MyBatis


[TOC]



## 文档

[官网](http://www.mybatis.org/mybatis-3/zh/)


## spring 整合 mybatis

- step1:依赖的jar包主要以下两种

 >`<mybatis.version>3.2.8</mybatis.version>`
 >`<mybatis-spring.version>1.2.2</mybatis-spring.version>`
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>${mybatis.version}</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>${mybatis-spring.version}</version>
</dependency>
	```


- step2:用了个简单的自定义注解

```java
package com.carl.demo.mybatis.spring;

/**
 * Created by carl on 16-2-3.
 */

import org.springframework.stereotype.Component;

import java.lang.annotation.*;

/**
 * 标识MyBatis的DAO,方便{@link org.mybatis.spring.mapper.MapperScannerConfigurer}的扫描。
 *
 * @author thinkgem
 * @version 2013-8-28
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Component
public @interface MyBatisDao {
    /**
     * The value may indicate a suggestion for a logical component name,
     * to be turned into a Spring bean in case of an autodetected component.
     *
     * @return the suggested component name, if any
     */
    String value() default "";
}

```


- step3:**配置文件如下:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:util="http://www.springframework.org/schema/util" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
		http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
		http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
		http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd"
       default-lazy-init="true">

    <description>Spring Configuration</description>

    <!-- 加载配置属性文件 -->
    <context:property-placeholder ignore-unresolvable="true" location="classpath:jdbc.properties"/>

    <!-- 加载应用属性实例，可通过  @Value("#{APP_PROP['jdbc.driver']}") String jdbcDriver 方式引用 -->
    <util:properties id="APP_PROP" location="classpath:jdbc.properties" local-override="true"/>

    <!-- 使用Annotation自动注册Bean，解决事务失效问题：在主容器中不扫描@Controller注解，在SpringMvc中只扫描@Controller注解。  -->
    <context:component-scan base-package="com.carl.demo.mybatis.spring"><!-- base-package 如果多个，用“,”分隔 -->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- MyBatis begin -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--<property name="typeAliasesPackage" value="com.carl.demo.mybatis"/>-->
        <property name="mapperLocations" value="classpath:/mapper/**/*.xml"/>
        <property name="configLocation" value="classpath:/spring-mybatis-config.xml"></property>
    </bean>

    <!-- 扫描basePackage下所有以@MyBatisDao注解的接口 -->
    <bean id="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="basePackage" value="com.carl.demo.mybatis.spring"/>
        <property name="annotationClass" value="com.carl.demo.mybatis.spring.MyBatisDao"/>
    </bean>

    <!-- 定义事务 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置 Annotation 驱动，扫描@Transactional注解的类定义事务  -->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
    <!-- MyBatis end -->

    <!-- 配置 JSR303 Bean Validator 定义 -->
    <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>

    <!-- 缓存配置 -->
    <bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean">
        <property name="configLocation" value="classpath:${ehcache.configFile}"/>
    </bean>

    <!-- 计划任务配置，用 @Service @Lazy(false)标注类，用@Scheduled(cron = "0 0 2 * * ?")标注方法 -->
    <task:executor id="executor" pool-size="10"/>
    <task:scheduler id="scheduler" pool-size="10"/>
    <task:annotation-driven scheduler="scheduler" executor="executor" proxy-target-class="true"/>

    <!-- 数据源配置, 使用 BoneCP 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <!-- 数据源驱动类可不写，Druid默认会自动根据URL识别DriverClass -->
        <property name="driverClassName" value="${jdbc.driver}"/>

        <!-- 基本属性 url、user、password -->
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <!-- 配置初始化大小、最小、最大 -->
        <property name="initialSize" value="${jdbc.pool.init}"/>
        <property name="minIdle" value="${jdbc.pool.minIdle}"/>
        <property name="maxActive" value="${jdbc.pool.maxActive}"/>

        <!-- 配置获取连接等待超时的时间 -->
        <property name="maxWait" value="60000"/>

        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="60000"/>

        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="300000"/>

        <property name="validationQuery" value="${jdbc.testSql}"/>
        <property name="testWhileIdle" value="true"/>
        <property name="testOnBorrow" value="false"/>
        <property name="testOnReturn" value="false"/>

        <!-- 打开PSCache，并且指定每个连接上PSCache的大小（Oracle使用）
        <property name="poolPreparedStatements" value="true" />
        <property name="maxPoolPreparedStatementPerConnectionSize" value="20" /> -->

        <!-- 配置监控统计拦截的filters -->
        <property name="filters" value="stat"/>
    </bean>
</beans>
```



## MyBatis 拦截器原理探究

### Plugins 介绍:



1. Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
2. ParameterHandler (getParameterObject, setParameters)
3. ResultSetHandler (handleResultSets, handleOutputParameters)
4. StatementHandler (prepare, parameterize, batch, update, query)

自定义分页插件源代码,关键在于设计，此时的思路如下:
1. 需要被拦截的方法可以用参数中的信息，自定义注解，或者正则表达式等方式进行判断拦截
2. page 对象可以以各种形式传入，主要跟设计方法有关


Page 对象:
```java
package com.carl.demo.mybatis.spring;

import java.io.Serializable;
import java.util.Collections;
import java.util.LinkedList;
import java.util.List;

/**
 * Created by carl on 16-2-3.
 */
public class Page<T> implements Serializable {
    /**
     * 显示因子，展示的最大页数是:<code>FACTOR*2+1</code>
     */
    private static final int FACTOR = 2;

    /**
     * 当前页之前的所有页数
     */
    private List<Integer> prev = new LinkedList<Integer>();

    /**
     * 当前页之后的所有页数
     */
    private List<Integer> next = new LinkedList<Integer>();


    /**
     * 总记录数
     */
    private int totalCount;

    /**
     * 总页数
     */
    private int totalPage;

    /**
     * 当前页，从第一页开始
     */
    private int nowPage;

    /**
     * 含有的记录数
     */
    private List<T> result = Collections.EMPTY_LIST;

    /**
     * 起始位置
     */
    private int start;

    /**
     * 每页记录数
     */
    private int pageShow = 10;

    public List<Integer> getPrev() {
        calPrev(prev, getNowPage(), getTotalPage());
        return prev;
    }

    public List<Integer> getNext() {
        calNext(next, getNowPage(), getTotalPage());
        return next;
    }


    private static void calPrev(List<Integer> list, int pageNum, int pageCount) {
        list.clear();
        int startNum = 0;
        int sideNum = FACTOR;
        if (pageNum <= sideNum) {
            startNum = 1;
        } else {
            if ((pageNum + sideNum) >= pageCount) {
                if ((2 * sideNum + 1) >= pageCount) {
                    if ((pageCount - 2 * sideNum) >= 1) {
                        startNum = pageCount - 2 * sideNum;
                    } else {
                        startNum = 1;
                    }
                } else {
                    startNum = pageCount - 2 * sideNum;
                }
            } else {
                if ((pageNum - sideNum) >= 1) {
                    startNum = pageNum - sideNum;
                } else {
                    startNum = 1;
                }
            }
        }
        if (startNum > 0) {
            for (int i = startNum; i <= pageNum - 1; i++) {
                list.add(i);
            }
        }
    }


    /**
     * 计算显示当前分页的起始页
     *
     * @param pageNum   当前页码
     * @param pageCount 总页数
     */
    private static void calNext(List<Integer> list, int pageNum, int pageCount) {
        list.clear();
        int endNum = 0;
        int sideNum = FACTOR;
        if (pageCount <= sideNum) {
            endNum = pageCount;
        } else {
            if ((sideNum + pageNum) >= pageCount) {
                endNum = pageCount;
            } else {
                endNum = sideNum + pageNum;
                if ((sideNum + pageNum) <= (2 * sideNum + 1)) {
                    if ((2 * sideNum + 1) >= pageCount) {
                        endNum = pageCount;
                    } else {
                        endNum = 2 * sideNum + 1;
                    }
                } else {
                    endNum = sideNum + pageNum;
                }
            }
        }
        //如果endNumber > pageNum
        if (endNum > pageNum) {
            for (int i = pageNum + 1; i <= endNum; i++) {
                list.add(i);
            }
        }
    }

    public int getStart() {
        start = (getNowPage() - 1) * getPageShow();
        if (start < 0) {
            start = 0;
        }
        return start;
    }

    public void setStart(int start) {
        this.start = start;
    }

    public int getPageShow() {
        return pageShow;
    }

    public void setPageShow(int pageShow) {
        this.pageShow = pageShow;
    }

    public int getTotalCount() {
        return totalCount;
    }

    public void setTotalCount(int totalCount) {
        this.totalCount = totalCount;
    }

    public List<T> getResult() {
        return result;
    }

    public void setResult(List<T> result) {
        this.result = result;
    }

    public void setTotalPage(int totalPage) {
        this.totalPage = totalPage;
    }

    public void setNowPage(int nowPage) {
        this.nowPage = nowPage;
    }

    public int getTotalPage() {
        return (int) Math.ceil(totalCount * 1.0 / pageShow);
    }

    public int getNowPage() {
        if (nowPage <= 0)
            nowPage = 1;
        if (nowPage > getTotalPage())
            nowPage = getTotalPage();
        return nowPage;
    }


}

```
Page Plugin对象

```java
package com.carl.demo.mybatis.spring;

import org.apache.ibatis.executor.ErrorContext;
import org.apache.ibatis.executor.ExecutorException;
import org.apache.ibatis.executor.statement.BaseStatementHandler;
import org.apache.ibatis.executor.statement.RoutingStatementHandler;
import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.ParameterMapping;
import org.apache.ibatis.mapping.ParameterMode;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.property.PropertyTokenizer;
import org.apache.ibatis.scripting.xmltags.ForEachSqlNode;
import org.apache.ibatis.session.Configuration;
import org.apache.ibatis.type.TypeHandler;
import org.apache.ibatis.type.TypeHandlerRegistry;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.Field;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import java.util.Map;
import java.util.Properties;

/**
 * Created by carl on 2016/2/3.
 */
@Intercepts({@Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class})})
public class PagePlugin implements Interceptor {
    private static final Logger LOGGER = LoggerFactory.getLogger(PagePlugin.class);
    private static String dialect = "";    //数据库方言
    private static String pageSqlId = ""; //mapper.xml中需要拦截的ID(正则匹配)

    @Override
    public Object intercept(Invocation ivk) throws Throwable {
        if (ivk.getTarget() instanceof RoutingStatementHandler) {
            RoutingStatementHandler statementHandler = (RoutingStatementHandler) ivk.getTarget();
            BaseStatementHandler delegate = (BaseStatementHandler) Reflections.getFieldValue(statementHandler, "delegate");
            MappedStatement mappedStatement = (MappedStatement) Reflections.getFieldValue(delegate, "mappedStatement");
            //通过正则表达式判断满足分页过滤条件
            if (mappedStatement.getId().matches(pageSqlId)) {
                BoundSql boundSql = delegate.getBoundSql();
                Object parameterObject = boundSql.getParameterObject();//分页SQL<select>中parameterType属性对应的实体参数，即Mapper接口中执行分页方法的参数,该参数不得为空
                if (parameterObject == null) {
                    throw new NullPointerException("parameterObject尚未实例化！");
                } else {
                    Connection connection = (Connection) ivk.getArgs()[0];
                    String sql = boundSql.getSql();
                    String countSql = "select count(0) from (" + sql + ") as tmp_count"; //记录统计
                    PreparedStatement countStmt = connection.prepareStatement(countSql);
                    BoundSql countBS = new BoundSql(mappedStatement.getConfiguration(), countSql, boundSql.getParameterMappings(), parameterObject);
                    setParameters(countStmt, mappedStatement, countBS, parameterObject);
                    ResultSet rs = countStmt.executeQuery();
                    int count = 0;
                    if (rs.next()) {
                        count = rs.getInt(1);
                    }
                    rs.close();
                    countStmt.close();
                    // 努力发现这个属性
                    Page page = searchPageObj(parameterObject);
                    page.setTotalCount(count);
                    String pageSql = generatePageSql(sql, page);
                    Reflections.setFieldValue(boundSql, "sql", pageSql); //将分页sql语句反射回BoundSql.
                }
            }
        }

        return ivk.proceed();
    }

    /**
     * 从parameterObject剥离出Page对象
     *
     * @param parameterObject
     * @return
     */
    public Page searchPageObj(Object parameterObject) throws NoSuchFieldException {
        Page ret = null;

        //1. 本身就是Page对象
        if (parameterObject instanceof Page) {
            ret = (Page) parameterObject;
        } else if (parameterObject instanceof Map) {
            Map map = (Map) parameterObject;
            Object obj = map.get("page");
            if (obj != null && obj instanceof Page) {
                ret = (Page) obj;
            } else {
                //遍历所有的value
                for (Object o : map.values()) {
                    if (o instanceof Page) {
                        ret = (Page) o;
                        break;
                    }
                }
            }
        } else {
            Field pageField = Reflections.getAccessibleField(parameterObject, "page");
            if (pageField != null) {
                ret = (Page) Reflections.getFieldValue(parameterObject, "page");
                if (ret == null) {
                    ret = new Page();
                    Reflections.setFieldValue(parameterObject, "page", ret);//通过反射，对实体对象设置分页对象
                }
            }
        }

        if (ret == null) {
            throw new NoSuchFieldException(parameterObject.getClass().getName() + "不存在 page 属性！");
        }
        return ret;
    }


    /**
     * 对SQL参数(?)设值,参考org.apache.ibatis.executor.parameter.DefaultParameterHandler
     *
     * @param ps
     * @param mappedStatement
     * @param boundSql
     * @param parameterObject
     * @throws SQLException
     */

    private void setParameters(PreparedStatement ps, MappedStatement mappedStatement, BoundSql boundSql, Object parameterObject) throws SQLException {
        ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        if (parameterMappings != null) {
            Configuration configuration = mappedStatement.getConfiguration();
            TypeHandlerRegistry typeHandlerRegistry = configuration.getTypeHandlerRegistry();
            MetaObject metaObject = parameterObject == null ? null : configuration.newMetaObject(parameterObject);
            for (int i = 0; i < parameterMappings.size(); i++) {
                ParameterMapping parameterMapping = parameterMappings.get(i);
                if (parameterMapping.getMode() != ParameterMode.OUT) {
                    Object value;
                    String propertyName = parameterMapping.getProperty();
                    PropertyTokenizer prop = new PropertyTokenizer(propertyName);
                    if (parameterObject == null) {
                        value = null;
                    } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                        value = parameterObject;
                    } else if (boundSql.hasAdditionalParameter(propertyName)) {
                        value = boundSql.getAdditionalParameter(propertyName);
                    } else if (propertyName.startsWith(ForEachSqlNode.ITEM_PREFIX) && boundSql.hasAdditionalParameter(prop.getName())) {
                        value = boundSql.getAdditionalParameter(prop.getName());
                        if (value != null) {
                            value = configuration.newMetaObject(value).getValue(propertyName.substring(prop.getName().length()));
                        }
                    } else {
                        value = metaObject == null ? null : metaObject.getValue(propertyName);
                    }
                    TypeHandler typeHandler = parameterMapping.getTypeHandler();
                    if (typeHandler == null) {
                        throw new ExecutorException("There was no TypeHandler found for parameter " + propertyName + " of statement " + mappedStatement.getId());
                    }
                    typeHandler.setParameter(ps, i + 1, value, parameterMapping.getJdbcType());
                }
            }
        }
    }

    /**
     * 根据数据库方言，生成特定的分页sql
     *
     * @param sql
     * @param page
     * @return
     */
    private String generatePageSql(String sql, Page page) {
        if (page != null && dialect != null && dialect.trim().length() > 0) {
            StringBuffer pageSql = new StringBuffer();
            if ("mysql".equals(dialect)) {
                pageSql.append(sql);
                pageSql.append(" limit " + page.getStart() + "," + page.getPageShow());
            } else if ("oracle".equals(dialect)) {
                pageSql.append("select * from (select tmp_tb.*,ROWNUM row_id from (");
                pageSql.append(sql);
                pageSql.append(") as tmp_tb where ROWNUM<=");
                pageSql.append(page.getStart() + page.getPageShow());
                pageSql.append(") where row_id>");
                pageSql.append(page.getStart());
            }
            return pageSql.toString();
        } else {
            return sql;
        }
    }

    @Override
    public Object plugin(Object o) {
        return Plugin.wrap(o, this);
    }

    @Override
    public void setProperties(Properties properties) {
        dialect = properties.getProperty("dialect");
        pageSqlId = properties.getProperty("pageSqlId");
    }
}

```



