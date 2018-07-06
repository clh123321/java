### 1，Java High Level REST Client 方式请求,受es版本影响较小  [Elasticsearch 内置的两个客户端的区别](https://blog.csdn.net/sidongxue2/article/details/73835695)  [Java 和 Elasticsearch 交互](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_talking_to_elasticsearch.html)
#### 1，pom.xml 依赖引用
```
        <!--es-->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>5.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>
            <version>5.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>5.6.0</version>
        </dependency>
        <!--es 依赖的jar-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
        </dependency>
```
#### 2，配置java文件
```
import org.apache.http.HttpHost;

import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.ArrayList;
import java.util.List;

/**
 * @author: caolh
 * @description: EsConfiguration
 * @date: 2018/7/2 11:28
 */
@Configuration
public class ElasticsearchConfiguration {

    private static final String http = "http";
    @Value("${spring.data.elasticsearch.cluster-nodes}")
    private String clusterNodes;

    @Bean
    public RestHighLevelClient buildClient() {
        if (clusterNodes != null && !clusterNodes.isEmpty()) {
            String[] nodesArray = clusterNodes.split(";");
            HttpHost[] httpHosts = new HttpHost[nodesArray.length];
            for (int i = 0; i < nodesArray.length; i++) {
                String inetSocket[] = nodesArray[i].split(":");
                String address = inetSocket[0];
                Integer port = Integer.valueOf(inetSocket[1]);
                httpHosts[i] = new HttpHost(address, port, http);
            }
            RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(httpHosts).build());
            return client;
        }
        return null;
    }
}
```
备注：
RestClient.builder(httpHosts).build() 5.0版本  
RestClient.builder(httpHosts) 6.0版本
