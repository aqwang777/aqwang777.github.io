---
layout: post
title:  "深入学习SpringBoot"
categories: JAVA
tags:  JAVA
author: aqwang
---

* content
{:toc}
## Springboot

### 组合注解@SpringBootApplication

@SpringBootApplication内包含很多个注解，其中最重要的是@SpringBootConfiguration，@EnableAutoConfiguration，@ComponentScan三个注解。

#### @ComponentScan

@ComponentScan注解的功能是自动扫描并加载符合条件的组件（比如@Component和@Repository）或者bean定义，最终将这些bean定义加载到IOC容器中。

我们可以通过指定basePackages等属性来定制@ComponentScan自动扫描的范围，一般来说我们都是没有指定的，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

#### @SpringBootConfiguration

@SpringBootConfiguration继承自@Configuration，标注当前类为配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入Spring容器中。

#### @EnableAutoConfiguration

这个注解是Springboot的核心功能，自动配置。这个注释告诉SpringBoot“猜”你将如何想配置Spring，基于你已经添加jar依赖项。 	



### SpringBoot,SpringMVC,Spring之间的区别

Spring是一个开源的应用程序框架，提供了一个简易的开发方式，通过这种开发方式，将避免那些可能致使代码变得繁杂混乱的大量的业务/工具对象，说的更通俗一点就是由框架来帮你管理这些对象，包括它的创建，销毁等。

SpringMVC是Spring的一部分，Spring出来过后，大家觉得很好用，所以按照这种模式设计了一个MVC框架，主要用于开发WEB应用和网络接口，它是Spring的一个模块，通过Dispatcher Servlet,ModelAndView和View Resolver,让应用开发变得更简单。

Spring和SpringMVC的问题是要配置大量的参数和配置进行开发，太多XML配置文件一直是Spring被人诟病的地方，SpringBoot的目的在于实现自动配置，降低项目搭建的复杂度。



### 开启SpringBoot特性有哪几种方式

- 继承spring-boot-starter-parent项目

```
<parent>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-parent</artifactId>
   <version>1.5.6.RELEASE</version>
</parent>
```

- 导入spring-boot-dependencies依赖

```
<dependencyManagement>
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-dependencies</artifactId>
           <version>1.5.6.RELEASE</version>
           <type>pom</type>
           <scope>import</scope>
       </dependency>
</dependencyManagement>
```



### SpringBoot需要独立的容器运行吗？

不需要，SpringBoot内置了Tomcat容器。



### SpringBoot自动配置原理是什么

注解@EnableAutoConfiguration,@Configuration,@ConfitionalOnClass就是自动配置的核心，首先它得是一个配置文件，其次根据类路径下是否有这个类去自动配置。



### 你如何理解Spring Boot中的Starters？

Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，有了Starters，我可以一站集成Spring及其他技术，不用到处找Jar包依赖。



### SpringBoot有哪几种读取配置的方式？

SpringBoot可以通过@PropertySource,@Value,@Environment,@ConfigurationProperties来绑定变量。



### SpringBoot支持哪些日志框架？推荐和默认的日志框架是啥？

SpringBoot支持Logging,Logback,Log4j2,如果使用starers启动器，SpringBoot将使用Logback作为默认日志框架。



### 怎么将SpringBoot Web应用程序部署为jar或war文件？

在pom.xml中加入这个插件即可：

```
 <plugin>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-maven-plugin</artifactId>
 </plugin>
```

加入这个插件后，我们执行mvn package步骤后得到一个jar包。可以直接使用java -jar进行本地部署，如果需要在服务器上进行部署，可以编写脚本来运行或者使用后台java -jar命令：nohup java -jar xxx.jar &

注意：为了打包成JAR文件，Pom.xml中packing属性必须指定为JAR



### SpringBoot的Actuator是做什么的？

Actuator通过启用production-ready使得Springboot应用程序变得更有生命力。这些功能允许我们对生产环境中的应用程序进行监视和管理。

引入依赖即可把Actuator到项目中。

```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



### 什么是AOP

AOP就是可以将一些系统性相关的编程工作，独立提取出来，独立实现，然后通过切面进入系统，从而避免在业务逻辑的代码中混入很多系统相关的逻辑。

举个例子，我想给我的网站加上鉴权，对某些url，你认为不需要鉴权就可以访问，对其他url，你认为需要有特定权限才可以访问。如果我还用OOP，那么就要在controller当中一个一个写上鉴权代码，如果使用AOP，则可以对原有代码毫无侵略性，没有在业务逻辑的代码中混入系统相关的逻辑。



### 什么是CSRF攻击

CSRF攻击代表跨站请求伪造，它迫使用户在当前通过身份验证的WEB上执行不必要的操作。CSRF专门针对状态改变请求，而不是窃取数据，因为它无法看到伪造请求的响应。



