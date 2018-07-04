### 1.POM依赖管理
```
<dependencyManagement>
  <dependencies>
     <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```
### 2，POM依赖引用
```
<!-- logback的数据写入到kafka -->
		<dependency>
			<groupId>com.github.danielwegener</groupId>
			<artifactId>logback-kafka-appender</artifactId>
			<version>0.2.0-RC1</version>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>1.2.3</version>
		</dependency>
		<dependency>
			<groupId>net.logstash.logback</groupId>
			<artifactId>logstash-logback-encoder</artifactId>
			<version>5.0</version>
		</dependency>
```
### 3，resources 增加logback-spring.xml 文件(springboot推荐使用logback-spring.xml而不是logback.xml)  [参考](https://blog.csdn.net/qianyiyiding/article/details/76565810)(logback.xml加载早于application.properties，所以如果你在logback.xml使用了变量时，而恰好这个变量是写在application.properties时，那么就会获取不到，只要改成logback-spring.xml就可以解决)
### 4，增加IPLogConfig 配置类
```
package com.bitauto.ep.fx.distributedjob1.configuration;

import ch.qos.logback.classic.pattern.ClassicConverter;
import ch.qos.logback.classic.spi.ILoggingEvent;
import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * @desc 配置日志中显示IP.
 */
public class IPLogConfig extends ClassicConverter {

    @Override
    public String convert(ILoggingEvent event) {
        try {
            return InetAddress.getLocalHost().getHostAddress();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
### 4，[文件](https://github.com/clh123321/java/blob/master/logback-%E6%8E%A5kafka%E5%BC%82%E6%AD%A5%E6%8E%A5%E6%94%B6%E6%B6%88%E6%81%AF-logback-spring.xml)

### 5，其他方式
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
<springProfile name="staging, production">
    <include resource="logback-production.xml"/>
</springProfile>

<springProfile name="dev">
        <include resource="logback-dev.xml"/>
</springProfile>

</configuration>
```