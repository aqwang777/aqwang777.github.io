---
layout: post
title:  "SpringCloud学习"
categories: JAVA
tags:  JAVA
author: aqwang
---

* content
{:toc}
## SpringCloud



### 什么是微服务架构

- 微服务是一种架构模式,它提倡将单一应用程序划分成一组小的服务,服务之间互相协和,互相配合,为用户提供最终价值.每个服务运行在每个独立的进程中,服务与服务之间采用轻量级的通信机制互相沟通(通常是基于HTTP的RestfulAPI).每个服务围绕具体业务进行构建.

### 什么是负载均衡

比如有很多服务器作为服务提供方提供同一个方法，服务调用方每次会调用一个服务器，调用这个服务器的这个方法。这个过程是随机的，负载均衡就是根据它的负载均衡策略把调用量均衡在所有服务提供方中，使得每个服务提供方被调用量保持一致。


### 负载均衡Ribbon

可以使用代码配置和属性配置的方式为某一个微服务提供负载均衡策略.

- 创建ribbon的配置类

```java
/**
 * @author aqwang
 * @date 2020/11/4 13:35
 */
@Configuration
public class LoadBalanced {

    @Bean
    public IRule ribbonRule(){
        //轮询
        return new RoundRobinRule();
        // return new WeightedResponseTimeRule();    //加权权重
        //return new RetryRule();                    //带有重试机制的轮训
        //return new RandomRule();                   //随机
        //return new TestRule();                     //自定义规则
    }
}
```

**注意：该类不能放在主应用程序上下文@ComponentScan所扫描的包中，否则配置将会被所有Ribbon Client共享。**

```java

@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "provider-demo",configuration = com.travelsky.config.LoadBalanced.class)
public class SpringCloudConsumerApplication {

	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudConsumerApplication.class, args);
	}

}
```

| 策略类                    | 命名               | 描述                                                         |
| ------------------------- | ------------------ | ------------------------------------------------------------ |
| RandomRule                | 随机策略           | 随机选择server                                               |
| RoundRobinRule            | 轮询策略           | 轮询选择， 轮询index，选择index对应位置的Server；            |
| RetryRule                 | 重试策略           | 对选定的负载均衡策略机上重试机制，在一个配置时间段内当选择Server不成功，则一直尝试使用subRule的方式选择一个可用的server； |
| BestAvailableRule         | 最低并发策略       | 逐个考察server，如果server断路器打开，则忽略，再选择其中并发链接最低的server |
| AvailabilityFilteringRule | 可用过滤策略       | 过滤掉一直失败并被标记为circuit tripped的server，过滤掉那些高并发链接的server（active connections超过配置的阈值）或者使用一个AvailabilityPredicate来包含过滤server的逻辑，其实就就是检查status里记录的各个Server的运行状态； |
| ResponseTimeWeightedRule  | 响应时间加权重策略 | 根据server的响应时间分配权重，响应时间越长，权重越低，被选择到的概率也就越低。响应时间越短，权重越高，被选中的概率越高，这个策略很贴切，综合了各种因素，比如：网络，磁盘，io等，都直接影响响应时间 |
| ZoneAvoidanceRule         | 区域权重策略       | 综合判断server所在区域的性能，和server的可用性，轮询选择server并且判断一个AWS Zone的运行性能是否可用，剔除不可用的Zone中的所有server |

### 负载均衡Feign

- Feign 是在 Ribbon 的基础上进行了一次改进，采用接口的方式，将需要调用的其他服务的方法定义成抽象方法即可，不需要自己构建 http 请求。不过要注意的是抽象方法的注解、方法签名要和提供服务的方法完全一致。
- 首先加入依赖开启feign：

```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <!--注意此处的依赖是SpringBoot2.0以后专用的，如果您使用的SpringBoot版本低于2.0请使用spring-cloud-starter-feign-->
      <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
```

- 在Springboot启动类上加入注解@EnableFeignClients。

