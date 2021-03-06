
## 简介   

Postgres-XL是一个开源项目，它可以向PostgreSQL透明地提供写可伸缩性和大规模并行处理。它是紧密耦合的数据库组件的集合，可以在多个系统或虚拟机上安装。
可写扩展意味着Postgres-XL可以配置与您想要的数据库服务器一样多的服务器，并且处理更多的写入(更新SQL语句)，而不是单个独立的数据库服务器。您可以拥有多个提供单一数据库视图的数据库服务器。任何数据库服务器上的任何数据库更新都可以在不同服务器上运行的任何其他事务中看到。透明意味着您不必担心数据是如何存储在多个数据库服务器内部的。[1]   

您可以配置Postgres-XL在多个机器上运行。它以分布式方式存储您的数据，根据每个表所选择的内容进行分区或复制。[2]当您发布查询时，Postgres-XL将确定目标数据的存储位置，并将相应的计划发送到包含目标数据的服务器。
在典型的web系统中，您可以拥有许多web服务器或应用服务器来处理您的事务。但是，对于数据库服务器，您不能这样做，因为所有更改的数据必须对所有事务都可见。与其他数据库集群解决方案不同，Postgres-XL提供了这种功能。您可以安装任意数量的数据库服务器。每个数据库服务器为您的应用程序提供统一的数据视图。从任何服务器上的任何数据库更新都可以从其他服务器连接数据库的应用程序中看到。这是Postgres-XL最重要的特性之一。   

Postgres-XL的另一个重要特性是MPP并行性。您可以使用Postgres-XL来处理业务智能、数据仓库或大数据的工作负载。在Postgres-XL中，计划在协调器上生成一次，然后发送到各个数据节点。然后执行这个操作，数据节点直接与另一个节点进行通信，在这些节点上，每个节点都可以理解它在哪里接收到它需要的任何元组，以及它需要发送给其他的任何元组。   

## Postgres-XL的目标   

Postgres-XL的最终目标是在所有类型的数据库工作负载中提供具有ACID一致性的数据库可伸缩性。也就是说，Postgres-XL应该提供以下功能:   

* Postgres-XL应该提供多个服务器来接受来自应用程序的事务和语句，这些应用程序被称为“协调器”进程。   

* 任何协调器都应该为应用程序提供一致的数据库视图。任何来自任何协调器的更新都必须实时可见，就好像这些更新都是在一个PostgreSQL server中完成的。
Postgres-XL应该允许Datanodes以高效和并行的方式直接与另一个执行查询进行通信。   

* 表应该能够存储在被指定为复制或分布的数据库中(称为片段或分区)。复制和分发应该对应用程序透明;也就是说，这样的复制和分布式表被视为单个表，每个记录/ tuple的位置或数量都是由Postgres-XL管理的，对于应用程序来说是不可见的。   

* Postgres-XL为应用程序提供了兼容的PostgreSQL API。   

* Postgres-XL应该提供基础PostgreSQL数据库服务器的单一和统一视图，以便SQL语句不依赖于如何实际存储表。   

## Postgres-XL关键组件   

在本节中，我们将介绍Postgres-XL的主要组件。   

Postgres-XL由三个主要组件组成:GTM(全局事务管理器)、Coordinator(协调器)和Datanode(数据节点)。它们的特性在下面几节中给出。   

## GTM(全球事务管理器)

GTM是Postgres-XL的一个重要组成部分，可以提供一致的事务管理和元组可视性控制。   

正如本文后面所述，PostgreSQL的事务管理基于MVCC(多版本并发控制)技术。Postgres-XL将此技术提取到单独的组件，如GTM，以便任何Postgres-XL组件的事务管理都基于单个全局状态。细节将在第48章中描述。   

## 协调器   

协调器是应用程序数据库的接口。它就像一个传统的PostgreSQL后台进程，但是协调器不存储任何实际的数据。实际数据存储在如下所述的Datanodes中。协调器接收SQL语句，根据需要获取全局事务Id和全局快照，确定所涉及的datanode，并要求它们执行(部分)语句。在向Datanodes发送语句时，它与GXID和全局快照相关联，以便多版本并发控制(MVCC)属性扩展集群范围。   

## Datanode   

Datanode实际上存储用户数据。表可以分布在数据节点中，或者复制到所有数据节点。Datanode不具有整个数据库的全局视图，它只负责本地存储的数据。接下来将由协调器检查传入语句，并进行子计划。然后将这些数据传输到每个Datanode，并根据需要将其与GXID和全局快照结合在一起。datanode可以在单独的会话中接收来自各个协调器的请求。但是，由于每个事务都是唯一的，并且与一致的(全局)快照相关联，所以每个Datanode都可以在其事务和快照上下文中适当地执行。   

## Postgres-XL继承自PostgreSQL   

Postgres-XL是PostgreSQL的扩展，并继承了它的大部分特性。   

它是PostgreSQL及其原始的Berkeley代码的一个开源后代。它支持很大一部分SQL标准，并提供了许多现代特性:   

* 复杂的查询
* 外键[3]
* 触发[4]
* 视图
* 事务完整性，除SSI之外，其支持是不完整的
* 多版本并发控制   

同样，与PostgreSQL类似，Postgres-XL可以在许多方面由用户扩展，例如添加新的   

* 数据类型   
* 功能
* 操作员
* 聚合函数
* 索引方法
* 程序语言   

Postgres-XL可以被任何人免费使用、修改和分发，无论是私有的、商业的还是学术的，只要它遵循PostgreSQL许可。   

## 笔记
* [1] 当然，您应该在设计数据库物理上从Postgres-XL获得最多的时候，使用关于如何存储表的信息。
* [2] 为了区别PostgreSQL的本地分区，我们将其称为“分布”。在分布式数据库教程中，这通常被称为“水平分片”。
* [3] Postgres-XL的外键使用有一些限制。有关详细信息，请参见[CREATE TABLE](建表语法.md)。
* [4] Postgres-XL在当前版本中不支持触发器。这可能在以后的版本中得到支持。
