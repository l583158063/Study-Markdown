Spring Boot/Cloud log4j2.x 引入与配置
---

#### 前言

​        Log4j 1.x 已被广泛采用，并用于许多应用中。然而，经过多年的发展，它的发展已经放缓。由于它需要与非常老的 Java 版本兼容，因此维护变得更加困难，并且在 2015 年 8 月成为 End of Life（停止维护）。它的替代方案 SLF4J / Logback，对框架进行了许多必要的改进。 那么，为什么还要浪费心思去使用 Log4j 2.x 呢？以下是一些原因。

1. Log4j 2 被设计成可用作**审计日志**框架。在重新配置时，Log4j 1.x和 Logback 都会**丢失事件**，而 Log4j 2 不会。在 Logback 中，Appender 中的异常永远不会对应用程序可见。 在 Log4j 中，可以将 Appender 配置为允许异常渗透到应用程序。
2. Log4j 2包含基于 **LMAX Disruptor** 库的下一代**异步日志记录器**（Asynchronous Loggers）。在多线程场景中，相比 Log4j 1.x 和 Logback，异步日志记录器的吞吐量高 10 倍，延迟低几个数量级，效率第一。详见[Log4j2的性能为什么这么好？](https://www.jianshu.com/p/359b14067b9e)
3. Log4j 2 对于独立应用程序是**[无垃圾](http://logging.apache.org/log4j/2.x/manual/garbagefree.html)**的，对于稳态日志记录期间的 Web 应用程序是低垃圾。 这减少了垃圾收集器的压力，并提供更好的响应时间性能。
4. Log4j 2 使用一个[插件系统](http://logging.apache.org/log4j/2.x/manual/plugins.html)，通过添加新的 [Appender](http://logging.apache.org/log4j/2.x/manual/appenders.html)，[Filters](http://logging.apache.org/log4j/2.x/manual/filters.html)，[Layouts](http://logging.apache.org/log4j/2.x/manual/layouts.html)，[Lookups](http://logging.apache.org/log4j/2.x/manual/lookups.html) 和 Pattern Converters，可以非常轻松地扩展框架，而无需对 Log4j 进行任何更改。
5. 由于插件系统配置更简单。配置中的条目不需要指定类名。
6. 支持[自定义日志级别](http://logging.apache.org/log4j/2.x/manual/customloglevels.html)。可以在代码或配置中定义自定义日志级别。
7. 支持 [lambda 表达式](http://logging.apache.org/log4j/2.x/manual/api.html#LambdaSupport)。只有在启用了请求的日志级别时，在 Java 8 上运行的客户端代码才能使用 lambda 表达式来延迟构造日志消息。不需要显式级别检查，从而产生更清晰的代码。
8. 支持 [Message 对象](http://logging.apache.org/log4j/2.x/manual/messages.html)。消息允许支持有趣和复杂的构造通过日志系统传递并被有效地操作。 用户可以自由创建自己的 [Message](http://logging.apache.org/log4j/2.x/log4j-api/apidocs/org/apache/logging/log4j/message/Message.html) 类型，并编写自定义 [Layouts](http://logging.apache.org/log4j/2.x/manual/layouts.html)，[Filters](http://logging.apache.org/log4j/2.x/manual/filters.html) 和 [Lookups](http://logging.apache.org/log4j/2.x/manual/lookups.html) 来操作它们。
9. Log4j 1.x 支持 Appender 上的 Filters。Logback 添加了 TurboFilters，允许在 Logger 处理事件之前对事件进行过滤。Log4j 2 支持可以配置为在 Logger 处理事件之前处理事件的 Filters，因为它们由一个 Logger 或在一个 Appender 上处理。
10. 许多 Logback Appender 不接受布局，只会以固定格式发送数据。大多数 Log4j 2 Appender 接受布局，允许以任何所需格式传输数据。
11. Log4j 1.x 和 Logback 中的布局返回一个 String。Log4j 2 采用更简单的方法，布局总是返回一个字节数组。  这样做的好处是，它们实际上几乎可以在任何 Appender 中使用，而不仅仅是写入 OutputStream 的 Appender。
12. [Syslog Appender](http://logging.apache.org/log4j/2.x/manual/appenders.html#SyslogAppender) 既支持 TCP 和 UDP，也支持 BSD syslog 和 [RFC5424](http://tools.ietf.org/html/rfc5424) 格式。
13. **Log4J 2 利用 Java 5 并发支持**，并在可能的最低级别上执行锁定。log4j 1.x 有已知的死锁问题。其中许多都是在 logback 中修复的，但许多 logback 类仍然需要在相当高的级别上进行同步。
14. 它是一个 Apache Software Foundation 项目，遵循所有 ASF 项目使用的社区和支持模型。 如果您想贡献或获得提交更改的权利，请按照贡献中列出的路径进行操作。

---

【正文】

#### 一、项目依赖 log4j2 包

1. 在 pom.xml 中引入相关的包，下面的示例还包含了将日志文件上传至阿里云的依赖，如果只是本地打印日志并写入文件的话只需要依赖 log4j2 即可。

```xml
<!--protobuf数据格式转换-->
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
</dependency>
<!--阿里云日志服务-->
<dependency>
    <groupId>com.aliyun.openservices</groupId>
    <artifactId>aliyun-log-log4j2-appender</artifactId>
</dependency>
<!--依赖LOG4J2-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

2. 由于 log4j2 与 log4j1 不相容（第一代已经停止维护了），而 spring-boot 会依赖 log4j1，所以还需要排除旧的依赖，全局搜索 spring-boot-starter-logging（在 pom.xml 文件中右键点击 Diagrams -> Show Dependencies，ctrl + f 搜索 logging）并右键 Exclude。注意重复依赖的情况，多查两次。

```xml
<!--spring boot-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

3. application.yml

```yaml
logging:
  config: classpath:log4j2-${logEnv:dev}.xml
```

> logEnv 参数默认是 dev 环境，可通过启动参数配置调成 uat 或 prod 等。

#### 二、标签含义与配置

​        引入 log4j2.0 以后通常在 log4j2.xml 中配置相关参数，在配置的时候我们需要理解这些参数的具体含义，通过项目中使用的配置模板，解释下列主要参数。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- log4j2-dev.xml -->

<configuration status="error">
    <properties>
        <property name="LOG_HOME">logs</property>
        <property name="FILE_NAME">log4j2-project-name</property>
    </properties>

    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout
                    pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight{%-5level} [%t] [%X{userId}] %logger : %msg%n"/>
        </Console>
        <!-- 外部服务器日志输出，一般是云服务器配置 -->
        <Loghub name="Loghub"
                project="***"
                logStore="***"
                endpoint="***"
                accessKeyId="***"
                accessKeySecret="***"
                totalSizeInBytes="104857600"
                maxBlockMs="60000"
                ioThreadCount="8"
                batchSizeThresholdInBytes="524288"
                batchCountThreshold="4096"
                lingerMs="2000"
                retries="10"
                baseRetryBackoffMs="100"
                maxRetryBackoffMs="100"
                topic="project-name"
                source=""
                timeFormat="yyyy-MM-dd'T'HH:mmZ"
                timeZone="Asia/Shanghai"
                ignoreExceptions="true">
            <PatternLayout pattern="%d%-5level[%thread]%logger{0}:%msg"/>
        </Loghub>
        <!-- 这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingRandomAccessFile name="LogFile"
                                 fileName="${LOG_HOME}/${FILE_NAME}.log"
                                 filePattern="${LOG_HOME}/$${date:yyyy-MM}/${FILE_NAME}-%d{yyyy-MM-dd}-%i.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%t] %logger{36} - %msg%n"/>
            <Policies>
                <!-- 每天创建一个日志文件 -->
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="50 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20"/>

            <Filters>
                <!-- 日志文件中的显示level级别的信息 -->
                <ThresholdFilter onMismatch="DENY" onMatch="ACCEPT" level="DEBUG"/>
            </Filters>
        </RollingRandomAccessFile>
    </appenders>

    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
            <appender-ref ref="Loghub"/>
            <appender-ref ref="LogFile"/>
        </root>

        <!-- 设置某些包或文件的日志输出级别，避免日志文件中出现多余无用的日志 -->
        <!--<logger name="org.springframework" level="WARN"/>
        <logger name="org.apache" level="WARN"/>
        <logger name="com.netflix" level="WARN"/>
        <logger name="org.hibernate.validator" level="WARN"/>
        <logger name="com.sun.jersey" level="WARN"/>
        <logger name="org.mybatis" level="INFO"/>
        <logger name="springfox.documentation" level="WARN"/>
        <logger name="tk.mybatis" level="WARN"/>
        <logger name="com.zaxxer.hikari" level="WARN"/>-->
        <logger name="org.apache.ibatis" level="DEBUG"/>
        <logger name="io.choerodon" level="DEBUG"/>
        <logger name="org.hzero.platform.infra.mapper" level="DEBUG"/>
        <logger name="org.xxx" level="DEBUG"/>
        <logger name="com.xxx" level="DEBUG"/>
    </loggers>
</configuration>
```
> \<properties> 和 \<property> 是 xml 通用节点，可在其他地方用 ${name} 引用其中数据。
##### 1、根节点 configuration

有两个属性：status 和 monitorinterval，有两个子节点：appenders 和 loggers。 

- **status** 用来指定 log4j 本身的打印日志的级别。其实 status 属性是帮助开发者找错用的，它可以检测 log4j2 的配置文件是否有错，也可以检测到死循环的 logger。
- **monitorinterval** 用于指定 log4j 自动重新配置的监测间隔时间，单位是 s，最小间隔 5s。log4j2 检测到配置文件有变化，会重新配置自己。

##### 2、appenders 节点

常见的有三种子节点：Console、RollingFile 等。

- **Console** 节点用来定义输出到控制台的 Appender。
  - name：指定 Appender 的名字。
  - target：SYSTEM_OUT 或 SYSTEM_ERR，一般只设置默认：SYSTEM_OUT。
  - PatternLayout：输出格式，不设置默认为：%msg%n。
- **RollingFile** 节点用来定义超过指定大小自动删除旧的创建新的 Appender。【多个类型后文详解】
  - name：指定 Appender 的名字。
  - fileName：指定输出日志的目的文件带全路径的文件名。
  - PatternLayout：输出格式，不设置默认为：%msg%n。
  - filePattern：指定新建归档日志文件的名称格式。
  - Policies：指定滚动日志的策略，就是什么时候进行新建归档日志文件输出日志。有两个子节点：
    - TimeBasedTriggeringPolicy：该策略完成周期性的 log 文件封存。有两个属性：
    
      1. interval，Integer 型，指定两次封存动作之间的时间间隔。单位：以日志的命名精度来确定单位，比如 yyyy-MM-dd HH 单位为小时，yyyy-MM-dd HH:mm 单位为分钟。
    
      2. modulate，boolean 型，说明是否对封存时间进行调制。若 modulate=true，则封存时间将以 0 时刻为边界进行偏移计算。比如，modulate=true，interval=4hours，假设上次封存日志的时间为 03:00，则下次封存日志的时间为 04:00，之后的封存时间依次为 08:00，12:00，16:00。
    
    - SizeBasedTriggeringPolicy：基于指定文件大小的滚动策略，*size* 属性用来定义每个日志文件的大小，单个日志文件超过了设置的大小，会创建新的日志文件，示例的 filePattern 中以 %i 来做自增。
  - DefaultRolloverStrategy：用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的， 如果不做配置，默认是 7。
  
##### 3、loggers 节点，常见的有两种：root 和 logger。

  - root 节点用来指定项目的根日志，如果没有单独指定 logger，那么就会默认使用该 eoot 日志输出。
    
    - appender-ref：root 的子节点，用来指定该日志输出到哪个 appender。
  - logger 节点用来单独指定日志的形式，比如要为指定包下的 class 指定不同的日志级别等。 
    - name：用来指定该 Logger 所适用的类或者类所在的包全路径，继承自 root 节点。
    
    - additivity：设置事件是否在 root 输出，为了避免重复输出（切断继承），可设置为 false。
    
    - level：设置日志输出的最低级别，从高到低依次为：OFF、FATAL、$ERROR$、$WARN$、$INFO$、$DEBUG$、TRACE、 ALL。（斜体为常用级别）
    
##### 4、日志级别

​        log4j 定义了8个级别的 log（除去 OFF 和 ALL，可以说分为 6 个级别），优先级从高到低依次为：OFF、FATAL、ERROR、WARN、INFO、DEBUG、TRACE、 ALL。

1. ALL：最低等级的，用于打开所有日志记录。

2. TRACE：很低的日志级别，一般不会使用。

3. DEBUG：指出细粒度信息事件对调试应用程序是非常有帮助的，主要用于开发过程中打印一些运行信息。

4. INFO：消息在粗粒度级别上突出强调应用程序的运行过程。打印一些业务流程上的重要信息，这个可以用于生产环境中输出程序运行的业务流程，但是不能滥用，避免打印过多的日志。

5. WARN：表明会出现潜在错误的情形，有些信息不是错误信息，但是也要给程序员的一些提示。

6. ERROR：指出虽然发生错误事件，但仍然不影响系统的继续运行。打印错误和异常信息，如果不想输出太多的日志，可以使用这个级别。【避免打印 ERROR 与抛出异常并存，因为会重复打印错误信息】

7. FATAL：指出每个严重的错误事件将会导致应用程序的退出。这个级别比较高了，属于重大错误，可以直接停止程序了。

8. OFF：最高等级的，用于关闭所有日志记录。


​        如果将 level 设置在某一个级别上，那么比此级别优先级高的日志都能打印出来。例如，如果设置优先级为WARN，那么 OFF、FATAL、ERROR、WARN 四个级别的日志能正常输出，而 INFO、DEBUG、TRACE、 ALL 级别的则会被忽略。Log4j2 建议只使用四个级别，优先级从高到低分别是 ERROR、WARN、INFO、DEBUG。

##### 5、Appender 设置在哪输出日志信息

> Console、RollingFile 和 SMTP 为常用 Appender 可以参考 log4j 的[官方文档](http://logging.apache.org/log4j/2.x/manual/appenders.html)

| Appenders                      | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| <u>File</u>                    | <u>普通地输出到本地文件</u>                                  |
| <u>RollingFile</u>             | <u>对日志文件进行封存</u>                                    |
| <u>RollingRandomAccessFile</u> | <u>与 RollingFile 类似，还可将日志压缩成 zip 文件，运行效率最高</u> |
| JMSQueue                       | 与 JMS 相关的日志输出                                        |
| JMSTopic                       | 与 JMS 相关的日志输出                                        |
| Rewrite                        | 对日志事件进行掩码或注入信息                                 |
| Flume                          | 将几个不同源的日志汇集、集中到一处                           |
| Routing                        | 在输出地之间进行筛选路由                                     |
| <u>SMTP</u>                    | <u>发送到指定邮件列表</u>                                    |
| Socket                         | 以普通格式发送到远程主机                                     |
| Syslog                         | 以 RFC 5424 格式发送到远程主机                               |
| Asynch                         | 异步地写入多个不同输出地                                     |
| <u>Console</u>                 | <u>输出到命令行控制台</u>                                    |
| Failover                       | 维护一个队列，尝试向其中的 Appender 依次输出日志，直到有一个成功为止 |

​        SMTP 主要用来给指定的 E-mail 发送日志事件。默认情况下只发送 ERROR 级别以上的日志。另外还需要导入 activation.jar 和 mail.jar 这两个jar包。

```xml
<Appenders>
    <SMTP name="Mail" subject="Error Log" 
          to="XXXXX@qq.com, XXXXX@126.com"
          from="XXXXX@163.com" 
          replyTo="XXXXX@163.com" 
          smtpProtocol="smtp" 
          smtpHost="smtp.163.com" 
          smtpPort="25" 
          bufferSize="50" 
          smtpDebug="false"
          smtpPassword="password" 
          smtpUsername="XXXXX@163.com">
    </SMTP>
</Appenders>
<Loggers>
    <Root level="error">
        <AppenderRef ref="Mail"/>
    </Root>
</Loggers>
```

##### 6、Layout 设置日志信息的输出格式

1. org.apache.log4j.HTMLLayout：以HTML表格形式布局。

2. org.apache.log4j.SimpleLayout：包含日志讯息的级别和讯息字符串。

3. org.apache.log4j.TTCCLayout：包含日志产生的时间、执行绪、类别等讯息。

4. org.apache.log4j.**PatterLayout**：可以灵活地指定布局格式。【常用|详解】


> 详细配置可以参考[官方文档](http://logging.apache.org/log4j/2.x/manual/layouts.html)。

​        PatternLayout 是最重要也是最常用的控制输出内容的节点，包括类名、时间、行号、日志级别、序号等都可以控制，同时还可以指定日志格式，可以使用正则表达式处理输出结果。PatternLayout 中包含的特殊字符包括\t、\n、\r、\f，用 \\ 输出单斜线，用 %% 输出 %。

###### 下表是 PatternLayout 的参数：

| 参数名                | 类型             | 描述                                                         |
| --------------------- | ---------------- | ------------------------------------------------------------ |
| charset               | String           | 输出的字符集。如果没有指定，则使用系统默认的字符集输出       |
| <u>pattern</u>        | String           | （单独列表格）                                               |
| replace               | RegexReplacement | 替换部分输出中的 String。这将会调用一个与 RegexReplacement 转换器相同的函数，不同的是这是针对整个消息的 |
| alwaysWriteExceptions | boolean          | 默认为 true。总是输出异常，即使没有定义异常对应的 pattern，也会被附在所有 pattern 的最后。设为 false 则不会输出异常 |
| header                | String           | 可选项。包含在每个日志文件的顶部                             |
| footer                | String           | 可选项。包含在每个日志文件的尾部                             |
| noConsoleNoAnsi       | boolean          | 默认为 false。如果为 true，且 System.console() 是 null，则不会输出 ANSI 转义码 |

###### 下表是 RegexReplacement 的参数：

| 参数名      | 类型   | 描述                               |
| ----------- | ------ | ---------------------------------- |
| regex       | String | Java 正则表达式                    |
| replacement | String | 任何匹配正则规则，被正则替换的后值 |

###### 下表是 pattern 属性具体描述（常用部分）：

| 参数名                                | 意义                                                         | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| %c{参数} 或 %logger{参数}             | 输出logger的名称，即语句private static final Logger LOG = LoggerFactory.getLogger(xxx.class)中的值，也可以使用其他字符串 | 默认不带参数则输出类以及全路径。如果参数是整数n（只支持正整数），则先将 logger 名称依照小数点(.)分割成 n 段，然后取右侧的 n 段；如果参数不是整数，则除了最右侧的一段，其他整段字符都用这个字符代替，保留小数点；不管怎么写，最右侧的一段都保持不变。【参数%c{1.1.!} 对应名称org.apache.com.test.Foo 输出 o.a.!.!.Foo】 |
| %d{参数}{时区}                        | 输出时间                                                     | 第一个大括号数可以是保留关键字，也可以是text.SimpleDateFormat字符拼接而成。保留关键字有：DEFAULT,ABSOLUTE, COMPACT, DATE, ISO8601, ISO8601_BASIC；第二个大括号中的参数是java.util.TimeZone.getTimeZone的值，可以设定时区【%d{yyyy-MM-dd HH:mm:ss.SSS}{GMT+8}输出 2019-12-12 15:30:28.972】 |
| %F 或 %file                           | 输出文件名                                                   | 仅仅是文件名称，如 Test.java，不包含路径名称                 |
| %highlight{pattern}{style}            | 高亮显示结果                                                 | 默认会按照不同的级别打出不一样的颜色                         |
| %l                                    | 输出完整的错误位置，如com.project.oms.test.test.app.tt(app.java:13) | 注意：使用该参数会影响日志输出的性能                         |
| %L                                    | 输出错误行号，如“13”                                         | 注意：使用该参数会影响日志输出的性能                         |
| %m 或 %msg 或 %message                | 输出信息，即 LOG.xxx(String msg) 中的 msg                    | 如果要打印 DEBUG 级别的日志，按规范需要判断是否开启 DEBUG 级别，这样可以减小内存消耗【if (LOG.isDebugEnabled()) { LOG.debug(msg); }】 |
| %M或%method                           | 输出方法名，如 “main”，“getMsg” 等字符串                     |                                                              |
| %n                                    | 换行符                                                       | 根据系统自行决定，如Windows是”\r\n”，Linux是”\n”             |
| %level{参数1}{参数2}{参数3}           | 参数1用来替换日志信息的级别，格式为：{level=label, level=label, …}，即使用 label 代替的字符串替换level。其中 level 为日志的级别，如WARN/DEBUG/ERROR/INFO；<br />参数2表示只保留前n个字符。格式为length=n，n为整型；<br />参数3表示结果是大写还是小写。<br />（参数1指定了label的字符串不受参数限制） | %level{ERROR=FSF, length=2, lowerCase=true}<br />logger.error会输出FSF，其他均只输出前两个字母，且小写，如wa/de/in/tr |
| %r 或 %relative                       | 输出自 JVM 启动以来到事件开始时的毫秒数                      |                                                              |
| replace{pattern}{regex}{substitution} | 将 pattern 的输出结果，按照正则表达式 regex 匹配后，使用 substitution 字符串替换 | 例如："%replace{%logger }{\.}{/}就是将所有%logger中的小数点(.)全部替换为斜杠，如果%logger是com.project.oms.test.test.app则输出为com/project/oms/test/test/app；pattern中可以写多个表达式，如%replace{%logger%msg%C}{\.}{/}%n |
| %sn或%sequenceNumber                  | 自增序号，每执行一次log事件，序号+1，是一个static变量。      |                                                              |
| %t或%thread                           | 创建logging事件的线程名                                      |                                                              |
| %u{RANDOM\|TIME}或%uuid{RANDOM\|TIME} | 依照一个随机数或当前机器的 MAC 和时间戳来随机生成一个 UUID   |                                                              |

> 忽略了用于控制输出颜色样式的 style 参数，花里胡哨的咱不搞。

###### pattern 的对齐修饰：

​        对齐修饰，可以指定信息的输出格式，如是否左对齐，是否留空格等。编写格式为在任何 pattern 和 % 之间加入一个小数，可以是正数，也可以是负数。如 %10.20c 表示对 LOG 的信息进行处理。%-10.20m 表示对 message 进行处理。整数表示右对齐，负数表示左对齐；整数位表示输出信息的最小 10 个字符，如果输出信息不够 10 个字符，将用空格补齐；小数位表示输出信息的最大字符数，如果超过 20 个字符，则只保留最后 20 个字符的信息（注意：保留的是后 20 个字符，而不是前 20 个字符）。

下表是一些示例：

| **格式** | **是否左对齐** | **最小宽度** | **最大宽度** | **说明**                                                     |
| -------- | -------------- | ------------ | ------------ | ------------------------------------------------------------ |
| %20      | 右对齐         | 20           | 无           | 右对齐，不足 20 个字符则在信息前面用空格补足，超过 20 个字符则保留原信息 |
| %-20     | 左对齐         | 20           | 无           | 左对齐，不足 20 个字符则在信息后面用空格补足，超过 20 个字符则保留原信息 |
| %.30     | 不对齐         | 无           | 30           | 如果信息超过 30 个字符，则只保留最后 30 个字符               |
| %20.30   | 右对齐         | 20           | 30           | 右对齐，不足 20 个字符则在信息前面用空格补足，超过 30 个字符则只保留最后 30 个字符 |
| %-20.30  | 左对齐         | 20           | 30           | 左对齐，不足 20 个字符则在信息后面用空格补足，超过 30 个字符则只保留最后 30 个字符 |

##### 7、Filters

​        Filter 可以过滤 log 事件，并控制 log 输出。例如 ThresholdFilter 可以实现单个 log 级别的过滤功能。注意：如果同时设置了过滤器和 logger 标签的输出级别，它们是同级，会取最高级别的日志输出。
