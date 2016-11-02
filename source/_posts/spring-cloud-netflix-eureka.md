---
layout:     post
title:      Spring Cloud Netflix之Eureka上篇
date:       2016-10-23 14:00:00 +0800
summary:   Eureka是Netflix开源的一款提供服务注册和发现的产品，本文主要介绍了Eureka的实现细节。
toc: true
categories:
- Spring Cloud Netflix
tags:
- Spring Cloud Netflix
- Eureka
---
前言:Spring Cloud NetFlix这个项目对NetFlix中一些久经考验靠谱的服务发现，熔断，网关，智能路由，以及负载均衡等做了封装，并通过注解的或简单配置的方式提供给Spring Cloud用户用。本文主要介绍 Spring Cloud中的Eureka组件。由于Spring Cloud做技术选型时中立的，因此Spring Cloud也提供了Spring Cloud Zookeeper,Spring Cloud Consul用于服务治理或服务发现供大家选择使用，另外我还发现[Spring Cloud etcd](https://github.com/SpringCloud/spring-cloud-etcd)这个项目，也可以用于服务注册和发现
<!--more-->
## 什么是 Spring Cloud Netflix ?
其官方文档中对自己的定义是如下，[官网连接](http://cloud.spring.io/spring-cloud-netflix/),[Github地址](https://github.com/spring-cloud/spring-cloud-netflix)
> This project provides Netflix OSS integrations for Spring Boot apps through autoconfiguration and binding to the Spring Environment and other Spring programming model idioms. With a few simple annotations you can quickly enable and configure the common patterns inside your application and build large distributed systems with battle-tested Netflix components. The patterns provided include Service Discovery (Eureka), Circuit Breaker (Hystrix), Intelligent Routing (Zuul) and Client Side Load Balancing (Ribbon).

Spring Cloud Netflix这个项目对于Spring Boot应用来说，它集成了NetFlix OSS的一些组件，只需通过注解配置和Spring环境的通用简单的使用注解，你可以快速的启用和配置这些久经测试考验的NetFlix的组件于你的应用和用于构建分布式系统中。这些组件包含的功能有服务发现（Eureka），熔断器（Hystrix），智能路由(Zuul)以及客户端的负载均衡器（Ribbon）

简单的来说，Spring Cloud NetFlix这个项目对NetFlix中一些久经考验靠谱的服务发现，熔断，网关，智能路由，以及负载均衡等做了封装，并通过注解的或简单配置的方式提供给Spring Cloud用户用。

## 什么是 Eureka?
官网定义是:
>Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers. We call this service, the Eureka Server. Eureka also comes with a Java-based client component,the Eureka Client, which makes interactions with the service much easier. The client also has a built-in load balancer that does basic round-robin load balancing.

简单来说Eureka就是Netflix开源的一款提供服务注册和发现的产品，并且提供了相应的Java客户端。

## 为什么要选择 Eureka?
那么为什么我们在项目中使用了Eureka呢？主要原因如下:
* **它提供了完整的Service Registry和Service Discovery实现**
	* 首先是提供了完整的实现，并且也经受住了Netflix的生产环境考验，使用比较方便只需通过注解或简单配置的方式即可。
* **和Spring Cloud无缝集成**
	* Spring Cloud对Eureka做了无缝集成，提供了一套完善的解决方案，所以使用起来非常方便。
	* 另外，Eureka支持嵌入到应用自身的容器中启动，应用启动完之后，既充当了Eureka的角色，同时也是服务的提供者。这样就极大的提高了服务的可用性。
* **开源**
	* 开源代码，方便学习掌握其源码并驾驭它。  

参考阅读：为什么不应该使用ZooKeeper做服务发现
英文链接:
Eureka! Why You Shouldn’t Use ZooKeeper for Service Discovery:
http://www.knewton.com/tech/blog/2014/12/eureka-shouldnt-use-zookeeper-service-discovery/
中文链接:
http://blog.csdn.net/jenny8080/article/details/52448403
Eureka vs. Zookeeper：
https://groups.google.com/forum/#%21topic/eureka_netflix/LXKWoD14RFY


## 进一步了解 Eureka

### Eureka基本架构图

![architecture-overview](/images/spring-cloud-netflix/eureka/architecture-overview.png)

上图简要描述了Eureka的基本架构，由3个角色组成：

1. **Eureka Server**
	* 提供服务注册和发现

2. **Service Provider**
	* 服务提供者，服务启动的时候会将自己的服务信息注册到Eureka

3. **Service Consumer**
	* 服务消费者，从Eureka中获取已注的服务信息，用于调用服务生产者

需要注意一点是：一个Service Provider既可以是Service Consumer，也可以是Service Provider。

### 集群模式下的Eureka

![architecture-detail](/images/spring-cloud-netflix/eureka/architecture-detail.png)

上图更进一步的展示了3个角色之间的交互。

1. Service Provider会向Eureka Server做Register（服务注册）、Renew（服务续约）、Cancel（服务下线）等操作。
2. Eureka Server之间会做注册服务的同步，从而保证状态一致
3. Service Consumer会向Eureka Server获取注册服务列表，并消费服务
