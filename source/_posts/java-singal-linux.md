---
layout:     post
title:      使用Java Signal将应用程序从LVS中摘除
date:       2016-11-04 14:00:00 +0800
summary:    Janus服务网关本地离线操作，让其从LVS中摘除。
toc: true
categories:
- 项目经验
tags:
- Java
- 项目经验
---
**摘要**:本文主要介绍了，如何使用Java Signal和SignalHandler实现，通过Linux 命令实现kill -s BUS pid和kill -s USR2 pid实现不kill应用进程，把应用程序从LVS中摘除。而不是通过Reset 请求调用。由于只允许本机操作，所以可选方案三种:1.reset 调用更改Status，2.Linux 信号量传递给Java程序 3.配置中心或者XX管理系统后台权限管理，调用reset服务。从安全性和快速解决需求的角度考虑使用Linux 信号量传递给Java程序方案。
## Java Signal 概述
 
### 信号简介
信号是在软件层次上对中断机制的一种模拟，在原理上，一个进程收到一个信号与处理器收到一个中断请求可以说是一样的。通俗来讲，信号就是进程间的一种异步通信机制。
典型的例子:`kill -s SIGKILL pid` (即kill -9 pid) 立即杀死指定pid的进程。
在上面这个例子中，SIGKILL就是往pid进程发送的信号。
<!--more-->
### 平台相关性
 信号具有平台相关性，不同平台下能使用的信号种类是有差异的。
在Linux下支持的信号(对比信号列表查看描述)
　SEGV, ILL, FPE, BUS, SYS, CPU, FSZ, ABRT, INT, TERM, HUP, USR1, USR2, QUIT, BREAK, TRAP, PIPE
在Windows下支持的信号
　SEGV, ILL, FPE, ABRT, INT, TERM, BREAK
### 信号选择
   为了不干扰正常信号的运作，又能模拟Java异步通知，我们需要先选定一种特殊的信号。
通过查看信号列表上的描述，发现 SIGUSR1 和 SIGUSR2 是允许用户自定义的信号。
那么选择它们，理论上就不会影响正常功能了。这里我选用了`BUS`和`USR2`作为传递信号。原因是USR1在Linux系统下面，很大可能性会被其它应用占用。在本次实践中，就是被占用导致`handle`出现异常。

## 实现代码
### JDK API实现调研
  Sun为我们提供了2个方便安装和替换信号处理器的工具类。通过下面的api可以快速实现。
  sun.misc.Signal
  sun.misc.SignalHandler

### JanusSignalHandler的code
```java
public class JanusSignalHandler implements SignalHandler {
    private static Logger logger = LoggerFactory.getLogger(JanusSignalHandler.class);
    @Override
    public void handle(Signal signal) {
        if (null != signal) {
            signalHandle(signal);
        }
    }
    private void signalHandle(Signal sn) {
        if (sn.getName().equals("BUS")) {
            JanusNettyServer.online = false;
            logger.info("Signal name is:SIGBUS,online is:false");
        } else if (sn.getName().equals("USR2")) {
            JanusNettyServer.online = true;
            logger.info("Signal name is:SIGUSR2,online is:true");
        } else {
            return;
        }
    }
} 
   
```
### addSingalHook的Code
   应用程序启动的时候，调用此方法install signals
```java
private void addSingalHook() {
        try {
            JanusSignalHandler janusSignalHandler = new JanusSignalHandler();
            // install signals
            Signal.handle(new Signal("BUS"), janusSignalHandler);
            Signal.handle(new Signal("USR2"), janusSignalHandler);
        } catch (IllegalArgumentException e) {
            logger.error("exception:[{}]", e.getMessage());
        }
    } 
```
### 部署到Linux程序中Test
执行 kill -s BUS pid  从LVS中摘除
执行 kill -s USR2 pid 加入LVS中

![linus中测试](/images/project/20161104/test.png)

健康检查result:
![从LVS中摘除](/images/project/20161104/offline.png)

![加入LVS中 ](/images/project/20161104/online.png)

  
## 总结
   本文主要介绍了，如何使用Java Signa和SignalHandler实现，通过Linux 命令实现kill -s BUS pid和kill -s USR2 pid实现不kill应用进程，把应用程序从LVS中摘除。但是在实践过程中，需要选对用户可以自定义的信号量。不然，会误杀应用程序本身。