- 创建feign最关键的接口。加上注解@FeignClient，里面的值是服务提供者的Spring-application-name。多个服务提供者的名字若是一样的provider-demo，那么就会使用feign的默认配置进行负载均衡，

  ```java
  /**
   * @author aqwang
   * @date 2020/11/4 15:01
   */
  @FeignClient("provider-demo")
  public interface UserFeignClient {
  
      @GetMapping(value = "/user/getUser/{id}")
      public User getUser(@PathVariable("id") Long id);
  
      @GetMapping(value = "/user/getName")
      public String getName();
  }
  ```

- controller中，就可以不使用restTemplate的调用方式了，直接调用userClient中的方式。

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserFeignClient userFeignClient;

    @GetMapping("/getUser/{id}")
    public User getUser(@PathVariable Long id){
        return userFeignClient.getUser(id);
    }

    @GetMapping("/getName")
    public String getName(){
        return userFeignClient.getName();
    }
}
```

### 什么是Hystrix

Hystrix是熔断机制，微服务架构中多层服务之间相互调用，如果有一层服务故障了，可能导致一层或多层服务故障，从而导致整个系统故障，这种现象称为雪崩效应。可以使用Hystrix进行保护，保护的方法是使用FallBack，调用的服务出现故障时，就可以使用 Fallback 方法的返回值；Hystrix 间隔时间会再次检查故障的服务，如果故障服务恢复，将继续使用服务。

### Hystrix的使用

- 引来Hystrix的依赖：
```
   		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

```

- SpringApplication上注解加入@EnableCircuitBreake注解，开启断路器模式。

- 在Controller中需要Hystrix的方法上加入@HystrixCommand(fallbackMethod = "getUserFallback")，然后添加getUserFallbcak方法，注意，入参和返回都要和之前的方法一样。

#### Feign如何使用Hystrix

  引入依赖和在SpringApplication上加入注解一样，不一样的是在yml配置中开启Hystrix，feign.hystrix.enabled: true。

创建一个接口

```
@FeignClient(name = "provider-demo", fallbackFactory = HystrixClientFactory.class)
public interface UserFeignClient {

    @GetMapping(value = "/user/getUser/{id}")
    public User getUser(@PathVariable("id") Long id);

}
```

方法和路径还有参数和Controller当中保持一致，在接口上方加入地址@FeignClient,name属性为服务提供方名字。

创建一个Factory

```
@Component
public class HystrixClientFactory implements FallbackFactory<UserFeignClient> {

    private static final Logger LOGGER = LoggerFactory.getLogger(HystrixClientFactory.class);

