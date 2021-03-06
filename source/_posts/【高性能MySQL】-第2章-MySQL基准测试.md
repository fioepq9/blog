---
title: 【高性能MySQL】- 第2章  MySQL 基准测试
date: 2022-04-27 15:36:30
tags:
  - 阅读笔记
  - MySQL
  - 高性能MySQL
categories:
  - 数据库
  - MySQL
---

## 基准测试是什么？

基准测试，是针对系统设计的一种压力测试。通常的目标是为了掌握系统的行为。但也有其他原因，如重现某个系统状态，或者是做新硬件的可靠性测试。

### 为什么需要基准测试？

因为基准测试是唯一方便有效的、可以学习系统在给定的工作负载下会发生什么的方法。

### 基准测试可以完成的工作？

+ 验证基于系统的一些假设，确认这些假设是否符合实际情况。
+ 重现系统中的某些异常行为，以解决这些异常。
+ 测试系统当前的运行情况。（即弄清楚系统当前的性能，以便用于确认某些优化的效果如何）
+ 模拟比当前系统更高的负载，以找出系统随着压力增加而可能遇到的扩展性瓶颈。
+ 规划未来的业务增长。（通过基准测试，可以评估在项目未来的负载下，需要什么样的硬件，需要多大容量的网络，以及其他相关资源。这有助于降低系统升级和重大变更的风险。）
+ 测试应用适应可变环境的能力。
+ 测试不同的硬件、软件和操作系统配置。
+ 证明新采购的设备是否配置正确。

### 基准测试的主要问题？

+ 其不是真实压力的测试。基准测试施加给系统的压力相对真实压力来说，通常比较简单。

#### 基准测试的压力和真实压力在哪些方面不同？

+ 数据量、数据和查询的分布。

+ 基准测试通常要求尽可能快地执行完成，所以经常给系统造成过大的压力。

## 基准测试的策略

+ 集成式（full-stack）基准测试：针对整个系统的整体测试。
+ 单组件式（single-component）基准测试：单独测试 MySQL。

### 为什么采用集成式测试？

+ 用户关注的并不仅仅是 MySQL 本身的性能，而是应用整体的性能。
+ MySQL 并不一定是应用的瓶颈，通过整体的测试可以揭示这一点。
+ 只有对应用做整体测试，才能发现各部分之间的缓存带来的影响。
+ 整体应用的集成式测试更能揭示应用的真实表现，而单组件式测试很难做到这一点。

#### 集成式测试存在的问题？

应用的整体基准测试很难建立，甚至很难正确设置。如果基准测试的设计有问题，那么结果就无法反映真实情况，基于此做的决策也就无法保证正确性。

### 为什么采用单组件式测试？

有时候不需要了解整个应用的情况，而只需要关注 MySQL 的性能。（比如，在项目的初期可以这样做）

+ 需要比较不同的 schema 或查询的性能。
+ 针对应用中某个具体问题的测试。
+ 为了避免漫长的基准测试，可以通过一个短期的基准测试，做快速的「周期循环」，来检测出某些调整后的效果。

### 测试有哪些指标？

+ 吞吐量

吞吐量，指的是单位时间内的事务处理数。常用的测试单位是每秒事务数（TPS），有些也采用每分钟事务数（TPM）。

+ 响应时间或者延迟

这个指标用于测试任务所需的整体时间。通常使用百分比响应时间（percentile response time）。

+ 并发性

Web 服务器的并发性，指的是在任意时间有多少同时发生的并发请求。并发性基准测试需要关注的是正在工作中的并发操作，或者是同时工作中的线程数或者连接数。当并发性增加时，需要测量吞吐量是否下降，响应时间是否变长，如果是这样，应用可能就无法处理峰值压力。

+ 可扩展性

可扩展性指的是，给系统增加一倍的资源，在理想情况下就能获得两倍的吞吐量。可扩展性指标对于容量规范非常有用，它可以提供其他测试无法提供的信息，来帮助发现应用的瓶颈。

## 基准测试的常见错误

+ 使用真实数据的子集，而不是使用全集。
+ 使用错误的数据分布。
+ 使用不真实的分布参数。
+ 在多用户场景中，只做单用户的测试。
+ 在单服务器上测试分布式应用。
+ 与真实用户行为不匹配。
+ 反复执行同一个查询。
+ 没有检查错误。
+ 忽略了系统预热（warm up）的过程。
+ 使用默认的服务器配置。
+ 测试时间太短。

## 如何设计一个基准测试？

1. 提出问题并明确目标。
2. 决定采用标准的基准测试，还是设计专用的测试。

### 采用标准的基准测试

确认选择了合适的测试方案。

### 设计专用的测试

