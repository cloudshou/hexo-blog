
title: Spring Cloud微服务框架主要子项目和RPC框架的对比
categories:
- Spring Cloud
tags:
- Spring Cloud
---
　**摘要**:Spring Cloud是一个相对比较新的微服务框架，今年(2016)推出1.0的release版本，目前Github上更新速度很快. 虽然Spring Cloud时间最短, 但是相比Dubbo等RPC框架, Spring Cloud提供的全套的分布式系统解决方案。spring cloud 为开发者提供了在分布式系统（配置管理，服务发现，熔断，路由，微代理，控制总线，一次性token，全居琐，leader选举，分布式session，集群状态）中快速构建的工具，使用Spring Cloud的开发者可以快速的启动服务或构建应用．它们将在任何分布式环境中工作，包括开发人员自己的笔记本电脑，裸物理机的数据中心，和像Cloud Foundry云管理平台。在未来引领这微服务架构的发展，提供业界标准的一套微服务架构解决方案。
<!--more-->

## 1.什么是Spring Cloud？
 　Spring Cloud是一个相对比较新的微服务框架，今年(2016)才推出1.0的release版本. 虽然Spring Cloud时间最短, 但是相比Dubbo等RPC框架, Spring Cloud提供的全套的分布式系统解决方案。spring cloud 为开发者提供了在分布式系统（配置管理，服务发现，熔断，路由，微代理，控制总线，一次性token，全居琐，leader选举，分布式session，集群状态）中快速构建的工具，使用Spring Cloud的开发者可以快速的启动服务或构建应用．它们将在任何分布式环境中工作，包括开发人员自己的笔记本电脑，裸物理机的数据中心，和像Cloud Foundry云管理平台。下面是官方对Spring Cloud定义和解释。
　{% blockquote %}
  　Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer’s own laptop, bare metal data centres, and managed platforms such as Cloud Foundry.
　{% endblockquote %}

## 2.Spring Cloud主要项目
  Spring Cloud 侧重于提供良好的开箱即用的功能，以便支持典型的开发场景和扩展支持。下面主要Spring Cloud项目在微服务框架中的主要子项目，具体的子项目源码分析，以及实现细节，将会在后面的文章中介绍。
- Spring Cloud Config---配置中心
   Spring Cloud Config就是我们通常意义上的配置中心 - 把应用原本放在本地文件的配置抽取出来放在中心服务器，从而能够提供更好的管理、发布。
   >在RPC服务治理框架中，一般都会开发一个配置中心和ZK配合使用，用于管理分布式应用中的配置信息。比如熔断的阀值，负载均衡的策略等。
- Spring Cloud Netflix--注册中心，服务发现，LB
   Spring Cloud Netflix通过Eureka Server实现服务注册中心(包括服务注册，服务发现)，通过Ribbon实现软负载均衡(load balance,简称LB)
   >在RPC框架中，例如：dubboX，HSF，OSP(唯品会的RPC框架)等RPC框架，都会通过ZK等实现服务注册，服务发现。当服务启动时，会将服务的IP地址，端口，服务命名，版本号等信心注册到ZK中，同时ZK会把服务注册信息，推送到服务的调用client端或Proxy端。
   >至于LB，都会有自己的实现算法，熔断等都有自己的实现方式。
   
未完待续-----------------