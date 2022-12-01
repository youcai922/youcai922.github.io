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

- 根节点< configuration >

  ```xml
  <configuration scan="true" scanPeriod="60 seconds" debug="false">  
      <!-- 其他配置省略-->  
  </configuration>
  ```

  - scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true
  - scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
  - debug：当此属性设置为ture时，将答应logback内部日志信息，实时查询logback运行状态，默认值为false

- 子节点< contextName >

  ```xml
  <contextName>logback-demo</contextName>
  
  <layout class="ch.qos.logback.classic.PatternLayout">
      <pattern>
          <pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>
      </pattern>
  </layout>
  ```

  每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用< contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。可以通过%contextName来打印日志上下文名称，一般来说我们不用这个属性，可有可无。

- 子节点< property >

  ```xml
  <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
  ```

  用来定义变量值的标签，< property> 有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值,通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

  注：多环境配置下，通过 application.yml 传递参数过来，< property >取不到环境参数，得用< springProperty >。

  ```xml
  <springProperty scope="context" name="APP_NAME" source="spring.application.name" defaultValue="springBoot"/>
  ```

- 子节点< appender>
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

  这个策略出现的原因就是对时间滚动策略的异格补充，使其不仅按时间进行生成而且考虑到文件大小的原因，因为在基于时间的滚动策略生成的日志文件，只是对一段时间总的日志大小做了限定，这就会造成个别日志文件过大，后期传递，阅读困难的问题，所以就有了这第二个策略

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

- FixedWindowRollingPolicy：基于固定窗口的滚动策略

  这个策略的出现，应该是因为需要日志文件保持某个特定的数量，防止滚动策略导致过多的日志文件出现。这个策略出现得配合triggering，给一个什么时候日志滚动一次的控制，这部分是跟上面两个策略所不一样的地方。

  ```xml
  <configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>test.log</file>
  
      <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
        <fileNamePattern>tests.%i.log.zip</fileNamePattern>
        <minIndex>1</minIndex>
        <maxIndex>3</maxIndex>
      </rollingPolicy>
  
      <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <maxFileSize>5MB</maxFileSize>
      </triggeringPolicy>
      <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
    </appender>
          
    <root level="DEBUG">
      <appender-ref ref="FILE" />
    </root>
  </configuration>
  ```

  注意：在RollingFileAppender中有一个file标签，也是设置文件的名称的。file可以设置 也可以不设置。如果你设置了file标签的话，他就不会转换到新的文件中。所有的日志 信息将会输入到同一个文件中。如果file标签没有设置。文件的名称就会在每一个阶段 由filenamePattern计算得出。

  - fileNamePattern:这是一个强制的标签。他的值可以包含：文件的名称、适当的%d转 换说明符。这个%d说明符可以包含一个【日期和时间】的模式。其中【模式】类似于 【SimpleDateFormat】类。如果这个【模式】没有写的话，默认就是【yyyy-MM-dd】的模式。 转换文件的名称从fileNamePattern中得到
  - **maxHistory：**这是一个可选的标签。以异步方式删除较旧的文件,例如，如果您指定每月滚动，并将maxHistory设置为6，则将保留6个月的归档文件，并删除6个月以上的文件。
  - **totalSizeCap：**这是一个可选的标签。这是所有日志文件的总大小空间。当日志文件的空间超过了设置的最大 空间数量，就会删除旧的文件。注意：这个标签必须和maxHistory标签一起使用。
  - **cleanHistoryOnStart:**如果设置为true，则将在追加程序启动时执行归档删除。默认情况下，此属性设置为false。

#### 子节点:< loger>

用来设置某一个包或者具体的某一个类的日志打印级别、以及指定< appender >。< loger >仅有一个name属性，一个可选的level和一个可选的addtivity属性。

- name:用来指定受此loger约束的某一个包或者具体的某一个类。

- level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前loger将会继承上级的级别。

addtivity:是否向上级loger传递打印信息。默认是true。

##### loger在实际使用的时候有两种情况

