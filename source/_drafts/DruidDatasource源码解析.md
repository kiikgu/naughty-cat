---
title: Druid源码剖析
tags:
---

Druid是阿里巴巴的一个开源项目，来源于阿里的一个SQL监控项目。Druid是一个非常优秀的数据库连接池，具有强大的监控特性，很好的扩展性，以及在阿里苛刻生产环境下优化出来的高性能。本文主要从代码层面分析Druid的原理。

<!--more-->

### DruidDataSource类继承图

![DruidDataSource继承图](/img/20171119/DruidDatasource_继承图.png)

DruidDataSource的继承接口中比较重要的几个接口: DataSource、ConnectionPoolDataSource、DruidAbstractDataSourceMBean、DruidDataSourceMBean。

#### DataSource

简单来说，DataSource就是数据源连接工厂，它是除DriverManager之外获取连接的又一方式。DataSource有三种类型的实现：

* 基本实现：创建一个标准的连接。
* 连接池实现：创建连接并添加到连接池，会使用中间层的连接池管理器。
* 分布式事务实现：创建连接，用于分布式事务处理，并添加到连接池；使用中间层的事务管理器和连接池管理器。

|主要方法|
|-------|
|Connection getConnection() throws SQLException|
|Connection getConnection(String username, String password) throws SQLException|

#### ConnectionPoolDataSource

ConnectionPoolDataSource是用来获取PooledConnection的工厂，PooledConnection是可重复利用的连接。

#### DruidAbstractDataSourceMBean

监控指标获取方法

#### DruidDataSourceMBean
监控指标获取方法



### 参考

[http://www.iteye.com/magazines/90#111](http://www.iteye.com/magazines/90#111)
