# Solr学习手册
[TOC]

## 一、全文搜索和Solr介绍

### 1.1	全文搜索技术简介

在一些大型门户网站、电子商务网站中，都需要站内搜索功能，使用传统的数据库查询方式实现搜索无法满足一些高级用户的需求，比如：

**搜索速度要快**
**搜搜结果按相关度排序**
**搜索内容格式不固定等**

这里就需要使用检索技术实现搜索功能:
-	单独使用Lucene实现站内搜索需要开发的工作量较大，主要表现在：索引维护，索引性能优化，搜索性能优化等，因此不建议使用
-	通过第三方搜索引擎提供的接口实现站内搜索，这样和第三方引擎系统依赖紧密，不方便扩展，不建议使用
-	基于Solr实现站内搜索扩展性能较好并且可以减少程序员的工作量，因为Solr提供了较为完备的搜索引擎解决方案，因此在门户，论坛等系统中常用此方案



### 1.2 Solr简介

- Solr是Apache下的一个顶级开源项目，采用Java开发，基于Lucene的全文搜索服务器。Solr提供了比Lucene更为丰富的查询语言，同时实现了可配置，可扩展，并对索引，搜索性能提供了优化。

- Solr可以独立运行，运行在Jetty，Tomcat等Servlet容器中，Solr索引实现的方法很简单，用POST方法向Solr服务器发送一个描述Field以及内容的XML文档，Solr根据xml文档添加、删除、更新索引。Solr检索只需要发送HTTP GET请求，然后对Solr返回Xml，Json等格式的查询结果进行解析，组织页面布局。Solr不提供构建UI的功能，Solr提供了一个管理界面，通过管理界面可以查询Solr的配置和运行情况。

### 1.3 什么是分词

**以下流程图演示了分词:**

![enter image description here](https://raw.githubusercontent.com/carl10086/my-pics/master/note/%E5%A6%82%E4%BD%95%E5%88%86%E8%AF%8D.png)


**以下演示如何进行搜索:**
![enter image description here](https://raw.githubusercontent.com/carl10086/my-pics/master/note/search.png)



### 1.4 如何安装solr

-	解压Solr:	tar –zxvf solr-4.10.3.tgz.tar
-	进入目录：cd solr-4.10.3/example/webapps
-	拷贝war文件到tomcat的webapps中: 	cp solr.war /usr/local/tomcat/webapps/
-	解压war:	mkdir solr && unzip solr.war –d solr && rm –rf solr.war
-	**修改解压好的solr文件夹，修改其文件**:	vim solr/WEB-INF/web.xml,查找env-entry内容，**解开注释文本**。并修改solr/home的地址: /usr/local/solr-4.10.3/example/solr。保存并退出即可。
-	**拷贝相关jar包到tomcat下**：	cd  /usr/local/solr-4.10.3/example/lib/ext && cp * /usr/local/tomcat/lib
-	启动tomcat: sh startup.sh
-	查看日志: tail -500 /usr/local/tomcat/logs/catalina.out
-	通过浏览器访问solr主页: http://...8080/solr 即可



### 1.5 如何解决中文问题

可能tomcat也要设置为UTF-8编码，一般项目都会完成这一点

无论是Solr还是lucense，对中文分词都不太好，我们一般索引中文需要用到IK中文分词器

- **把jar包拷贝到tomcat的solor的应用服务下：**
- `cp jar {TOMCAT_HOME}/webapps/solor/WEB-INF/lib`
- 在WEB-INF下创建classes文件夹
- 将IKAnalyzer.cfg.xml和stopword.dic拷贝到新创建的classes文件夹中
- 修改schema.xml文件

```
<fieldType name="text_ik" class="solr.TextField">

        <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer" />

        <analyzer type="query" isMaxWordLength="true"  class="org.wltea.analyzer.lucene.IKAnalyzer" />

    </fieldType>

```

- 重新启动tomcat，即可

**如何扩展自定义分词：**

- 修改刚刚自定的配置文件xml，加入新的dic文件即可
`<entry key="ext_dict">ext.dic;</entry>`

- vim ext.dic 添加词汇即可
