# Spring Boot 整合log4j2

### 本文内容为: Spring Boot 整合log4j2 并配置自动归档与删除,以及控制台日志答应时添加颜色显示

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在spring boot 项目中日志框架默认使用的是logback, 而logback 早在2017年就已经不再更新稳定版本了，并且Apache日志框架log4g2(作为log4g的升级版本)，性能优异(如下图)，尤其是异步性能拉开logback与log4j一大截，并且默认以零GC的模式（不会由于log4j2导致GC）运行。除此之外，log4j2 还拥有更高性能I/O写入支持`，`更强大的参数格式化功能。

![pic1][pic1]

![pic2][]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来将介绍如何在spring boot中使用log4j2替代logback框架

1. 在项目的pom文件中修改配置如下:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <!-- 去掉logback的默认配置 -->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- 添加log4j2的依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
```

2. 在项目的`/src/main/resource`文件夹下创建`log4j2.xml`文件并填充以下内容(PS:log4j2的配置除了xml也可以使用yml或者json的配置文配置):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration monitorInterval="5">
    <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->

    <!--变量配置-->
    <Properties>
         <!-- 定义日志存储的路径 -->
        <property name="FILE_PATH" value="logs" />
        <property name="FILE_NAME" value="spring" />
        <Property name="LOG_EXCEPTION_CONVERSION_WORD">%xwEx</Property>
        <Property name="LOG_LEVEL_PATTERN">%5p</Property>
        <Property name="LOG_DATEFORMAT_PATTERN">yyyy-MM-dd HH:mm:ss.SSS</Property>
        <!-- 控制台日志格式化方案，在控制台中可以打印出颜色 -->
        <Property name="CONSOLE_LOG_PATTERN">%clr{%d{${sys:LOG_DATEFORMAT_PATTERN}}}{faint} %clr{${sys:LOG_LEVEL_PATTERN}} %clr{%pid}{magenta} %clr{---}{faint} %clr{[%15.15t]}{faint} %clr{%-40.40c{1.}}{cyan} %clr{:}{faint} %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
        <!-- 输入到文件中的格式化方案 -->
        <Property name="FILE_LOG_PATTERN">%d{${LOG_DATEFORMAT_PATTERN}} ${LOG_LEVEL_PATTERN} %pid --- [%t] %-40.40c{1.} : %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
        <!-- 以上方案参考spring boot项目的默认配置：https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml -->
    </Properties>

    <appenders>
        
        <!-- 配置控制台日志输出 -->
        <console name="Console" target="SYSTEM_OUT">
            <!--输出日志的格式-->
            <PatternLayout pattern="${CONSOLE_LOG_PATTERN}"/>
            <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
        </console>

        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFileInfo" fileName="${env:FILE_PATH}/spring.log" filePattern="${env:FILE_PATH}/${FILE_NAME}.log.%d{yyyy-MM-dd}_%i.log.gz">
            <Filters>
                <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
                <!-- 只记录ERROR级别日志信息，程序打印的其他信息不会被记录 -->
                <!-- 此level设置的日志级别，是过滤日志文件中打印出的日志信息，和Root的level有所区别 -->
                <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY" />
            </Filters>
            <PatternLayout pattern="${env:FILE_LOG_PATTERN}"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1, 单位与filePattern中设置的时间最小单位一致-->
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="1MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一时间范围下7个文件开始覆盖-->
            <DefaultRolloverStrategy max="10">
                <Delete basePath="${env:FILE_PATH}/" maxDepth="2">
                    <IfFileName glob="*.log.gz" />
                    <!--!Note: 这里的age必须和filePattern协调, 后者是精确到HH, 这里就要写成xH, xd就不起作用
                    另外, 数字最好大于2, 否则可能造成删除的时候, 最近的文件还处于被占用状态,导致删除不成功!-->
                    <!--7天-->
                    <IfLastModified age="600s" />
                </Delete>
            </DefaultRolloverStrategy>
        </RollingFile>

    </appenders>

    <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
    <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>
        <!--若是additivity设为false，则 子Logger 只会在自己的appender里输出，而不会在 父Logger 的appender里输出。-->
        <Logger name="org.springframework" level="info" additivity="false">
            <AppenderRef ref="Console"/>
        </Logger>

        <root level="info">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
        </root>
    </loggers>

</configuration>

```

3. 参数解释
   - 

参考链接:

- [log4j2官方文档](https://logging.apache.org/log4j/2.x/manual/appenders.html#TriggeringPolicies)
- [Spring Boot 实战 —— 日志框架 Log4j2 SLF4J 的学习](https://michael728.github.io/2019/08/10/java-spring-boot-log4j2/ )
- [spring boot logging 默认配置](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml )

[pic1]: http://eddyzhou.gitee.io/picture-bed/202101/AeAzbU.png
[pic2]: https://raw.githubusercontent.com/EddyZhou97/picture-bed/master/images/202101/20210420193256.png