- 带有loger的配置，不指定级别，不指定appender

  ```xml
  <logger name="com.xf.controller.TestLogController"/>
  ```

  将TestLogController类的日志的打印，但是并没有设置打印级别，所以继承他的上级的日志级别info，没有设置addtivity，默认为true，将此loger的打印信息向上级传递，没有设置appender，此loger本身不打印任何信息。

  < root level=“info”>将root的打印级别设置为“info”，指定了名字为“console”的appender。当执行com.xf.controller.TestLogController类的testLog方法时，所以首先执行< logger name=“com.xf.controller.TestLogController”/>，将级别为“info”及大于“info”的日志信息传递给root，本身并不打印；root接到下级传递的信息，交给已经配置好的名为“console”的appender处理，“console” appender 将信息打印到控制台；

- 带有多个loger的配置，指定级别，指定appender

  ```xml
  <configuration>
      <logger name="com.xf.controller.TestLogController" level="WARN" additivity="false">
          <appender-ref ref="console"/>
      </logger>
  </configuration>
  ```

  控制com.xf.controller.TestLogController类的日志打印，打印级别为“WARN”;additivity属性为false，表示此loger的打印信息不再向上级传递;指定了名字为“console”的appender;

  这时候执行com.xf.controller.TestLogController类的login方法时，先执行< logger name=“com.xf.controller.TestLogController” level=“WARN” additivity=“false”>,将级别为“WARN”及大于“WARN”的日志信息交给此loger指定的名为“console”的appender处理，在控制台中打出日志，不再向上级root传递打印信息。

  注：当然如果你把additivity="false"改成additivity="true"的话，就会打印两次，因为打印信息向上级传递，logger本身打印一次，root接到后又打印一次。
  

#### 子节点:< root >

root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性。level默认是DEBUG。

level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。

可以包含零个或多个元素，标识这个appender将会添加到这个loger。

```xml
<root level="debug">
  <appender-ref ref="console" />
  <appender-ref ref="file" />
</root>
```

#### 多环境配置

< springProfile > 标签允许你自由的包含或排除基于激活的Spring profiles的配置的一部分。在< configuration >元素的任何地方都支持Profile部分。使用name属性来指定哪一个profile接受配置。多个profiles可以用一个逗号分隔的列表来指定。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev, staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

