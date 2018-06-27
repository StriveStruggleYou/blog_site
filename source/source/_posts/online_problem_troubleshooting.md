title: 线上问题排查过程
author: Chen Mr
tags:
  - 问题排查
categories:
  - 漫游笔记
date: 2018-06-27 09:30:00
---
线上问题处理步骤：发现问题->快速恢复->定位与修复->方法论-为故障和失败做设计

需关注的系统参数：

应用层：接口响应时间，qps,并发数

软件层：jvm,DB，缓存

系统层：CPU，内存，IO

1.发现问题：人肉主动发现->生产事件上报->关联系统故障追溯->业务监控报警->系统监控报警

2.快速恢复：验证->排查解决->恢复服务->保留现场

3.定位与修复:

常规操作：重启，回滚，降级，摘机

![avatar](https://upload-images.jianshu.io/upload_images/3943604-d5f68ba67e3e18f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

图1：现象收集&故障定位

![avatar](https://upload-images.jianshu.io/upload_images/3943604-4a28b55f3f65c76c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


图2:故障排除&服务恢复
特殊场景：无法定位故障

原则：确保线上服务快速恢复，不能完全恢复的情况下，确保线上服务尽可能少的受到影响

手段：

1）服务降级：定位到服务异常，但不清楚异常原因，直接降级该服务，确保其他服务不受影响

2）服务紧急扩容：服务器资源飙升但无法定位到问题时，紧急扩容服务器（可能为恶意攻击，促销活动，秒杀等情况）

3）回退版本：有新版本发布，但是不能确定故障是否和新版本有关系，先回退到上一个稳定版本

保留现场

1）执行top命令，观察Cpus-ids（CPU的空闲程度）,值过低时，shift+P按使用率倒排，记录最耗资源的进程信息

2）执行free -m命令，观察cache行free列的值，值过低是，执行top命令，shift+m按内存使用量倒排，记录最耗资源的进程信息

3）对耗资源进程执行ps xuf| grep pid命令,打印进程具体信息并记录

4）执行jstack pid打印日志，取多组方便比较

5）执行jstat -gcutil查看Old区占用率，若达到或接近100%，则执行jmap -histo pid

常见故障原因

![avatar](https://upload-images.jianshu.io/upload_images/3943604-080bcc6dde4308a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/580)
图3-常见故障原因
![avatar](https://upload-images.jianshu.io/upload_images/3943604-7a0b773f07bcb517.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

图4-故障画像
问题排查常用命令：

CPU：top -Hp

内存：free -m

IO： iostat

磁盘：df -h

网络连接：netstat

GC:jstat -gcutil(建议重点了解)

线程：jstack

内存：jmap

辅助工具：MAT，btrace，jprofile

4.方法论

系统资源的异常现象：cpu飙高&内存不足&磁盘IO高&网络连接高

服务内部的异常现象：OOM&异常日志&疑难杂症(死锁、死循环、等待外部响应)


![avatar](https://upload-images.jianshu.io/upload_images/3943604-b8e3af8b53e02fda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/552)
图5：逐步排查
5.为故障和失败做设计

1）故障发生时尽可能维持系统核心功能的可用性

2）依赖模型&依赖治理

3）超时机制（系统超时、网络超时、Fail fast）

4) 回退机制

5）熔断器

* [漫游鹰](https://www.jianshu.com/p/b4f958fb7fcf)