    @Override
    public UserFeignClient create(Throwable throwable) {
        HystrixClientFactory.LOGGER.info("the provider error is:{}", throwable.getMessage());
        return new UserFeignClient() {
            @Override
            public User getUser(Long id) {
                User user = new User();
                user.setName("阿良");
                return user;
            }
        };
    }
}
```

在这个create的工厂方法中，入参就是服务提供者的异常，在Controller中的方法出错过后得到这个异常则会去做实现。

### Zuul网关

#### Zuul的作用

统一入口：为其他微服务提供一个唯一的入口，网关起到外部和内部隔离的作用，保障了后台服务的安全性。

鉴别权限：识别每一个请求的权限，拒绝不符合要求的请求。

动态路由：动态的将请求路由到不同的后端集群中。

减少客户端和服务端的耦合：加入了网关，就不是客户端与服务进行直接的交互，服务可以独立发展，通过网关来做映射。

#### Zuul的应用

首先创建一个SpringBoot项目，加入

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

这两个依赖，然后在SpringBoot启动类上加入注解@SpringBootApplication和@EnableZuulProxy，然后在YML中进行eureka注册，然后配置网关属性，

```
zuul:
  routes:
    eureka-application-service:
      path: /**
      serviceId: provider-demo
```

serviceId则是服务提供方的Spring.application.name。这样即可进行路由转发。

##### 路由前缀配置

配置请求路径前缀，所有基于此前缀的请求都由zuul网关提供代理。

```
zuul.prefix=/api
```

　**zuul网关其底层使用ribbon来实现请求的路由，并内置Hystrix，可选择性提供网关fallback逻辑。使用zuul的时候，并不推荐使用Feign作为application client端的开发实现。**毕竟**Feign技术是对ribbon的再封装，使用Feign本身会提高通讯消耗，降低通讯效率**，只在服务相互调用的时候使用Feign来简化代码开发就够了。而且**商业开发中，使用Ribbon+RestTemplate来开发的比例更高**。





#### Zuul网关过滤器

Zuul中提供了过滤器定义，可以用来过滤代理请求，提供额外功能逻辑。如：权限验证，日志记录等。

Zuul提供的过滤器是一个父类。父类是ZuulFilter。通过父类中定义的抽象方法filterType，来决定当前的Filter种类是什么。有前置过滤、路由后过滤、后置过滤、异常过滤。

- **前置过滤**：是请求进入Zuul之后，立刻执行的过滤逻辑。
- **路由后过滤**：是请求进入Zuul之后，并Zuul实现了请求路由后执行的过滤逻辑，路由后过滤，是在远程服务调用之前过滤的逻辑。
- **后置过滤**：远程服务调用结束后执行的过滤逻辑。
- **异常过滤**：是任意一个过滤器发生异常或远程服务调用无结果反馈的时候执行的过滤逻辑。无结果反馈，就是远程服务调用超时。

#####　过滤器实现方式

继承父类ZuulFilter。在父类中提供了4个抽象方法，分别是：**filterType, filterOrder, shouldFilter, run**。其功能分别是：

- filterType：方法返回字符串数据，代表当前过滤器的类型。可选值有-pre, route, post, error。

  pre - 前置过滤器，**在请求被路由前执行**，通常用于处理身份认证，日志记录等；

  route - 在**路由执行后，服务调用前**被调用；

  error - **任意一个filter发生异常**的时候执行或**远程服务调用没有反馈的时候执行（超时）**，通常用于处理异常；

  post - 在**route或error执行后被调用**，一般用于收集服务信息，统计服务性能指标等，也可以对response结果做特殊处理。

- filterOrder：返回int数据，用于为**同filterType的多个过滤器定制执行顺序，返回值越小，执行顺序越优先**。

- shouldFilter：返回boolean数据，**代表当前filter是否生效**。

- run：具体的**过滤执行逻辑**。如pre类型的过滤器，可以通过对请求的验证来决定是否将请求路由到服务上；如post类型的过滤器，可以对服务响应结果做加工处理（如为每个响应增加footer数据）。

  ```java
  @Component
  public class LoggerFilter extends ZuulFilter {
  
      private static final Logger logger = LoggerFactory.getLogger(LoggerFilter.class);
      
      /**
       * 返回boolean类型。代表当前filter是否生效。
       * 默认值为false。
       * 返回true代表开启filter。
       */
      @Override
      public boolean shouldFilter() {
          return true;
      }
  
      /**
       * run方法就是过滤器的具体逻辑。
       * return 可以返回任意的对象，当前实现忽略。（spring-cloud-zuul官方解释）
       * 直接返回null即可。
       */
      @Override
      public Object run() throws ZuulException {
          // 通过zuul，获取请求上下文
          RequestContext rc = RequestContext.getCurrentContext();
          HttpServletRequest request = rc.getRequest();
  
          logger.info("LogFilter1.....method={},url={}",
                  request.getMethod(),request.getRequestURL().toString());
          // 可以记录日志、鉴权，给维护人员记录提供定位协助、统计性能
          return null;
      }
  
      /**
       * 过滤器的类型。可选值有：
       * pre - 前置过滤
       * route - 路由后过滤
       * error - 异常过滤
       * post - 远程服务调用后过滤
       */
      @Override
      public String filterType() {
          return "pre";
      }
  
      /**
       * 同种类的过滤器的执行顺序。
       * 按照返回值的自然升序执行。
       */
      @Override
      public int filterOrder() {
          return 0;
      }
  }
  ```





#### Zuul网关的容错(与豪猪Hystrix的无缝结合)

在SpringCloud中,Zuul启动器包含了Hystrix的相关依赖,zuul和Hystrix是无缝链接的.

在Edgware版本之前,Zuul提供了接口ZuulFallbackProvider用于实现fallback处理,Edgware版本开始,Zuul提供了ZuulFallbackProvider的子接口FallbackProvider来提供fallback处理.Zuul的fallback容错处理逻辑，只针对timeout异常处理，当请求被Zuul路由后，**只要服务有返回（包括异常），都不会触发Zuul的fallback容错逻辑。**　

因为对于Zuul网关来说，做请求路由分发的时候，结果由远程服务运算的。那么远程服务反馈了异常信息，Zuul网关不会处理异常，因为无法确定这个错误是否是应用真实想要反馈给客户端的。

```java
/**
 * 如果需要在Zuul网关服务中增加容错处理fallback，需要实现接口ZuulFallbackProvider
 * spring-cloud框架，在Edgware版本(包括)之后，声明接口ZuulFallbackProvider过期失效，
 * 提供了新的ZuulFallbackProvider的子接口 - FallbackProvider
 * 在老版本中提供的ZuulFallbackProvider中，定义了两个方法。
 *  - String getRoute()
 *    当前的fallback容错处理逻辑处理的是哪一个服务。可以使用通配符‘*’代表为全部的服务提供容错处理。
 *    如果只为某一个服务提供容错，返回对应服务的spring.application.name值。
 *  - ClientHttpResponse fallbackResponse()
 *    当服务发生错误的时候，如何容错。
 * 新版本中提供的FallbackProvider提供了新的方法。
 *  - ClientHttpResponse fallbackResponse(Throwable cause)
 *    如果使用新版本中定义的接口来做容错处理，容错处理逻辑，只运行子接口中定义的新方法。也就是有参方法。
 *    是为远程服务发生异常的时候，通过异常的类型来运行不同的容错逻辑。
 */
