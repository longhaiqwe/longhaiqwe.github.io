---
title: 每天5分钟学MySQL内核(3) -- MySQL redo log 简介
date: 2018-09-01 16:49:53
tags: MySQL内核
---

> 本文采用创作共用署名2.5中国大陆版许可证（Creative Commons Attribution 2.5 China Mainland License）授权。

[[1] MySQL · 源码分析 · Innodb 引擎Redo日志存储格式简介](http://mysql.taobao.org/monthly/2017/09/07/)

[[2] InnoDB存储引擎Log](https://blog.csdn.net/qiuyepiaoling/article/details/7838951)

## 1. 概念 

任何对 Innodb 表的变动, redo log 都要记录对数据的修改，redo 日志就是记录要修改后的数据。redo 日志是保证事务一致性非常重要的手段，同时也可以使在 bufferpool 修改的数据不需要在事务提交时立刻写到磁盘上减少数据的 IO 从而提高整个系统的性能。这样的技术推迟了 bufferpool 页面的刷新，从而提升了数据库的吞吐，有效的降低了访问时延。带来的问题是额外的写 redo log 操作的开销。而为了保证数据的一致性，都要求 WAL（Write Ahead Logging）。而 redo 日志也不是直接写入文件，而是先写入 redo log buffer，而是批量写入日志。当需要将日志刷新到磁盘时（如事务提交），将许多日志一起写入磁盘。

## 2. 原理

- 和 undo log 相反，redo log 记录的是**新数据**的备份。
- 在事务提交前，只要将 redo log 持久化即可，不需要将数据持久化。
- 当系统崩溃时，虽然数据没有持久化，但是 redo log 已经持久化。
- 系统可以根据 redo log 的内容，将所有数据恢复到最新的状态。 

## 3. 案例

**undo + redo 事务的简化过程**

![undo+redo保证原子性与持久性示例](http://oi435vw1u.bkt.clouddn.com/undo+redo%E4%BF%9D%E8%AF%81%E5%8E%9F%E5%AD%90%E6%80%A7%E4%B8%8E%E6%8C%81%E4%B9%85%E6%80%A7%E7%A4%BA%E4%BE%8B.jpg)

- 为了保证持久性，必须在事务提交前将 redo log 持久化
- 数据不需要在事务提交前写入磁盘，而是缓存在内存中
- redo log 保证事务的持久性
- undo log 保证事务的原子性
- 有一个隐含的特点，数据必须要晚于 redo log 写入持久存储

## 4. 性能

> undo + redo 的设计主要考虑的是提升 IO 性能。
>
> 虽说通过缓存数据，减少了写数据的 IO，但是却引入了新的IO，即写 redo log 的 IO。
>
> 如果 redo log 的 IO 性能不好，就不能起到提高性能的目的。

为了保证 redo log 能够有比较好的 IO 性能，InnoDB 的 redo log 的设计有以下几个特点：

- 尽量持 redo log 存储在一段连续的空间上。因此在系统第一次启动时就会将日志文件的空间完全分配，以顺序追加的方式记录 redo log，通过顺序 IO 来改善性能。
- 批量写入日志。日志并不是直接写入文件，而是先写入 redo log buffer。当需要将日志刷新到磁盘时 （如事务提交），将许多日志一起写入磁盘。
- 并发的事务共享 redo log 的存储空间，它们的 redo log 按语句的执行顺序，依次交替的记录在一起，以减少日志占用的空间。例如，redo log 中的记录内容可能是这样的：
  - 记录1:  <trx1, insert …>
  - 记录2:  <trx2, update …>
  - 记录3:  <trx1, delete …>
  - 记录4:  <trx3, update …>
  - 记录5:  <trx2, insert …>
- 基于上一点，当一个事务将 redo log 写入磁盘时，也会将其他未提交的事务的日志写入磁盘。
- redo log 上只进行顺序追加的操作，当一个事务需要回滚时，它的 redo log 记录也不会从 redo log 中删除掉。