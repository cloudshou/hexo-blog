---
layout:     post
title:      什么是Spring Cloud Config？
date:       2016-10-19 14:00:00 +0800
summary:    Spring Cloud Config是Spring Cloud系列中提供中心化配置管理的工具，本文主要介绍了Spring Cloud Config的实现细节。
toc: true
categories:
- Spring Cloud
tags:
- Spring Cloud Config
---
前言:在单体应用中，我们一般的做法是把Property和Code放在一起，没有什么问题。但是在分布式系统中，由于存在多个服务实例，需要分别管理到每个具体的服务工程中的配置，上线需要准备check list 并逐个检查每个上线的服务是否正确。在系统上线之后修改某个配置，需要重启服务。这样开发就相当麻烦。因此我们急需需要把分布式系统中的配置信息抽取出来统一管理，服务获取系统信息时有一个覆盖顺序:property--> Evn---->配置中心。这样修改环境变量或者修改配置中心的配置就能取到最新的配置信息。在唯品会 Venus Framework中我们专门设计了这个功能。Spring cloud出现之后，避免了大家重复造轮子。
## 什么是 Spring Cloud Config ?

其官方文档中对自己的定义是如下，官网连接:<a href="http://cloud.spring.io/spring-cloud-config/" target="_blank">Spring Cloud Config</a>。

> Spring Cloud Config provides server and client-side support for externalized configuration in a distributed system. 
> With the Config Server you have a central place to manage external properties for applications across all environments.


简单来说，Spring Cloud Config就是我们通常意义上的配置中心 - 把应用原本放在本地文件的配置抽取出来放在中心服务器，从而能够提供更好的管理、发布能力。
<!--more-->

另外，Spring Cloud Config提供基于以下3个维度的配置管理：

* **应用**
	* 这个比较好理解，每个配置都是属于某一个应用的
* **环境**
	* 每个配置都是区分环境的，如dev, test, prod等
* **版本**
	* 这个可能是一般的配置中心所缺乏的，就是对同一份配置的不同版本管理，比如:可以通过Git进行版本控制。
	* Spring Cloud Config提供版本的支持，也就是说对于一个应用的不同部署实例，可以从服务端获取到不同版本的配置，这对于一些特殊场景如：灰度发布，A/B测试等提供了很好的支持。

## 为什么会诞生Spring Cloud Config?
   配置中心目前现状:不管是开源的(百度的disconf)，还是一些公司自己闭源投入使用的产品已经不少了，那为什么还会诞生Spring Cloud Config呢？

在我看来，Spring Cloud Config在以下几方面还是有比较独特的优势，如下：

* **基于应用、环境、版本三个维度管理**
	* 这个在前面提过了，主要是有版本的支持
* **配置存储支持Git**
	* 这个就比较有特色了，后端基于Git存储，一方面程序员非常熟悉，另一方面在部署上会非常简单，而且借助于Git，天生就能非常好的支持版本
	* 当然，它还支持其它的存储如本地文件、SVN等
* **和Spring无缝集成**
	* 它无缝支持Spring里面`Environment`和`PropertySource`的接口
	* 所以对于已有的Spring应用程序的迁移成本非常低，在配置获取的接口上是完全一致的

## Spring Cloud Config 入门例子
上述节点主要介绍了Spring cloud的相关理论，大家对Spring Cloud Config有了一个初步的认识，接下来例子让大家感受一下Spring cloud config的魅力。

### Overview
![Overview](/images/2016-10-18/overview.png)

上图简要描述了一个普通Spring Cloud Config应用的场景。其中主要有以下几个组件：

* *Config Client*
	* Client很好理解，就是使用了Spring Cloud Config的应用
	* Spring Cloud Config提供了基于Spring的客户端，应用只要在代码中引入Spring Cloud Config Client的jar包即可工作
* *Config Server*
	* Config Server是需要独立部署的一个web应用，它负责把git上的配置返回给客户端
* *Remote Git Repository*
	* 远程Git仓库，一般而言，我们会把配置放在一个远程仓库，通过现成的git客户端来管理配置
* *Local Git Repostiory*
	* Config Server的本地Git仓库
	* Config Server接到来自客户端的配置获取请求后，会先把远程仓库的配置clone到本地的临时目录，然后从临时目录读取配置并返回
