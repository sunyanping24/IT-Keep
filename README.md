<!-- TOC -->

- [Tools](#tools)
- [Java基础](#java基础)
- [Database](#database)
  - [MySQL](#mysql)
  - [Redis](#redis)
- [消息中间件](#消息中间件)
- [Framework](#framework)
  - [Mybatis](#mybatis)
  - [分布式架构](#分布式架构)
  - [Spring](#spring)
  - [SpringBoot](#springboot)
  - [Netty](#netty)
- [BuildTools](#buildtools)
- [Network(网络基础)](#network网络基础)
- [DevOps](#devops)
- [Java专题](#java专题)
- [[杂混整理](杂混整理.md)](#杂混整理杂混整理md)
- [GroceryStore](#grocerystore)
- [文档说明](#文档说明)

<!-- /TOC -->

***

# Tools
*  [GIT](Tools/GIT.md)
*  [idea](Tools/idea.md)
*  [vscode](Tools/vscode.md)
*  [linux](Tools/linux.md)
*  [centos](Tools/centos.md)
*  [虚拟机及服务](Tools/虚拟机及服务.md)
*  [windows](Tools/windows.md)

# Java基础
* [Java基础知识整理](Java基础/Java基础知识整理.md)
* [JVM知识点整理](Java基础/JVM知识点整理.md)
* [Java多线程](Java基础/Java多线程.md)
* [Java IO](Java基础/Java%20IO.md)
* [Java8的新特性使用](Java基础/Java8的新特性使用.md)
* [Java容器](Java基础/Java容器.md)

# Database
- [MySQL](#MySQL)
- [Redis](#Redis)
## MySQL
* [数据库连接池](Database/MySQL/数据库连接池.md)
* [Mysql常用操作](Database/MySQL/Mysql常用的操作.md)
* [Mysql存储过程、函数](Database/MySQL/Mysql存储过程、函数.md)
* [MySQL常见面试题](Database/MySQL/MySQL常见面试题.md)
* [MySQL高性能优化规范建议](Database/MySQL/MySQL高性能优化规范建议.md)
* [Mysql服务常见问题处理](Database/MySQL/Mysql服务常见问题处理.md)
## Redis
* [Redis](Database/Redis/Redis.md)
* [Redis的单线程模型](Database/Redis/Redis的单线程模型.md)
* [Redis的三种集群模式](Database/Redis/Redis的三种集群模式.md)

# 消息中间件
* [Kafka常用操作](Messages/Kafka常用操作.md)
* [Kafka入门整理](Messages/Kafka入门整理.md)
* [RabbitMQ入门整理](Messages/RabbitMQ入门整理.md)

# Framework
- [Mybatis](#Mybatis)
- [分布式架构](#分布式架构)
- [SpringBoot](#SpringBoot)
- [Spring](#Spring)
- [Netty](#Netty)

## Mybatis
* [Mybatis常见面试题](Framework/Mybatis/Mybatis常见面试题.md)

## 分布式架构
* [微服务架构的设计](Framework/分布式架构/微服务架构的设计.md)
* [分布式架构基础知识](Framework/分布式架构/分布式架构基础知识.md)

## Spring
* [Spring框架和常见面试题](Framework/Spring/Spring框架和常见面试.md)

## SpringBoot
* [SpringBoot常见面试题](Framework/SpringBoot/SpringBoot常见面试题.md)

## Netty
* [快速创建一个netty服务端](Framework/Netty/快速创建一个netty服务端.md)

# BuildTools
* [maven](BuildTools/maven.md)

# Network(网络基础)
* [计算机网络基础](Network/计算机网络基础.md)
* [后端Web常见面试题](Network/后端Web常见面试题.md)

# DevOps
* [认识Docker](DevOps/认识Docker.md)
* [Dockerfile的编写](DevOps/Dockerfile的编写.md)
* [Docker与K8S的理解](DevOps/Docker与K8S的理解.md)

# Java专题
* [Java锁机制](Java专题/Java锁机制.md)

# [杂混整理](杂混整理.md)

# GroceryStore
* [还没来得及归类整理](GroceryStore/还没来得及归类整理.md)

***
# 文档说明

**文档内容说明**    
1. 该文档由我个人维护整理，纯粹是当作个人笔记来做的，维护在这个平台仅仅是为了查看笔记方便，加强和有同样兴趣的人的交流。**不能用于任何商业用途**。
2. 文档内容形式包含自创、引用其它处文章。引用的文章一般都放了原文链接，支持鼓励大家访问原创作者的文章内容。
3. 若有其它想参与进行维护的朋友可以提 `pull request`，但是注意需要和文档原内容维护方式保持一致。

**文档维护注意点**    
1. 注意文档目录：若添加目录、文件，则需要在 `README.md` 文件中主动编写引导目录；编写的内容文件中使用 `TOC` 工具在文件开始位置生成对应文章目录。
2. 注意图片引用：若文中包含图片，需要将图片移动到根目录 **/ASSET/*** 目录下，然后在文中使用 `![名称](http://sunyanping.gitee.io/it-keep/ASSET/xxx.jpg)` 进行引用。注意路径中 `/ASSET/xxx.jpg` 部分是文件在 `/ASSET/*` 目录的位置，若在该目录下创建了子目录，注意要使用正确的引用。当然在开发时可以在本地先使用 `![名称](/ASSET/xxx.jpg)` 这种方式测试图片显示正常，然后再在链接前面加上 `http://sunyanping.gitee.io/it-keep` 即可。
***
