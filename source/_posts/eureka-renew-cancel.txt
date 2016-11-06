---
layout:     post
title:      Spring Cloud Eureka服务续约和服务下线源码分析
date:       2016-11-05 14:00:00 +0800
summary:    Spring Cloud Eureka服务续约和服务下线源码分析
toc: true
categories:
- Spring Cloud Eureka
tags:
- Spring Cloud Eureka
---
**摘要**:在上一篇中，主要分析了Eureka 服务注册的源码分析，在本篇文章中将对Eureka的Renew(服务续约)，cancel(服务下线)进行源码分析。
## 概述
 ### 名词解释
 1. Renew:我的理解是续约，为什么叫续约呢？
   Renew（服务续约）操作由Service Provider`定期调用`，类似于heartbeat。目的是隔一段时间Service Provider调用接口，告诉Eureka Server它还活着没挂，不要把它提了。通俗的说就是它们两之间的心跳检测，避免服务提供者被剔除掉。
 2. Cancel（服务下线）
   一般在Service Provider`挂了`或`shut down`的时候调用，用来把自身的服务从Eureka Server中`删除`，以防客户端调用到不存在的服务。
 3. Fetch Registries(获取注册信息)，
   Fetch Registries由Service Consumer(服务消费者)调用，用来获取Eureka Server上注册的服务info。
<!--more-->
 4. Eviction(剔除)
   Eviction（失效服务剔除）用来定期在Eureka Server检测失效的服务，检测标准就是超过一定时间没有Renew的服务。
## 回顾Eureka架构图
### Eureka架构图
   Eureka架构图如下图所示，github地址:https://github.com/netflix/eureka
   document地址:https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance
  ![Eureka架构图](/images/spring-cloud-netflix/eureka/eureka_architecture.png)
&ensp;　从图中我们可以看出，Eureka 组件分为两部分：`Eureka server`和 `Eureka client`。而客户端又分为 `Application Service 客户端`和 `Application Client 客户端`两种。Eureka 的工作机制每个 region 都有自己的 Eureka 服务器集群，每个 zone 至少要有一个 Eureka 服务器以应对 zone 瘫痪。 
&ensp;　Application Service 在启动时注册到 Eureka 服务器，之后每 `30` 秒钟发送心跳以更新自身状态,即`Renew(续约)`。如果该客户端没能发送心跳更新，它将在 `90` 秒之后被其注册的 Eureka 服务器剔除，即`Eviction(剔除)`。来自任意 zone 的 Application Client 可以获取这些注册信息(每隔 `30` 秒查看一次)并依此定位到在任何区域可以给自己提供服务的提供者(即Fetch Registries)，进而进行远程调用。
>服务提供者本身携带的Eureka Client既能`服务注册`，`服务续约`，也能通过client`定位服务`和`调用其它的服务`。

## Renew(服务续约)
### 服务续约
  Renew操作会在Service Provider端定期发起，用来通知Eureka Server自己还活着。 这里有两个比较重要的配置需要注意一下：
```java
  eureka.instance.leaseRenewalIntervalInSeconds
```
  Renew频率。默认是30秒，也就是每30秒会向Eureka Server发起Renew操作。
```java
  eureka.instance.leaseExpirationDurationInSeconds
```
 服务失效时间。默认是90秒，也就是如果Eureka Server在90秒内没有接收到来自Service Provider的Renew操作，就会把Service Provider剔除。
### Renew源码分析

