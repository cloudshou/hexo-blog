---
layout:     post
title:      Spring Cloud Sleuth-全链路监控调研
date:       2016-10-21 14:00:00 +0800
toc: true
categories:
- Spring Cloud Sleuth
tags:
- Spring Cloud Sleuth
- Spring Cloud
- 全链路监控
- 微服务
---
前言:做过软件开发的都知道，对`系统进行全链路的监控`是非常有必要的。在单体应用中，传统的方式是软件开发者，通过自定义日志的level，日志文件的方式记录单体应用的`运行日志`。从而排查线上系统出现运行过慢，出现故障，异常等问题，但是在微服务架构或分布式系统中，一个系统被拆分成了A、B、C、D、E等多个服务，而每个服务可能又有多个实例组成集群，采用上诉定位问题的方式就行不通了，你充其量就知道某个服务是应用的瓶颈，但中间发生了什么你完全不知道。而且问题的查询，因为有海量各种各样的日志等文件，导致`追溯定位问题`等极其不方便。因此需要`全链路监控系统的收集，上报，对海量日志实时计算生成，监控告警，视图报表，帮助开发人员快速定位问题`。

## 服务追踪分析
一个由微服务构成的应用系统由N个服务实例组成，通过`REST请求`或者`RPC协议`等来通讯完成一个业务流程的调用。对于入口的一个调用可能需要有多个后台服务协同完成，链路上`任何一个调用超时`或`出错`都可能造成前端请求的失败。服务的调用链也会越来越长，并形成一个树形的调用链。如下图所示:
![调用链](/images/spring-cloud-sleuth/1/dyl.png)
<!--more-->
但是随着服务的增多，对调用链的分析也会越来越负责。设想你在负责下面这个系统，其中每个小点都是一个微服务，他们之间的调用关系形成了复杂的网络。如下图所示:
![调用关系复杂网络图](/images/spring-cloud-sleuth/1/qzfw.png)

通过该图，可以看出错综复杂的调用网路图。针对服务化应用全链路追踪的问题，Google发表了Dapper论文，介绍了他们如何进行服务追踪分析。其基本思路是在服务调用的请求和响应中加入ID，标明上下游请求的关系。利用这些信息，可以可视化地分析服务调用链路和服务间的依赖关系。

## 什么是 Spring Cloud Sleuth ?

Spring Cloud Sleuth为Spring Cloud提供了分布式追踪方案，为了更好的理解这个领域中的一些概念，建议先自行搜索学习一下Google Dapper相关的论文，http://research.google.com/pubs/pub36356.html，github Code连接:<a href="https://github.com/spring-cloud/spring-cloud-sleuth" target="_blank">Spring Cloud Sleuth Code</a>。官方文档地址:http://cloud.spring.io/spring-cloud-sleuth/spring-cloud-sleuth.html. 其官方文档中对自己的定义是如下：

>Spring Cloud Sleuth implements a distributed tracing solution for Spring Cloud, borrowing heavily from Dapper, Zipkin and HTrace. For most users Sleuth should be invisible, and all your interactions with external systems should be instrumented automatically. You can capture data simply in logs, or by sending it to a remote collector service.

简单来说，Spring Cloud Sleuth就是APM(Application Performance Monitor),全链路监控的APM的一部分，如果要完整的使用该组件需要自己定制化或者和开源的系统集成，例如:ZipKin。

>APM（Application Performance Monitor）这个领域最近异常火热。国外该领域知名公司包括New Relic，Appdynamics，Splunk。其中New Relic已经成功IPO，估值超过20亿美元。
>国内外的个大互联网公司也都有类似大名鼎鼎的APM产品，例如淘宝鹰眼Eagle Eyes，点评的CAT，微博的Watchman，twitter的Zipkin。他们的产品虽未像专业APM公司的产品这样功能强大，但结合各自公司的业务特点，这些产品在支撑业务系统的高性能和稳定性方面，发挥了显著的作用。

## Spring Cloud Sleuth和Zipkin
对应Dpper的开源实现是Zipkin，支持多种语言包括JavaScript，Python，Java, Scala, Ruby, C#, Go等。其中Java由多种不同的库来支持。

## SpringCloudSleuth 借用了 Dapper 的术语

* **Span**
	* 基本工作单元，例如，在一个新建的span中发送一个RPC等同于发送一个回应请求给RPC，span通过一个64位ID唯一标识，trace以另一个64位ID表示，span还有其他数据信息，比如摘要、时间戳事件、关键值注释(tags)、span的ID、以及进度ID(通常是IP地址) span在不断的启动和停止，同时记录了时间信息，当你创建了一个span，你必须在未来的某个时刻停止它。
* **Trace**
	* 一系列spans组成的一个树状结构，例如，如果你要在分布式中大数据存储中使用，Trace将会由一个请求执行调用链形成。
* **Annotation**
	* 用来及时记录一个事件的存在，一些核心annotations用来定义一个请求的开始和结束。
	*cs：Client Sent - 客户端发起一个请求，这个annotion描述了这个span的开始
    *sr：Server Received - 服务端获得请求并准备开始处理它，如果将其sr减去cs时间戳便可得到网络延迟
    *ss：Server Sent - 注解表明请求处理的完成(当请求返回客户端)，如果ss减去sr时间戳便可得到服务端需要的处理请求时间
    *cr：Client Received - 表明span的结束，客户端成功接收到服务端的回复，如果cr减去cs时间戳便可得到客户端从服务端获取回复的所有所需时间
将Span和Trace在一个系统中使用Zipkin注解的过程图形化，如下图所示:

![Spring Cloud Sleuth使用ZipKin过程图形化](/images/spring-cloud-sleuth/1/trace-id.png)