@Component
public class TestFallBbackProvider implements FallbackProvider {

    /**
     * return - 返回fallback处理哪一个服务。返回的是服务的名称
     * 推荐 - 为指定的服务定义特性化的fallback逻辑。
     * 推荐 - 提供一个处理所有服务的fallback逻辑。
     * 好处 - 服务某个服务发生超时，那么指定的fallback逻辑执行。如果有新服务上线，未提供fallback逻辑，有一个通用的。
     */
    @Override
    public String getRoute() {
        return "eureka-application-service";
    }

    /**
     * fallback逻辑。在早期版本中使用。
     * Edgware版本之后，ZuulFallbackProvider接口过期，提供了新的子接口FallbackProvider
     * 子接口中提供了方法ClientHttpResponse fallbackResponse(Throwable cause)。
     * 优先调用子接口新定义的fallback处理逻辑。
     */
    @Override
    public ClientHttpResponse fallbackResponse() {
        System.out.println("ClientHttpResponse fallbackResponse()");
        
        List<Map<String, Object>> result = new ArrayList<>();
        Map<String, Object> data = new HashMap<>();
        data.put("message", "服务正忙，请稍后重试");
        result.add(data);
        
        ObjectMapper mapper = new ObjectMapper();
        
        String msg = "";
        try {
            msg = mapper.writeValueAsString(result);
        } catch (JsonProcessingException e) {
            msg = "";
        }
        
        return this.executeFallback(HttpStatus.OK, msg, 
                "application", "json", "utf-8");
    }

