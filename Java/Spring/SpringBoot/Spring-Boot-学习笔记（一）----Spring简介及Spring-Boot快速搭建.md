不久前在学习Spring Cloud的相关知识，原本准备基于Spring Cloud来搭建自己的网站的，但最近工作实在有点繁忙，看书断断续续的，书包里的kindle强行撑了两个多礼拜没关机，现在对于之前看的东西已经忘了不少，因此现在把做的笔记以及编写的demo都记录下来，以便分享给他人（其实我知道没人看）以及自己后续翻看复习。

## Spring简介

在说Spring Boot之前大家应该都听说过Spring。

> Spring框架是一个轻量级的企业级开发的一站式解决方案。
>
>
> 所谓解决方案就是可以基于Spring解决JavaEE开发的所有问题。Spring框架主要提供了IoC容器、AOP、数据访问、Web开发、消息、测试等相关技术的支持。
>     Spring使用简单的POJO（Plain Old Java Object）来进行企业级开发。每一个被Spring管理的Java对象都称之为Bean；而Spring提供了一个IoC容器来初始化对象，解决对象之间的依赖管理和对象的使用。

Spring目前提供了大量的基于Spring的项目，如我本文要讲述的SpringBoot以及基于Spring Boot而产生的Spring Cloud：

 - Spring Boot：使用默认开发配置来实现快速开发；
 - Spring Cloud：为分布式系统开发提供工具集；

Spring框架本身又四大原则：
- 使用POJO进行轻量级和最小侵入式开发；
- 通过依赖注入和基于接口变成实现松耦合；
- 通过AOP和默认习惯进行声明式编程；
- 使用AOP和模板（template）减少模式化代码；
Spring所有功能的设计和实现都是基于此四大原则的。

## Spring Boot概述

相对于我们传统常用的SpringMVC的一大堆繁琐的xml配置，Spring Boot的“习惯优于配置”理念使得我们很容易创建一个独立运行的，准生产级别的基于Spring框架项目。其核心功能包括以下几点：
- 独立运行的Spring项目：可以通过jar包形式运行，即java -jar xx.jar；
- 内嵌servlet容器，无需以war包形式部署项目；
- 提供starter简化Maven配置：Spring提供了一系列的starter pom简化maven的依赖加载，后文创建Spring Boot项目会看出来；
- 自动配置Spring：Spring Boot会根据在类路径中的jar包、类，为jar包中的类自动配置Bean，这样会极大的减少我们使用的配置。
- 准生产的应用监控；
- 无代码生产和xml配置：Spring Boot的神奇不是借助代码生成实现的，而是通过条件注解来实现。这是Spring4.x的新特性，在这，Spring4.x提倡使用Java配置和注解配置组合，Spring Boot不需要任何xml配置额即可实现Spring所有配置。

## Spring Boot快速搭建
本人使用的开发工具是IDEA，创建Spring Boot项目也比较方便，也推荐大家使用此工具，颜值高，开发体验也很好。
1新建Spring Initializr项目
![image](http://upload-images.jianshu.io/upload_images/1537405-318c98e73b7a10b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.填写项目信息，选择项目使用技术，我们这里选择Web就可以了，填好项目名称，finish。
![image](http://upload-images.jianshu.io/upload_images/1537405-a11c5617232cad23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image](http://upload-images.jianshu.io/upload_images/1537405-1dd1ccfa4c1270fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image](http://upload-images.jianshu.io/upload_images/1537405-a710a01fbcb6039d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
至此，项目搭建结束，其项目依赖如图：
![image](http://upload-images.jianshu.io/upload_images/1537405-7fddb8cadf130a36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
同时我们可以看到，在我们groupId+artifactId组合包名下有个入口类：
![image](http://upload-images.jianshu.io/upload_images/1537405-c6a7c93238556d97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们启动该类中的main方法，这里即是启动本项目：
![image](http://upload-images.jianshu.io/upload_images/1537405-47a1ba03c21df07f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
控制台打印输出启动成功，我们为了能看到页面效果，给该类加上@RestController注解，并添加index方法：
![image](http://upload-images.jianshu.io/upload_images/1537405-1cd9c4ecf87da138.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再次启动项目，访问localhost:8080 如图所示
![image](http://upload-images.jianshu.io/upload_images/1537405-9250c19eb2364fa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过查看源码可以发现，我们这里的*Application的入口类上的注解@SpringBootApplication注解实际上是组合了@Configuration、@EnableAutoConfiguration、@ComponentScan这几个注解，若不适用@SpringBootApplication，也可使用这三个注解代替。其中@EnableAutoConfiguration让Spring Boot根据类路径中的jar包依赖为当前项目进行自动配置。
	Spring Boot会自动扫描@SpringBootApplication所在类的同级包，因此建议入口类防止在groupId+artifactId组合包名下。
	这里也顺便讲个有意思的东西，我们启动Spring Boot的时候是有个默认启动图案的，其实这个是可以换的，我们在src/main/resources下新建一个banner.txt，然后去http://patorjk.com/software/taag 网站生成字符，然后将字符复制到banner.txt中，我写的是LEAFW，再次启动如图：
	![image](http://upload-images.jianshu.io/upload_images/1537405-f6b079ed773e12be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	
以上，即为Spring Boot学习第一章节，也是最最最最为基础的东西，后续会更新进一步的学习笔记。

本博客中所有相关代码已上传至本人GitHub的[leafw-blog-demo][1]项目下，如有需要可下载查看。本文对应的为springboot_1目录。

本文参考书籍：博文视点-Spring Boot实战

[1]: https://github.com/careywyr/leafw-blog-demo "leafw-blog-demo"
