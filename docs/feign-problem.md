# Spring Cloud项目中通过Feign进行内部服务调用发生401\407错误无返回信息的问题

## 问题描述
最近在使用Spring Cloud改造现有服务的工作中，在内部服务的调用方式上选择了Feign组件，由于服务与服务之间有权限控制，发现通过Feign来进行调用时如果发生了401、407错误时，调用方不能够取回被调用方返回的错误信息。

## 产生原因
Feign默认使用java.net.HttpURLConnection进行通信，通过查看其子类sun.net.www.protocol.http.HttpURLConnection源码发现代码中在进行通信时单独对错误码为401\407的错误请求做了处理，当请求的错误码为401\407时，会关闭请求流，由于此时还并没有将返回的错误信息写入响应流中，所以接收的返回信息中仅仅能获取到response.status()，而response.body()为null。
HttpURLConnection相关信息的源码链接：http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/6-b14/sun/net/www/protocol/http/HttpURLConnection.java#1079

## 解决思路
关于此问题产生的原因已经很明显了，就是feign.Client实现通信的方式选用了我们不想使用的HttpURLConnection。想到通常在Spring的代码中OCP都是运用得很好的，所以基本上有解决此问题的信心了，最不济就是自己扩展Feign，实现一个自己想要的feign.Client，当然这种事情Spring Cloud基本都会自己搞定，这也是Spring Cloud强大完善的一个地方。
通过这个思路查看源码，果然看到了Spring Cloud在使用Feign提前内置了三种通信方式（feign.Client.Default，feign.httpclient.ApacheHttpClient，feign.okhttp.OkHttpClient），其中缺省的情况使用的就是feign.Client.Default，这个就是使用HttpURLConnection通信的方式。

## 源码解析
在Spring Cloud项目中使用了Ribbon的组件，其会帮助我们管理使用Feign，查看org.springframework.cloud.netflix.feign.ribbon.FeignRibbonClientAutoConfiguration源码
```
@ConditionalOnClass({ ILoadBalancer.class, Feign.class })
@Configuration
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FeignRibbonClientAutoConfiguration {

        @Bean
    @ConditionalOnMissingBean
    public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
            SpringClientFactory clientFactory) {
        return new LoadBalancerFeignClient(new Client.Default(null, null),
                cachingFactory, clientFactory);
    }

        @Configuration
    @ConditionalOnClass(ApacheHttpClient.class)
    @ConditionalOnProperty(value = "feign.httpclient.enabled", matchIfMissing = true)
    protected static class HttpClientFeignLoadBalancedConfiguration {

        @Autowired(required = false)
        private HttpClient httpClient;

        @Bean
        @ConditionalOnMissingBean(Client.class)
        public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
                SpringClientFactory clientFactory) {
            ApacheHttpClient delegate;
            if (this.httpClient != null) {
                delegate = new ApacheHttpClient(this.httpClient);
            }
            else {
                delegate = new ApacheHttpClient();
            }
            return new LoadBalancerFeignClient(delegate, cachingFactory, clientFactory);
        }
    }

    @Configuration
    @ConditionalOnClass(OkHttpClient.class)
    @ConditionalOnProperty(value = "feign.okhttp.enabled", matchIfMissing = true)
    protected static class OkHttpFeignLoadBalancedConfiguration {

        @Autowired(required = false)
        private okhttp3.OkHttpClient okHttpClient;

        @Bean
        @ConditionalOnMissingBean(Client.class)
        public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
                SpringClientFactory clientFactory) {
            OkHttpClient delegate;
            if (this.okHttpClient != null) {
                delegate = new OkHttpClient(this.okHttpClient);
            }
            else {
                delegate = new OkHttpClient();
            }
            return new LoadBalancerFeignClient(delegate, cachingFactory, clientFactory);
        }
    }
}
```

* 从feignClient(CachingSpringLoadBalancerFactory cachingFactory, SpringClientFactory clientFactory) 方法结合其上注解我们可以很清楚的知道，当没有feign.ClientBean的时候会默认生成feign.Client.Default来进行通信，这就是之前说的缺省通信方式
* 从HttpClientFeignLoadBalancedConfiguration、OkHttpFeignLoadBalancedConfiguration，我们可以看到其生效的条件，当classpath中有feign.httpclient.ApacheHttpClient并且配置feign.httpclient.enabled=true（缺省为true）、feign.okhttp.OkHttpClient并且配置feign.okhttp.enabled=true（缺省为true）
* 当使用ApacheHttpClient或者OkHttpClient进行通信时就不会导致发生401\407错误时，取不到返回的错误信息了

## 解决方法
通过其上的分析，解决方法已经显而易见了

1、pom.xml文件中新增
```
<dependency>
         <groupId>com.netflix.feign</groupId>
         <artifactId>feign-okhttp</artifactId>
         <version>8.18.0</version>
   </dependency>
```

或者
```
<dependency>
         <groupId>com.netflix.feign</groupId>
         <artifactId>feign-httpclient</artifactId>
         <version>8.18.0</version>
   </dependency>
```

2、application.properties文件中增加（可缺省，如有切换调用方式需求可添加配置进行控制）

feign.okhttp.enabled=true

或者

feign.httpclient.enabled=true