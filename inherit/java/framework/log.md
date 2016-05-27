# Log


[TOC]



## Log4j

### demo

1. 先看依赖的jar,没有这么多,主要是slf4j和log4j,其中slf4j并不是日志框架，是`facade`外观设计模式的实战体现

	```xml
 		<dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${apache-log4j.version}</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- common-logging 实际调用slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- java.util.logging 实际调用slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jul-to-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
	```
    
    >其中 `<apache-log4j.version>1.2.14</apache-log4j.version>`  ` <slf4j.version>1.7.7</slf4j.version>`

2. 具体的demo
	```xml
    <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j='http://jakarta.apache.org/log4j/'>
    <appender name="test" class="org.apache.log4j.ConsoleAppender">
        <!--<param name="Threshold" value="ERROR"/>-->
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern"
                   value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n"/>
        </layout>
    </appender>
    <appender name="file" class="org.apache.log4j.DailyRollingFileAppender">
        <!--只记录error-->
        <filter class="org.apache.log4j.varia.LevelRangeFilter">
            <param name="levelMin" value="error"/>
            <param name="levelMax" value="error"/>
            <param name="AcceptOnMatch" value="true"/>
        </filter>
        <!--日志文件路径-->
        <param name="File" value="test.log"/>
        <!--每天一个，文件名格式-->
        <param name="DatePattern" value="'.'yyyy-MM-dd'.log'"/>
        <!--日志输出形式-->
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern"
                   value="[%d{MMdd HH:mm:ss SSS\} %-5p] [%t] %c{3\} - %m%n"/>
        </layout>
    </appender>
    <!--additivity为true表示会重复记录其他Logger记录的日志-->
    <logger name="cn.javass" additivity="true">
        <!--<level value="error"/>-->
        <appender-ref ref="file"/>
    </logger>
    <root>
        <level value="debug"/>
        <appender-ref ref="test"/>
        <appender-ref ref="file"/>
    </root>
</log4j:configuration>
    ```

3. java
	```java
    	private static final Logger logger = LoggerFactory.getLogger(LogTest.class);

        public static void main(String[] args) {
            logger.debug("debug");
            logger.info("info");
            logger.warn("warn");
            logger.error("error");
            System.out.println("isDebugEnabled:" + logger.isDebugEnabled());
            System.out.println("isErrorEnabled:" + logger.isErrorEnabled());
            System.out.println("isInfoEnabled:" + logger.isInfoEnabled());
            System.out.println("isWarnEnabled:" + logger.isWarnEnabled());
        }

    ```

### 日志级别

<div style="font-size:12px;">
1. `OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL`。Log4j建议只使用四个级别,优先级从高到低分别是ERROR、WARN、INFO、DEBUG。通过在这里定义的级别,可以控制到应用程序中相应级别的日志信息的开关。

2. 级别的控制,就是只要`大于等于`指定的控制级别,就可以输出
3. 如果有多个logger,都可以匹配输出,则每个logger都产生输出,其中根logger匹配所有的输出;**而级别控制来源于路径最详细的logger**。
</div>


### 输出源 Appender

1. 定义

    >a. Log4j允许日志请求被输出到多个输出源。用Log4j的话说,一个输出源被称做一个Appender 。
    >b. 一个logger可以设置超过一个的appender。

2. 常见Appender
	- org.apache.log4j.ConsoleAppender(控制台)
	- org.apache.log4j.FileAppender(文件)
	- org.apache.log4j.DailyRollingFileAppender(每天产生一个日志文件)
	- org.apache.log4j.RollingFileAppender(文件大小到达指定尺寸的时候产生一个新的文件)
	- org.apache.log4j.WriterAppender(将日志信息以流格式发送到任意指定的地方)
	- org.apache.log4j.jdbc.JDBCAppender(把日志用JDBC记录到数据库中)

### Layout
- org.apache.log4j.HTMLLayout(以HTML表格形式布局),
- org.apache.log4j.PatternLayout(可以灵活地指定布局模式),
- org.apache.log4j.SimpleLayout(包含日志信息的级别和信息字符串),
- org.apache.log4j.TTCCLayout(包含日志产生的时间、线程、类别等等信息)
- org.apache.log4j.PatternLayout
	1. `%m` 输出代码中指定的消息
    2. `%p` 输出优先级,即DEBUG,INFO,WARN,ERROR,FATAL
    3. `%r` 输出自应用启动到输出该log信息耗费的毫秒数
    4. `%c` 输出所属的类目,通常就是所在类的全名
    5. `%t` 输出产生该日志事件的线程名
    6. `%n` 输出一个回车换行符,Windows平台为“\r\n”,Unix平台为“\n”
    7. `%d` 输出日志时间点的日期或时间,默认格式为ISO8601,也可以在其后指定格式,比如:%d{yyy MMM dd HH:mm:ss,SSS},输出类似:2002年10月18日22:10:28,921
    8. `%l` 输出日志事件的发生位置,包括类目名、发生的线程,以及在代码中的行数。举例:Testlog4.main(TestLog4.java:10)
