#### logback配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!--  项目名称 -->
    <springProperty scope="context" name="service_name" source="spring.application.name" defaultValue="spring-boot-service"/>
    <!-- 格式化输出：%date表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
    <property name="LOG_PATTERN" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n" />
    <!-- 定义日志存储的路径，不要配置相对路径 -->
    <property name="LOG_FILE_PATH" value="E:/logs/demo.%d{yyyy-MM-dd}.%i.log" />
    <!-- 日志文件根节点 -->
    <property name="PATH" value="/var/log/matterhorn/cet-oil-task-service/%d{yyyy-MM-dd}"/>

    <!-- 控制台输出日志 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 按照上面配置的LOG_PATTERN来打印日志 -->
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--  每天生成一个日志文件，保存30天的日志文件。 -->
    <appender name="DEFAULT_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${PATH}/0服务总日志.%i.log</fileNamePattern>
            <!-- 日志存储30天 -->
            <maxHistory>30</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- 日志文件的最大大小 -->
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 任务日志记录位置   -->
    <appender name="EFFECT_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${PATH}/1能效相关日志.%i.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 任务日志记录位置   -->
    <appender name="PRODUCT_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${PATH}/2产量相关日志.%i.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 任务日志记录位置   -->
    <appender name="QUANTITY_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${PATH}/3物理量相关日志.%i.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 任务日志记录位置   -->
    <appender name="OTHER_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${PATH}/4其他日志.%i.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

	<!--additivity设置为true,则会在自己的日志和主日志中分别出现一次，因此不需要进行控制台输出-->
    <logger name="effectLog" additivity="true" level="INFO">
        <appender-ref ref="EFFECT_FILE"/>
	<!--<appender-ref ref="CONSOLE"/>-->
    </logger>

    <logger name="productLog" additivity="true" level="INFO">
        <appender-ref ref="PRODUCT_FILE"/>
    </logger>

    <logger name="quantityLog" additivity="true" level="INFO">
        <appender-ref ref="QUANTITY_FILE"/>
    </logger>

    <logger name="otherLog" additivity="true" level="INFO">
        <appender-ref ref="OTHER_FILE"/>
    </logger>

    <!-- 日志输出级别 常用的日志级别按照从高到低依次为：ERROR、WARN、INFO、DEBUG。 -->
    <root level="INFO">
        <!--对应appender name="DEFAULT_FILE"。 -->
        <appender-ref ref="DEFAULT_FILE" />
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```



#### 使用

application.yml

```yml
logging:
	config: classpath:logback-fly.xml
```

Test.java

```java
public class Test {
    private static final Logger effectLog = LoggerFactory.getLogger("effectLog");
    
	public void test(){
        effectLog.info("test");
    }
}
```