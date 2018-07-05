### 1，项目打包
### 2，文件名称修改
### 3，jar包文件上传linux服务器（使用WinSCP-SFTP客户端）
### 4，使用PuTTY或者Xshell 连接linux服务器
### 5，执行命令
```
1. nohup java -jar distributedjob-loanfinance.jar &   -后台运行 (nohup 后台执行，&命令)
2. ps -ef | grep java                                 -查看进程
3. kill -9 java                                       -停止进程
4. netstat -ntulp |grep 80                            -查看端口占用情况
5. vi nohup.out 显示文件内容                           -查看文件内容
6. ifconfig -a                                        -查看内网ip
```
### 6，配置xxl-job
#### 1，pom.xmljar依赖引用（包括logstash入kafka）
```
 <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <jetty-server.version>9.4.8.v20171121</jetty-server.version>
        <!--MAVEN 打包 跳过测试打包-->
        <maven.test.skip>true</maven.test.skip>
    </properties>
    <dependencyManagement>
        <dependencies>
            <!-- jetty -->
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-server</artifactId>
                <version>${jetty-server.version}</version>
            </dependency>
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-util</artifactId>
                <version>${jetty-server.version}</version>
            </dependency>
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-http</artifactId>
                <version>${jetty-server.version}</version>
            </dependency>
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-io</artifactId>
                <version>${jetty-server.version}</version>
            </dependency>

            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.xuxueli/xxl-job-core -->
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>1.9.1</version>
        </dependency>

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
    </dependencies>
```
#### 2，application.properties 配置
```
xxl.job.admin.addresses=http://172.20.15.88:8282/xxl-job-admin
xxl.job.executor.ip=192.168.15.114
### xxl-job executor address
xxl.job.executor.appname=wxappzwmc-loanfinance
xxl.job.executor.port=3003
### xxl-job, access token
xxl.job.accessToken=
### xxl-job log path
xxl.job.executor.logpath=../data/applogs/xxl-job/jobhandler
### xxl-job log retention days
xxl.job.executor.logretentiondays=-1
```
#### 3，XxlJobConfig 配置文件
```
import com.xxl.job.core.executor.XxlJobExecutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author xuxueli 2017-04-28
 */
@Configuration
@ComponentScan(basePackages = "com.distributedjob1.jobs")
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.executor.appname}")
    private String appName;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;


    @Bean(initMethod = "start", destroyMethod = "destroy")
    public XxlJobExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobExecutor xxlJobExecutor = new XxlJobExecutor();
        xxlJobExecutor.setAdminAddresses(adminAddresses);
        xxlJobExecutor.setAppName(appName);
        xxlJobExecutor.setIp(ip);
        xxlJobExecutor.setPort(port);
        xxlJobExecutor.setAccessToken(accessToken);
        xxlJobExecutor.setLogPath(logPath);
        xxlJobExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobExecutor;
    }

}
```
#### 4，任务java文件
```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.handler.IJobHandler;
import com.xxl.job.core.handler.annotation.JobHandler;
import com.xxl.job.core.log.XxlJobLogger;
import org.jinq.orm.stream.JinqStream;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author: caolh
 * @description: DownPaymentToEsTask
 * @date: 2018/6/29 14:13
 */
@JobHandler(value = "loanfinancetoesJob")
@Component
public class DownPaymentToEsTask extends IJobHandler {
    @Override
    public ReturnT<String> execute(String param) throws Exception {
        XxlJobLogger.log(String.format("resultList=%s,sccessCount=%s", resultList.size(), sccessCount.toString()));
        return SUCCESS;
    }
}
```