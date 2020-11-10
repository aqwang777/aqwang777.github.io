---
layout: post
title:  "SpringCloud"
categories: SpringCloud
tags:  SpringCloud
author: aqwang
---

* content
{:toc}


###　什么是微服务架构

- 微服务是一种架构模式,它提倡将单一应用程序划分成一组小的服务,服务之间互相协和,互相配合,为用户提供最终价值.每个服务运行在每个独立的进程中,服务与服务之间采用轻量级的通信机制互相沟通(通常是基于HTTP的RestfulAPI).每个服务围绕具体业务进行构建.


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

###  Hystrix

