### 问题一，当前项目如果引用其他项目时，而其他项目需要redis配置获，但是当前项目没有reids配置时，pom.xml 使用exclusions排除依赖传递
```
<dependency>
    <groupId>com.fx</groupId>
    <artifactId>utils</artifactId>
    <version>1.0-SNAPSHOT</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
        </exclusion>
    </exclusions>
    <scope>compile</scope>
</dependency>
```

### 2,关键词说明
关键词 | 说明 | 备注
---|---|---
POM|项目对象模型(Project Object Model)的简称|[参考](https://www.cnblogs.com/wkrbky/p/6353285.html)
scope|主要管理依赖的部署|[参考](https://blog.csdn.net/cd18333612683/article/details/66478332)