1. 需要获得生产数据集的快照，并且该快照很容易还原，以便进行后续的测试。
2. 针对数据运行查询。
   1. 可以建立一个单元测试集作为初步的测试，并运行多遍。
   2. 更好的办法：选择一个有代表性的时间段，比如高峰期的一个小时，或者一整天，记录生产系统上的所有查询。如果时间段选得比较小，则可以选择多个时间段。这样有助于覆盖整个系统的活动状态。
3. 详细地写下测试规划。测试规划应该记录测试数据、系统配置的步骤、如何测量和分析结果，以及预热的方案。
4. 应该建立将参数和结果文档化的规范，每一轮测试都必须进行详细记录。
5. 执行基准测试时，需要尽可能多地收集被测试系统的信息。最好为基准测试建立一个目录，并且每执行一轮测试都创建单独的子目录，将测试结果、配置文件、测试指标、脚本和其他相关说明都保存在其中。
   + 需要记录的数据：系统状态和性能指标，诸如 CPU 使用率、磁盘 I/O、网络流量统计、`SHOW GLOBAL STATUS`计数器等。

## 基准测试应该运行多长时间？

基准测试应该运行足够长的时间，这一点很重要。

有时候无法确认测试需要运行多次的时间才足够。如果是这样，可以让测试一直运行，持续观察直到确认系统已经稳定。

一个简单的测试规则，就是等系统看起来稳定的时间至少等于系统预热的时间。

## 如何获得准确的基准测试结果？

回答一些关于基准测试的基本问题。

+ 是否选择了正确的基准测试？
+ 是否为问题收集了相关的数据？
+ 是否采用了错误的测试标准？

确认测试结果是否可重复。

+ 如果测试的是经过预热的系统，确保预热的时间足够长、是否可重复。（如果预热采用的是随机查询，那么测试结果可能就是不可重复的）
+ 如果测试的过程会修改数据或者 schema，那么每次测试前，需要利用快照还原数据。（数据的碎片度和在磁盘上的分布，都可能导致测试是不可重复的。一个确保物理磁盘数据的分布尽可能一致的办法是，每次都进行快速格式化并进行磁盘分区复制）
+ 要注意很多因素，包括外部的压力、性能分析和监控系统、详细的日志记录、周期性作业，以及其他一些因素，都会影响到测试结果。

每次测试中，修改的参数应该尽量少。如果必须要一次修改多个参数，那么可能会丢失一些信息。

+ 一般情况下，都是通过迭代逐步地修改基准测试的参数，而不是每次运行时都做大量的修改。

基于 MySQL 的默认配置的测试没有什么意义，因为默认配置是基于消耗很少内存的极小应用的。

如果测试中出现异常结果，不要轻易当作坏数据点而丢弃。应该认真研究并找到产生这种结果的原因。

## 基准测试工具

### 集成式测试工具

+ ab

ab 是一个 Apache HTTP 服务器基准测试工具。它可以测试 HTTP 服务器每秒最多可以处理多少请求。如果测试的是 Web 应用服务，这个结果可以转换成整个应用每秒可以满足多少请求。

这是个非常简单的工具，用途也有限，只能针对单个 URL 进行尽可能快的压力测试。

+ http_load

这个工具概念上和 ab 类似，也被设计为对 Web 服务器进行测试，但比 ab 要更加灵活。可以通过一个输入文件提供多个 URL，http_load 在这些 URL 中随机选择进行测试。也可以定制 http_load，使其按照时间比率进行测试，而不仅仅是测试最大请求处理能力。

+ JMeter

JMeter 是一个 Java 应用程序，可以加载其他应用并测试其性能。它虽然是设计用来测试 Web 应用的，但也可以用于测试其他诸如 FTP 服务器，或者通过 JDBC 进行数据库查询测试。

JMeter 比 ab 和 http_load 都要复杂得多。

### 单组件式测试工具

+ mysqlslap

mysqlslap 可以模拟服务器得负载，并输出计时信息。它包含在 MySQL 5.1 的发行包中。

+ MySQL Benchmark Suite（sql-bench）

MySQL 发行包中提供的一款基准测试套件，可以用于在不同数据库服务器上进行比较测试。

+ Super Smack

Super Smack 是一款用于 MySQL 和 PostgreSQL 的基准测试工具，可以提供压力测试和负载生产。

+ Database Test Suite

Database Test Suite 是由 OSDL（开源软件开发实验室，Open Source Development Labs）设计的，这是一款类似某些工业标准测试的测试工具集。

+ Percona's TPCC-MySQL Tool

一个类似 TPC-C 的基准测试工具集，其中有部分是专门为 MySQL 测试开发的。

+ sysbench

sysbench 是一款多线程系统压测工具。它可以根据影响数据库服务器性能的各种因素来评估系统的性能。
