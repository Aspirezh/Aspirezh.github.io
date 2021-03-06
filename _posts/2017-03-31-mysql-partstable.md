---
layout: post
title:  "关于数据库分表"
date:   2017-03-31 11:23:18 +0800
categories: what
tags:  数据库 mysql 分表
author: Aspirezh
---

* content
{:toc}

虽然没有遇到过分表的问题，但是已经想到这个问题，在大数据量的情况下，分表应该是很重要的。



## 什么是数据库分表

从字面上简单理解，就是把原本存储于一个库的数据分块存储到多个库上，把原本存储于一个表的数据分块存储到多个表上。
## 为什么要分库分表
看你的业务数据量，一个数据库的数据量是无法控制的（一般是不可控的），随着业务量的增大，库中的表会越来越多，数据量越来越大，应地，数据操作，增删改查的开销也会越来越大；另外，由于无法进行分布式式部署，而一台服务器的资源（CPU、磁盘、内存、IO等）是有限的，最终数据库所能承载的数据量、数据处理能力都将遭遇瓶颈。
## 分库分表的实施策略
分库分表有垂直切分和水平切分两种。
### 垂直切分
何谓垂直切分，即将表按照功能模块、关系密切程度划分出来，部署到不同的库上。例如，我们会建立定义数据库workDB、商品数据库payDB、用户数据库userDB、日志数据库logDB等，分别用于存储项目数据定义表、商品定义表、用户数据表、日志数据表等。
### 水平切分
何谓水平切分，当一个表中的数据量过大时，我们可以把该表的数据按照某种规则，例如userID散列，进行划分，然后存储到多个结构相同的表，和不同的库上。例如，我们的userDB中的用户数据表中，每一个表的数据量都很大，就可以把userDB切分为结构相同的多个userDB：part0DB、part1DB等，再将userDB上的用户数据表userTable，切分为很多userTable：userTable0、userTable1等，然后将这些表按照一定的规则存储到多个userDB上。
### 切分方式
应该使用哪一种方式来实施数据库分库分表，这要看数据库中数据量的瓶颈所在，并综合项目的业务类型进行考虑。
如果数据库是因为表太多而造成海量数据，并且项目的各项业务逻辑划分清晰、低耦合，那么规则简单明了、容易实施的垂直切分必是首选。
而如果数据库中的表并不多，但单表的数据量很大、或数据热度很高，这种情况之下就应该选择水平切分，水平切分比垂直切分要复杂一些，它将原本逻辑上属于一体的数据进行了物理分割，除了在分割时要对分割的粒度做好评估，考虑数据平均和负载平均，后期也将对项目人员及应用程序产生额外的数据管理负担。
在现实项目中，往往是这两种情况兼而有之，这就需要做出权衡，甚至既需要垂直切分，又需要水平切分。我们的游戏项目便综合使用了垂直与水平切分，我们首先对数据库进行垂直切分，然后，再针对一部分表，通常是用户数据表，进行水平切分。

## 分库分表存在的问题
### 事务问题。
在执行分库分表之后，由于数据存储到了不同的库上，数据库事务管理出现了困难。如果依赖数据库本身的分布式事务管理功能去执行事务，将付出高昂的性能代价；如果由应用程序去协助控制，形成程序逻辑上的事务，又会造成编程方面的负担。
### 跨库跨表的join问题。
在执行了分库分表之后，难以避免会将原本逻辑关联性很强的数据划分到不同的表、不同的库上，这时，表的关联操作将受到限制，我们无法join位于不同分库的表，也无法join分表粒度不同的表，结果原本一次查询能够完成的业务，可能需要多次查询才能完成。
### 额外的数据管理负担和数据运算压力。
额外的数据管理负担，最显而易见的就是数据的定位问题和数据的增删改查的重复执行问题，这些都可以通过应用程序解决，但必然引起额外的逻辑运算，例如，对于一个记录用户成绩的用户数据表userTable，业务要求查出成绩最好的100位，在进行分表之前，只需一个order by语句就可以搞定，但是在进行分表之后，将需要n个order by语句，分别查出每一个分表的前100名用户数据，然后再对这些数据进行合并计算，才能得出结果。

##**下面是分库分表产生的问题，及注意事项**

###1.   分库分表维度的问题
假如用户购买了商品,需要将交易记录保存取来，如果按照用户的纬度分表，则每个用户的交易记录都保存在同一表中，所以很快很方便的查找到某用户的购买情况，但是某商品被购买的情况则很有可能分布在多张表中，查找起来比较麻烦。反之，按照商品维度分表，可以很方便的查找到此商品的购买情况，但要查找到买人的交易记录比较麻烦。
所以常见的解决方式有：
     a.通过扫表的方式解决，此方法基本不可能，效率太低了。
     b.记录两份数据，一份按照用户纬度分表，一份按照商品维度分表。
     c.通过搜索引擎解决，但如果实时性要求很高，又得关系到实时搜索。
2.   联合查询的问题
联合查询基本不可能，因为关联的表有可能不在同一数据库中。
3.   避免跨库事务
避免在一个事务中修改db0中的表的时候同时修改db1中的表，一个是操作起来更复杂，效率也会有一定影响。
4.   尽量把同一组数据放到同一DB服务器上
例如将卖家a的商品和交易信息都放到db0中，当db1挂了的时候，卖家a相关的东西可以正常使用。也就是说避免数据库中的数据依赖另一数据库中的数据。
一主多备
在实际的应用中，绝大部分情况都是读远大于写。Mysql提供了读写分离的机制，所有的写操作都必须对应到Master，读操作可以在Master和Slave机器上进行，Slave与Master的结构完全一样，一个Master可以有多个Slave,甚至Slave下还可以挂Slave,通过此方式可以有效的提高DB集群的QPS.                                                      
所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。
此外，可以看出Master是集群的瓶颈，当写操作过多，会严重影响到Master的稳定性，如果Master挂掉，整个集群都将不能正常工作。
所以，1. 当读压力很大的时候，可以考虑添加Slave机器的分式解决，但是当Slave机器达到一定的数量就得考虑分库了。 2. 当写压力很大的时候，就必须得进行分库操作。
## MySQL使用为什么要分库分表？
可以用说用到MySQL的地方,只要数据量一大, 马上就会遇到一个问题,要分库分表.
这里引用一个问题为什么要分库分表呢?MySQL处理不了大的表吗?
其实是可以处理的大表的.我所经历的项目中单表物理上文件大小在80G多,单表记录数在5亿以上,而且这个表
属于一个非常核用的表:朋友关系表.
但这种方式可以说不是一个最佳方式. 因为面临文件系统如Ext3文件系统对大于大文件处理上也有许多问题.
这个层面可以用xfs文件系统进行替换.但MySQL单表太大后有一个问题是不好解决: 表结构调整相关的操作基
本不在可能.所以大项在使用中都会面监着分库分表的应用.
从Innodb本身来讲数据文件的Btree上只有两个锁, 叶子节点锁和子节点锁,可以想而知道,当发生页拆分或是添加
新叶时都会造成表里不能写入数据.
所以分库分表还就是一个比较好的选择了.
那么分库分表多少合适呢?
经测试在单表1000万条记录一下,写入读取性能是比较好的. 这样在留点buffer,那么单表全是数据字型的保持在
800万条记录以下, 有字符型的单表保持在500万以下.
如果按 100库100表来规划,如用户业务:
500万*100*100 = 50000000万 = 5000亿记录.
心里有一个数了,按业务做规划还是比较容易的.
## 声明：以上整理于互联网