- 配置演示

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!--scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。-->
  <!--debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。-->
  <!--scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。-->
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
  
      <!-- 全局变量 -->
      <!--日志文件保存路径-->
      <springProperty scope="context" name="logPath" source="logging.file.path" defaultValue="logs"/>
      <!--应用名称-->
      <springProperty scope="context" name="appName" source="spring.application.name" defaultValue="springBoot"/>
  
      <!-- 通用日志文件输出appender -->
      <appender name="COMMON_FILE_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!--日志名称，如果没有File 属性，那么只会使用FileNamePattern的文件路径规则
              如果同时有<File>和<FileNamePattern>，那么当天日志是<File>，明天会自动把今天
              的日志改名为今天的日期。即，<File> 的日志都是当天的。
          -->
          <File>${logPath}/${appName}.log</File>
          <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <!--文件路径,定义了日志的切分方式——把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间-->
              <FileNamePattern>${logPath}/${appName}.%d{yyyy-MM-dd}.log</FileNamePattern>
              <!--只保留最近30天的日志-->
              <maxHistory>30</maxHistory>
              <!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
              <totalSizeCap>1GB</totalSizeCap>
          </rollingPolicy>
          <!--日志输出编码格式化-->
          <encoder>
              <charset>UTF-8</charset>
              <!--<pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>-->
              <pattern>%d [%thread] %-5level %logger{36} %line - %msg%n</pattern>
          </encoder>
      </appender>
  
      <!-- 开发环境,多个使用逗号隔开,本地开发环境只输出控制台日志-->
      <springProfile name="dev">
          <!-- 本地开发控制栏彩色日志 -->
          <!-- 彩色日志依赖的渲染类 -->
          <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
          <conversionRule conversionWord="wex"
                          converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
          <conversionRule conversionWord="wEx"
                          converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
          <!-- 彩色日志格式 -->
          <property name="CONSOLE_LOG_PATTERN"
                    value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
  
          <!-- Console 输出设置 -->
          <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
              <layout>
                  <pattern>${CONSOLE_LOG_PATTERN}</pattern>
              </layout>
          </appender>
          <!-- 控制台不打印tomcat启动过程中的JAR包扫描警告 -->
          <logger name="o.a.tomcat.util.scan.StandardJarScanner" level="ERROR" additivity="false">
              <appender-ref ref="CONSOLE"/>
          </logger>
          <!--指定最基础的日志输出级别-->
          <root level="INFO">
              <appender-ref ref="CONSOLE"/>
          </root>
      </springProfile>
  
      <!-- 测试环境,多个使用逗号隔开,测试环境输出INFO级别日志,只以文件形式输出-->
      <springProfile name="test">
          <logger name="com.xf" level="INFO" additivity="false">
              <appender-ref ref="COMMON_FILE_LOG"/>
          </logger>
          <root level="INFO">
              <appender-ref ref="COMMON_FILE_LOG"/>
          </root>
      </springProfile>
  
      <!-- 生产环境,多个使用逗号隔开,生产环境输出ERROR级别日志,只以文件形式输出-->
      <springProfile name="prod">
          <root level="ERROR">
              <appender-ref ref="COMMON_FILE_LOG"/>
          </root>
      </springProfile>
  
  </configuration>
  ```

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!--scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。-->
  <!--debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。-->
  <!--scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。-->
  <configuration debug="false" scan="false">
      <springProperty scop="context" name="spring.application.name" source="spring.application.name" defaultValue="springboot"/>
      <property name="LOG_HOME" value="logs/${spring.application.name}"/>
  
      <!-- 日志格式和颜色渲染 -->
      <!-- 彩色日志依赖的渲染类 -->
      <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
      <conversionRule conversionWord="wex"
                      converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
      <conversionRule conversionWord="wEx"
                      converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
      <!-- 彩色日志格式 -->
      <property name="CONSOLE_LOG_PATTERN"
                value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
  
  
      <!-- 把日志输出到控制台-->
      <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
          <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>debug</level>
          </filter>
          <encoder>
              <pattern>${CONSOLE_LOG_PATTERN}</pattern>
              <charset>UTF-8</charset>
          </encoder>
      </appender>
  
  
      <!--把debug日志输出-->
      <appender name="debug" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${LOG_HOME}/debug.log</file>
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>DEBUG</level>
          </filter>
          <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
              <!--日志文件输出的文件名-->
              <fileNamePattern>${LOG_HOME}/debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
              <!-- 日志文件最大尺寸 -->
              <maxFileSize>100MB</maxFileSize>
              <!--日志文件保留天数-->
              <MaxHistory>30</MaxHistory>
          </rollingPolicy>
          <encoder>
              <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
              <charset>UTF-8</charset>
          </encoder>
      </appender>
  
  
      <!-- 记录错误日志到单独文件 error.log -->
      <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${LOG_HOME}/error.log</file>
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>ERROR</level>
          </filter>
          <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
              <!--日志文件输出的文件名-->
              <fileNamePattern>${LOG_HOME}/error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
              <!-- 日志文件最大尺寸 -->
              <maxFileSize>100MB</maxFileSize>
              <!--日志文件保留天数-->
              <MaxHistory>30</MaxHistory>
          </rollingPolicy>
          <encoder>
              <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
              <charset>UTF-8</charset>
          </encoder>
      </appender>
  
      <!-- 记录错误日志到单独文件 info.log -->
      <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${LOG_HOME}/info.log</file>
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>INFO</level>
          </filter>
          <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
              <!--日志文件输出的文件名-->
              <fileNamePattern>${LOG_HOME}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
              <!-- 日志文件最大尺寸 -->
              <maxFileSize>100MB</maxFileSize>
              <!--日志文件保留天数-->
              <MaxHistory>30</MaxHistory>
          </rollingPolicy>
          <encoder>
              <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
              <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
              <charset>UTF-8</charset>
          </encoder>
      </appender>
  
      <!-- 下面配置一些第三方包的日志过滤级别，用于避免刷屏 -->
      <logger name="org.springframework.web" level="INFO"/>
  
      <!-- Log日志级别从高到低排序
      ERROR：用户程序报错，必须解决的时候使用此级别打印日志
      WARN：警告，不会影响程序的运行，但是值得注意。
      INFO：一般处理业务逻辑的时候使用，就跟 system.err打印一样
      DEBUG：一般放于程序的某个关键点的地方，用于打印一个变量值或者一个方法返回的信息之类的信息
      TRACE：一般不会使用，在日志里边也不会打印出来，好像是很低的一个日志级别。
       -->
  
      <!--  定义本地环境日志级别  -->
      <springProfile name="local">
          <logger name="com.xf" level="DEBUG"/><!-- 定义将com.xf包下的最低级别日志信息 -->
          <root level="INFO"><!-- 系统全局日志输出最低级别，但不包括com.xf包 -->
              <appender-ref ref="console"/>
          </root>
      </springProfile>
  
      <!--  定义测试环境日志级别  -->
      <springProfile name="dev">
          <logger name="com.xf" level="DEBUG"/>
          <root level="INFO">
              <appender-ref ref="console"/>
              <appender-ref ref="debug"/>
          </root>
      </springProfile>
  
  
      <!--  定义线上环境日志级别  -->
      <springProfile name="online">
          <root level="INFO">
              <appender-ref ref="error"/>
          </root>
      </springProfile>
  
  </configuration>
  ```

  

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE configuration>
  <configuration>
      <!--引用默认日志配置-->
      <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
      <!--使用默认的控制台日志输出实现-->
      <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
      <!--应用名称-->
      <springProperty scope="context" name="APP_NAME" source="spring.application.name" defaultValue="springBoot"/>
      <!--日志文件保存路径-->
      <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
      <!--LogStash访问host-->
      <springProperty name="LOG_STASH_HOST" scope="context" source="logstash.host" defaultValue="localhost"/>
  
      <!--DEBUG日志输出到文件-->
      <appender name="FILE_DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!--输出DEBUG以上级别日志-->
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>DEBUG</level>
          </filter>
          <encoder>
              <!--设置为默认的文件日志格式-->
              <pattern>${FILE_LOG_PATTERN}</pattern>
              <charset>UTF-8</charset>
          </encoder>
          <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
              <!--设置文件命名格式-->
              <fileNamePattern>${LOG_FILE_PATH}/debug/${APP_NAME}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
              <!--设置日志文件大小，超过就重新生成文件，默认10M-->
              <maxFileSize>${LOG_FILE_MAX_SIZE:-10MB}</maxFileSize>
              <!--日志文件保留天数，默认30天-->
              <maxHistory>${LOG_FILE_MAX_HISTORY:-30}</maxHistory>
          </rollingPolicy>
      </appender>
  
      <!--ERROR日志输出到文件-->
      <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!--只输出ERROR级别的日志-->
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>ERROR</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
          <encoder>
              <!--设置为默认的文件日志格式-->
              <pattern>${FILE_LOG_PATTERN}</pattern>
              <charset>UTF-8</charset>
          </encoder>
          <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
              <!--设置文件命名格式-->
              <fileNamePattern>${LOG_FILE_PATH}/error/${APP_NAME}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
              <!--设置日志文件大小，超过就重新生成文件，默认10M-->
              <maxFileSize>${LOG_FILE_MAX_SIZE:-10MB}</maxFileSize>
              <!--日志文件保留天数，默认30天-->
              <maxHistory>${LOG_FILE_MAX_HISTORY:-30}</maxHistory>
          </rollingPolicy>
      </appender>
  
      <!--DEBUG日志输出到LogStash-->
      <appender name="LOG_STASH_DEBUG" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>DEBUG</level>
          </filter>
          <destination>${LOG_STASH_HOST}:4560</destination>
          <encoder charset="UTF-8" class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
              <providers>
                  <timestamp>
                      <timeZone>Asia/Shanghai</timeZone>
                  </timestamp>
                  <!--自定义日志输出格式-->
                  <pattern>
                      <pattern>
                          {
                          "project": "mall",
                          "level": "%level",
                          "service": "${APP_NAME:-}",
                          "pid": "${PID:-}",
                          "thread": "%thread",
                          "class": "%logger",
                          "message": "%message",
                          "stack_trace": "%exception{20}"
                          }
                      </pattern>
                  </pattern>
              </providers>
          </encoder>
      </appender>
  
      <!--ERROR日志输出到LogStash-->
      <appender name="LOG_STASH_ERROR" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>ERROR</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
          <destination>${LOG_STASH_HOST}:4561</destination>
          <encoder charset="UTF-8" class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
              <providers>
                  <timestamp>
                      <timeZone>Asia/Shanghai</timeZone>
                  </timestamp>
                  <!--自定义日志输出格式-->
                  <pattern>
                      <pattern>
                          {
                          "project": "mall",
                          "level": "%level",
                          "service": "${APP_NAME:-}",
                          "pid": "${PID:-}",
                          "thread": "%thread",
                          "class": "%logger",
                          "message": "%message",
                          "stack_trace": "%exception{20}"
                          }
                      </pattern>
                  </pattern>
              </providers>
          </encoder>
      </appender>
  
      <!--业务日志输出到LogStash-->
      <appender name="LOG_STASH_BUSINESS" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
          <destination>${LOG_STASH_HOST}:4562</destination>
          <encoder charset="UTF-8" class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
              <providers>
                  <timestamp>
                      <timeZone>Asia/Shanghai</timeZone>
                  </timestamp>
                  <!--自定义日志输出格式-->
                  <pattern>
                      <pattern>
                          {
                          "project": "xxx",
                          "level": "%level",
                          "service": "${APP_NAME:-}",
                          "pid": "${PID:-}",
                          "thread": "%thread",
                          "class": "%logger",
                          "message": "%message",
                          "stack_trace": "%exception{20}"
                          }
                      </pattern>
                  </pattern>
              </providers>
          </encoder>
      </appender>
  
      <!--接口访问记录日志输出到LogStash-->
      <appender name="LOG_STASH_RECORD" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
          <destination>${LOG_STASH_HOST}:4563</destination>
          <encoder charset="UTF-8" class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
              <providers>
                  <timestamp>
                      <timeZone>Asia/Shanghai</timeZone>
                  </timestamp>
                  <!--自定义日志输出格式-->
                  <pattern>
                      <pattern>
                          {
                          "project": "xxx",
                          "level": "%level",
                          "service": "${APP_NAME:-}",
                          "class": "%logger",
                          "message": "%message"
                          }
                      </pattern>
                  </pattern>
              </providers>
          </encoder>
      </appender>
  
      <!--控制框架输出日志-->
      <logger name="org.slf4j" level="INFO"/>
      <logger name="springfox" level="INFO"/>
      <logger name="io.swagger" level="INFO"/>
      <logger name="org.springframework" level="INFO"/>
      <logger name="org.hibernate.validator" level="INFO"/>
  
  
      <logger name="com.xf.common.log.WebLogAspect" level="DEBUG">
          <appender-ref ref="LOG_STASH_RECORD"/>
      </logger>
  
      <logger name="com.xf" level="DEBUG">
          <appender-ref ref="LOG_STASH_BUSINESS"/>
      </logger>
  
      <root>
          <appender-ref ref="CONSOLE"/>
          <appender-ref ref="FILE_DEBUG"/>
          <appender-ref ref="FILE_ERROR"/>
          <appender-ref ref="LOG_STASH_DEBUG"/>
          <appender-ref ref="LOG_STASH_ERROR"/>
      </root>
  </configuration>
  ```