    /**
     * fallback逻辑。优先调用。可以根据异常类型动态决定处理方式。
     */
    @Override
    public ClientHttpResponse fallbackResponse(Throwable cause) {
        System.out.println("ClientHttpResponse fallbackResponse(Throwable cause)");
        if(cause instanceof NullPointerException){
            
            List<Map<String, Object>> result = new ArrayList<>();
            Map<String, Object> data = new HashMap<>();
            data.put("message", "网关超时，请稍后重试");
            result.add(data);
            
            ObjectMapper mapper = new ObjectMapper();
            
            String msg = "";
            try {
                msg = mapper.writeValueAsString(result);
            } catch (JsonProcessingException e) {
                msg = "";
            }
            
            return this.executeFallback(HttpStatus.GATEWAY_TIMEOUT, 
                    msg, "application", "json", "utf-8");
        }else{
            return this.fallbackResponse();
        }
    }
    
    /**
     * 具体处理过程。
     * @param status 容错处理后的返回状态，如200正常GET请求结果，201正常POST请求结果，404资源找不到错误等。
     *  使用spring提供的枚举类型对象实现。HttpStatus
     * @param contentMsg 自定义的响应内容。就是反馈给客户端的数据。
     * @param mediaType 响应类型，是响应的主类型， 如： application、text、media。
     * @param subMediaType 响应类型，是响应的子类型， 如： json、stream、html、plain、jpeg、png等。
     * @param charsetName 响应结果的字符集。这里只传递字符集名称，如： utf-8、gbk、big5等。
     * @return ClientHttpResponse 就是响应的具体内容。
     *  相当于一个HttpServletResponse。
     */
    private final ClientHttpResponse executeFallback(final HttpStatus status, 
            String contentMsg, String mediaType, String subMediaType, String charsetName) {
        return new ClientHttpResponse() {

            /**
             * 设置响应的头信息
             */
            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders header = new HttpHeaders();
                MediaType mt = new MediaType(mediaType, subMediaType, Charset.forName(charsetName));
                header.setContentType(mt);
                return header;
            }

            /**
             * 设置响应体
             * zuul会将本方法返回的输入流数据读取，并通过HttpServletResponse的输出流输出到客户端。
             */
            @Override
            public InputStream getBody() throws IOException {
                String content = contentMsg;
                return new ByteArrayInputStream(content.getBytes());
            }

            /**
             * ClientHttpResponse的fallback的状态码 返回String
             */
            @Override
            public String getStatusText() throws IOException {
                return this.getStatusCode().getReasonPhrase();
            }

            /**
             * ClientHttpResponse的fallback的状态码 返回HttpStatus
             */
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return status;
            }

            /**
             * ClientHttpResponse的fallback的状态码 返回int
             */
            @Override
            public int getRawStatusCode() throws IOException {
                return this.getStatusCode().value();
            }

            /**
             * 回收资源方法
             * 用于回收当前fallback逻辑开启的资源对象的。
             * 不要关闭getBody方法返回的那个输入流对象。
             */
            @Override
            public void close() {
            }
        };
    }
}
```



#### Zuul网关的限流保护

Zuul网关组件也提供了限流保护。当请求并发达到阀值，自动触发限流保护，返回错误结果。只要提供error错误处理机制即可。

加入依赖

```
<dependency>
    <groupId>com.marcosbarbero.cloud</groupId>
    <artifactId>spring-cloud-zuul-ratelimit</artifactId>
    <version>1.3.4.RELEASE</version>
</dependency>
```

##### 全局限流

在yml上开启限流保护：

```java
# 开启限流保护
zuul.ratelimit.enabled=true
# 60s内请求超过3次，服务端就抛出异常，60s后可以恢复正常请求
zuul.ratelimit.default-policy.limit=3
zuul.ratelimit.default-policy.refresh-interval=60
# 针对IP进行限流，不影响其他IP
zuul.ratelimit.default-policy.type=origin
```

##### 局部限流

```java
# 开启限流保护
zuul.ratelimit.enabled=true
# hystrix-application-client服务60s内请求超过3次，服务抛出异常。
zuul.ratelimit.policies.hystrix-application-client.limit=3
zuul.ratelimit.policies.hystrix-application-client.refresh-interval=60
# 针对IP限流。
zuul.ratelimit.policies.hystrix-application-client.type=origin
```





