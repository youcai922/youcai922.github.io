#### 简介

Logback是SpringBoot内置的日志处理框架。spring-boot-start包含的spring-boot-starter-logging，该依赖内容就是springboot默认的日志框架Logback

官方文档：https://logback.qos.ch/manual/



#### springBoot默认Logback配置

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Base logback configuration provided for compatibility with Spring Boot 1.1
-->

<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```



#### 自定义配置

- 根节点<configuration>

  ```xml
  <configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <!-- 其他配置省略-->  
  </configuration>
  ```

  - scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true
  - scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
  - debug：当此属性设置为ture时，将答应logback内部日志信息，实时查询logback运行状态，默认值为false

- 子节点<contextName>

  ```xml
  <contextName>logback-demo</contextName>
  
  <layout class="ch.qos.logback.classic.PatternLayout">
      <pattern>
          <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
      </pattern>
  </layout>
  ```

  每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用< contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。可以通过%contextName来打印日志上下文名称，一般来说我们不用这个属性，可有可无。

- 子节点<property>

  ```xml
  <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
  ```

  用来定义变量值的标签，< property> 有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值,通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

  注：多环境配置下，通过 application.yml 传递参数过来，< property >取不到环境参数，得用< springProperty >。

  ```xml
  <springProperty scope="context" name="APP_NAME" source="spring.application.name" defaultValue="springBoot"/>
  ```

- 子节点<appender>
  appender用来格式化日志输出节点，有两个属性name和class，class用来指定哪种输出策略，常用就是控制台输出策略和文件输出策略。

  **ConsoleAppender：**就想他的名字一样，将日志信息打印到控制台上，更加准确的说：使用System.out或者System.err方式输出，主要子标签有：encoder，target

  **FileAppender：**用于将日志信息输出到文件中，主要子标签有：append，encoder，file

  **RollingFileAppender：**从名字我们就可以得出：FileAppender是RollingFileAppender的父类。即RollingFileAppender继承 FIleAppender类。功能：能够动态的创建一个文件。也就是说：到满足一定的条件，就会创建一个新的文 件，然后将日志写入到新的文件中。有两个重要的标签与rolingFileAppender进行交互：RollingPolicy，TriggeringPolicy，主要子标签：file，append，encoder，rollingPolicy，triggerPolicy

  - file:被写入的文件名，可以是相对目录也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值

  - tartget：设置是System.out还是System.err方式输出，默认为System.out

  - append:如果是true，日志被追加到文件结尾，如果是false，清空现存文件，默认是true

  - product:

    - FileAppender:如果是ture，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是false
    - RollingFileAppender：如果是ture，不支持FixedWindowRollingPolicy，支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空

  - layout和encoder

    ```xml
    <!--输出到控制台 ConsoleAppender-->
    <appender name="consoleLog1" class="ch.qos.logback.core.ConsoleAppender">
        <layout>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </layout>
    </appender>
    
    <!--输出到控制台 ConsoleAppender-->
    <appender name="consoleLog2" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d -2 %msg%n</pattern>
        </encoder>
    </appender>
    <!--设置项目日志输出级别为INFO-->
    <root level="INFO">
        <appender-ref ref="consoleLog1"/>
        <appender-ref ref="consoleLog2"/>
    </root>
    ```

#### layout和encoder区别

- encoder：主要工作有两个：①将一个event时间转换成一组byte数组，②将转换后 对字节数据输出到文件中。
- encoder组件是在0.9.19版本之后才引进来的。在以前的版本中，appender是使用layout（将一个event事件转换成一个字符串），然后使用【java.io.writer】对象将字符串写入到文件中。
- 自从0.9.19版本之后，Fileappender和它的子类是期望使用encoder，不再使用layout。
- 其中layout仅仅完成了将一个event时间转换成一个字符串这一个功能。此外，layout不能控制将字符串写出到文件中。layoutbunengzhengheevent事件到一组中。与enoder相比，不仅仅能按照格式进行转化，而且还能将数据写入到文件中。

因为layout已经不再推荐使用了，那么这里重点讲一下encoder。

其中patternLayoutEncoder是最常用的encoder，也就是默认的，他包含可patternLayout大部分的工作。

````xml
<--PatternLayoutEncoder-->
<--1.考虑到PatternLayout是layout中最常用的组件，所以logback人员开发出了patternLayoutEncoder类，这个类是LayoutWrappingEncoder的扩展，这个类包含了PatternLayout。-->
<--2.immediateFlush标签与LayoutWrappingEncoder是一样的。默认值为【true】。这样的话，在已存在的项目就算没有正常情况下的关闭，也能记录所有的日志信息到磁盘上，不会丢失任何日志信息。因为是立即刷新。如果将【immediateFlush】设置为【false】，可能就是五倍的原来的logger接入量。但是可能会丢失日志信息在没有正常关闭项目的情况下。例如：-->
<appender name="FILE" class="ch.qos.logback.core.FileAppender"> 
  <file>foo.log</file>
  <encoder><!--默认就是PatternLayoutEncoder类-->
    <pattern>%d %-5level [%thread] %logger{0}: %msg%n</pattern>
    <!-- this quadruples logging throughput -->
    <immediateFlush>false</immediateFlush>
  </encoder> 
</appender>

<--3.如果想在文件的开头打印出日志的格式信息：即打印日志的模式。使用【outputPatternAsHeader】标签，并设置为【true】.默认值为【false】。例如：-->
<!--输出到控制台 ConsoleAppender-->
<appender name="consoleLog2" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d -2 %msg%n</pattern>
        <outputPatternAsHeader>true</outputPatternAsHeader>
    </encoder>
</appender>
<--他就会在打印开始之前第一句输出日志格式，如：#logback.classic pattern: %d -2 %msg%n-->
````

patternLayoutEncoder类既有layout将一个事件转化为字符串，又有将字符创写入到文件中的作用。他是encoder标签的默认类实例。



#### filter

logback具有过滤器的支持，logback允许给日志记录器appender配置一个或多个Filter（或者给整体配置一个或多个TurboFilter），来控制：当满足过滤器指定的条件时，猜记录日志（或不满足条件时，拒绝记录日志）。logback支持自定义过滤器，当然logback也自带了一些常用的过滤器，在绝大多数的时候，自带的过滤器其实就够用了，一般是不需要自定义过滤器的。

logback提供的过滤器支持主要分为两大类：

- ch.qos.logback.core.filter.Filter
- ch.qos.logback.classic.turbo.TurboFilter

Filter与TurboFilter自带的几种常用过滤器

| 过滤器      | 来源   | 说明                                                         | 相对常用 |
| ----------- | ------ | ------------------------------------------------------------ | -------- |
| LevelFilter | Filter | 对指定level的日志进行记录(或不记录)，对不等于指定level的日志不记录(或进行记录) | 是       |
|ThresholdFilter		|Filter			|对大于或等于指定level的日志进行记录(或不记录)，对小于指定level的日志不记录(或进行记录) 提示：info级别是大于debug的	|是|
|EvaluatorFilter		|Filter			|对满足指定表达式的日志进行记录(或不记录)，对不满足指定表达式的日志不作记录(或进行记录)								|是|
|MDCFilter				|TurboFilter	|若MDC域中存在指定的key-value，则进行记录，否者不作记录																|是|
|DuplicateMessageFilter	|TurboFilter	|根据配置不记录多余的重复的日志																						|是|
|MarkerFilter			|TurboFilter	|针对带有指定标记的日志，进行记录(或不作记录)																		|否|

TurboFilter的性能是优于Filter的，这是因为TurboFilter的作用时机是在创建日志事件ILoggingEvent对象之前，而Filter的作用时机是在创建之后。若一个日志注定是会被过滤掉不记录的，那么创建ILoggingEvent对象(包括后续的参数组装方法调用等)这个步骤无疑是非常消耗性能的。


- ThresholdFilter：

  ThresholdFilter为系统定义的拦截器，例如我们用ThresholdFilter来过滤调ERROR级别一下的日志不输出到文件中。如果不用记得住是调，不然控制台会没有日志

  ```xml
  <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>ERROR</level>
  </filter>
  ```

- LevelFilter：

  如果只想要Info级别的日志，只是过滤info还是会输出Error日志，因为Error的级别高，所以我们使用下面的策略，可以避免输出Error的日志

  ```xml
  <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <!--过滤 Error-->
      <level>ERROR</level>
      <!--匹配到就禁止-->
      <onMatch>DENY</onMatch>
      <!--没有匹配到就允许-->
      <onMismatch>ACCEPT</onMismatch>
  </filter>
  ```

  FilterReply有三种枚举值：

  - DENY：表示不用看后面的过滤器了，这里就给拒绝了，不作记录。

  - NEUTRAL：表示需不需要记录，还需要看后面的过滤器。若所有过滤器返回的全部都是NEUTRAL，那么需要记录日志。
  - ACCEPT：表示不用看后面的过滤器了，这里就给直接同意了，需要记录。

#### rollingPolicy

- TimeBasedRollingPolicy: 它根据时间来制定滚动策略.时间滚动策略可以基于时间滚动按时间生成日志。

  下面是官网给出的示例

  ```xml
  <configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logFile.log</file>
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- daily rollover -->
        <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>
        <!-- keep 30 days' worth of history capped at 3GB total size -->
        <maxHistory>30</maxHistory>
        <totalSizeCap>3GB</totalSizeCap>
      </rollingPolicy>
  
      <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
    </appender> 
  
    <root level="DEBUG">
      <appender-ref ref="FILE" />
    </root>
  </configuration>
  ```

  紧跟着有给出了多个JVM写同一个日志文件的配置，主要是加一行开启prudent（节俭）模式

  ```xml
  <configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <!-- Support multiple-JVM writing to the same log file -->
      <prudent>true</prudent>
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>30</maxHistory> 
        <totalSizeCap>3GB</totalSizeCap>
      </rollingPolicy>
  
      <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
    </appender> 
  
    <root level="DEBUG">
      <appender-ref ref="FILE" />
    </root>
  </configuration>
  ```

- SizeAndTimeBasedRollingPolicy：基于大小和时间的滚动策略

- 这个策略出现的原因就是对时间滚动策略的异格补充，使其不仅按时间进行生成而且考虑到文件大小的原因，因为在基于时间的滚动策略生成的日志文件，只是对一段时间总的日志大小做了限定，这就会造成个别日志文件过大，后期传递，阅读困难的问题，所以就有了这第二个策略

  ```xml
  <configuration>
    <appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>mylog.txt</file>
      <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!-- rollover daily -->
        <fileNamePattern>mylog-%d{yyyy-MM-dd}.%i.txt</fileNamePattern>
         <!-- each file should be at most 100MB, keep 60 days worth of history, but at most 20GB -->
         <maxFileSize>100MB</maxFileSize>    
         <maxHistory>60</maxHistory>
         <totalSizeCap>20GB</totalSizeCap>
      </rollingPolicy>
      <encoder>
        <pattern>%msg%n</pattern>
      </encoder>
    </appender>
  
    <root level="DEBUG">
      <appender-ref ref="ROLLING" />
    </root>
      
  </configuration>
  ```

  