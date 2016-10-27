
---
title: Jdk的万能配置
categories: java
date: 2016-10-15 21:00:00 +0800
tags:
- Jdk
- Java
---
　java是通过java虚拟机来解释运行的,也就是通过java命令; javac编译生成的.class文件就是虚拟机要执行的代码, 称之为字节码(bytecode),虚拟机通过classloader来装载这些字节码,也就是通常意义上的类.这里就有一个问题,classloader从哪里知道java本身的类库及用户自己的类在什么地方呢?或者有着缺省值(当前路径).或者要有一个用户指定的变量来表明, 这个变量就是类路径(classpath),或者在运行的时候传参数给虚拟机.
通过这段文字，你就知道，为什么javac编译通过了，但是java命令却出错(类定义没找到)的原因了。
就是环境变量classpath(类路径)没有设置正确，使得JAVA虚拟机的classloader无法找到类来执行目标程序
<!--more-->

## 快速配置

### 1.新建系统变量JAVA_HOME变量(JAVA_HOME指明JDK安装路径。)

　E:\development\Java\Java8\jdk1.8.0_73

### 2.在系统变量中的path中添加(Path使得系统可以在任何路径下识别java命令。)

　;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

### 3、新建系统变量CLASSPATH(CLASSPATH为java加载类(class or lib)路径，只有类在classpath中，java命令才能识别.)
　设定值为：.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
　注意 一定要加“.”，“.”代表当前目录，即可到处建立.java文件，java class都能找到并编译运行用户的.java文件。
### 4.进入dos窗口运行“java –version" 如果显示下面内容则成功。

![Java -version](/images/2016-10-18/jdk-config.png)

---
