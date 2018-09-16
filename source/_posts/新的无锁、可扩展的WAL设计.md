---
title: MySQL 8.0: 新的无锁、可扩展的WAL设计
date: 2018-09-01 17:13:29
tags: MySQL内核
---

## 1. 前言

Write Ahead Log (WAL) 是数据库的重要组成部分之一。对数据文件的所有改动都被记录在 WAL (在 Innodb 中称为 redo log，即重做日志)。这样就允许将被修改的页刷新到磁盘的操作延时进行，并且能防止丢失数据。

在写入重做日志时，涉及许多用户线程的写入密集型工作负载的性能受到同步的限制。在具有多个CPU核心和快速存储设备(如现代SSD磁盘)的服务器上测试性能时，这一点尤为明显。

![redolog-old-1](/home/longhai/Nutstore/blog/redolog-old-1.png)

我们需要一种新的设计来解决我们的客户和用户现在和将来可能面临的问题。调整旧设计以实现可扩展性不再是一种选择。新设计还必须灵活，以便我们将来可以扩展它来进行分片和并行写入。对于新的设计，我们希望确保它能与现有的 API 一起工作，最重要的是不要违反 InnoDB 的其他部分所依赖的 contract？？。在这些限制下，这是一项具有挑战性的任务。

![redolog-new-2](https://mysqlserverteam.com/wp-content/uploads/2018/05/redolog-new-2.png)

重做日志可以被视为生产者/消费者持久队列。执行更新的用户线程可以被视为生产者，当 InnoDB 必须执行崩溃恢复时，恢复线程就是消费者。当服务器运行时，InnoDB 不会从重做日志中读取任何数据。

![](/home/longhai/Nutstore/blog/redo-bufferpool-1.png)

