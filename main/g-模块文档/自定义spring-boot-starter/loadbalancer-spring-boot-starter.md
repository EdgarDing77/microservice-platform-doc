# loadbalancer-spring-boot-starter

## Introduction

由于采用RestTemplate作为LB，因此需要独立一个`loadbalancer-spring-boot-starter`进行使用。

## RestTemplate作为LB

配置案例：

```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
    @Autowired
    private RestTemplate restTemplate;

    public String doOtherStuff() {
        String results = restTemplate.getForObject("http://stores/stores", String.class);
        return results;
    }
}
```

需要注意：

>  A `RestTemplate` bean is no longer created through auto-configuration. Individual applications must create it.

 To use a load-balanced `RestTemplate`, you need to have a load-balancer implementation in your classpath. Add [Spring Cloud LoadBalancer starter](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#spring-cloud-loadbalancer-starter) to your project in order to use it.

这也是独立出来一个loadbalancer通用逻辑封装的原因。

## 自定义实现HTTPclient

```java
@EnableConfigurationProperties(RestTemplateProperties.class)
public class RestTemplateAutoConfigure {
    @Resource
    private RestTemplateProperties restTemplateProperties;

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory httpRequestFactory) {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setRequestFactory(httpRequestFactory);
        return restTemplate;
    }

    /**
     * httpclient 实现的ClientHttpRequestFactory
     * 使用给定的HttpClient实例创建HttpComponentsClientHttpRequestFactory的新实例。
     */
    @Bean
    public ClientHttpRequestFactory httpRequestFactory(HttpClient httpClient) {
        return new HttpComponentsClientHttpRequestFactory(httpClient);
    }

    /**
     * 使用连接池的 httpclient
     */
    @Bean
    public HttpClient httpClient() {

        Registry<ConnectionSocketFactory> registry = RegistryBuilder.<ConnectionSocketFactory>create()
            .register("http", PlainConnectionSocketFactory.getSocketFactory())
            .register("https", SSLConnectionSocketFactory.getSocketFactory())
            .build();

        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager(registry);
        // 最大链接数
        connectionManager.setMaxTotal(restTemplateProperties.getMaxTotal());
        // 同路由并发数20
        connectionManager.setDefaultMaxPerRoute(restTemplateProperties.getMaxPerRoute());

        RequestConfig requestConfig = RequestConfig.custom()
            // 读超时
            .setSocketTimeout(restTemplateProperties.getReadTimeout())
            // 链接超时
            .setConnectTimeout(restTemplateProperties.getConnectTimeout())
            // 链接不够用的等待时间
            .setConnectionRequestTimeout(restTemplateProperties.getReadTimeout())
            .build();

        return HttpClientBuilder.create()
            .setDefaultRequestConfig(requestConfig)
            .setConnectionManager(connectionManager)
            .setRetryHandler(new DefaultHttpRequestRetryHandler(3, true))
            .build();
    }
}

```

## Reference

- https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#rest-template-loadbalancer-client