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
### 2，resources 增加logback-spring.xml 文件
### 3，[文件](https://github.com/clh123321/java/blob/master/logback-%E6%8E%A5kafka%E5%BC%82%E6%AD%A5%E6%8E%A5%E6%94%B6%E6%B6%88%E6%81%AF-spring.xml)