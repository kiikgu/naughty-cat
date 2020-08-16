---
title: 记一次Spring日志问题查找
date: 2017-10-21 16:54:00
categories: [问题排查]
tags: [问题排查, logback]
toc: true
---

本地启动netty时，控制台一直打印DEBUG级别的日志，但是查看logback的level级别是ERROR级别，自己对log这块儿也不是很了解。

<!--more-->

### 现象
本地启动netty时，控制台一直打印DEBUG级别的日志

```
16:56:29.147 [main] DEBUG o.s.w.c.s.XmlWebApplicationContext - Bean factory for Root WebApplicationContext: org.springframework.beans.factory.support.DefaultListableBeanFactory@656672fb: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,octopus-demeter,dubbo,com.alibaba.dubbo.config.ConsumerConfig,dubbo2,org.springframework.aop.config.internalAutoProxyCreator,springContextHolder,cn.fraudmetrix.octopus.demeter.dal.aop.AroundDaoAspect#0,log-filter,dataSource,stat-filter,wall-filter,sqlSessionFactory,org.mybatis.spring.mapper.MapperScannerConfigurer#0,txManager,org.springframework.transaction.annotation.AnnotationTransactionAttributeSource#0,org.springframework.transaction.interceptor.TransactionInterceptor#0,org.springframework.transaction.config.internalTransactionAdvisor,codeServiceImp,getCodeInfoServiceImp,getDateTypeServiceImp,getDistrictServiceImp,getMobileCategoryImpl,mobileServiceImp,telephoneServiceImp,baseInfoService,districtService,getResultService,logsService,mobileCategoryService,sessionService,mobileInfoServiceImpl,cn.fraudmetrix.octopus.demeter.client.service.MobileInfoService,cn.fraudmetrix.octopus.demeter.client.service.GetCodeInfoService,cn.fraudmetrix.octopus.demeter.client.service.GetDateTypeService,cn.fraudmetrix.octopus.demeter.client.service.GetDistrictService,cn.fraudmetrix.octopus.demeter.client.service.GetMobileCategoryService,logAspect,myCut,org.springframework.aop.aspectj.AspectJPointcutAdvisor#0,org.springframework.aop.aspectj.AspectJPointcutAdvisor#1,org.springframework.aop.aspectj.AspectJPointcutAdvisor#2,redisObject,shutterConfig,redisLockService,redisService,org.springframework.cache.annotation.AnnotationCacheOperationSource#0,org.springframework.cache.interceptor.CacheInterceptor#0,org.springframework.cache.config.internalCacheAdvisor,cacheManager,redisPool,redisClient,cn.fraudmetrix.octopus.demeter.engine.config.EnvironmentConfig#0,disconf,com.baidu.disconf.client.DisconfStaticScanner#0,com.baidu.disconf.client.DisconfDynamicScanner#0,org.springframework.context.support.PropertySourcesPlaceholderConfigurer#0,registry,influxDB,influxDBReporter,grafanaTriggerReporter,cn.fraudmetrix.metrics.RegistryReporter#0,serverInfoProvider,redisMeasurement,org.springframework.aop.aspectj.AspectJPointcutAdvisor#3,org.springframework.aop.aspectj.AspectJPointcutAdvisor#4,org.springframework.aop.aspectj.AspectJPointcutAdvisor#5,redisMeasurementPointcut]; root of factory hierarchy
16:56:29.199 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Registering scope 'request' with implementation [org.springframework.web.context.request.RequestScope@26c2f767]
16:56:29.205 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Registering scope 'session' with implementation [org.springframework.web.context.request.SessionScope@bc88295]
16:56:29.205 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Registering scope 'globalSession' with implementation [org.springframework.web.context.request.SessionScope@341df373]
16:56:29.206 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Registering scope 'application' with implementation [org.springframework.web.context.support.ServletContextScope@51f009ef]
16:56:29.376 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
16:56:29.377 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Creating instance of bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
16:56:29.444 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Eagerly caching bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor' to allow for resolving potential circular references
16:56:29.448 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Finished creating instance of bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
16:56:29.448 [main] DEBUG o.s.b.f.s.DefaultListableBeanFactory - Creating shared instance of singleton bean 'com.baidu.disconf.client.DisconfStaticScanner#0'
```

logback.xml配置

```
<logger name="org.springframework" level="ERROR" addtivity="true"/>
<logger name="org.apache" level="ERROR" addtivity="true"/>
<logger name="org.mybatis" level="ERROR" addtivity="true"/>

```

### 排查分析过程

#### 1. 最开始怀疑是jar包冲突导致日志绑定不对，

**找一条日志记录打断点调试**

```
17:07:16.824 [main] DEBUG o.s.b.f.x.BeanDefinitionParserDelegate - Neither XML 'id' nor 'name' specified - using generated bean name [cn.fraudmetrix.octopus.demeter.dal.aop.AroundDaoAspect#0]

```

进入BeanDefinitionParserDelegate类，找到这条日志记录，打断点
![日志断点](/img/20171021/20171021-log-debug001.png)

![日志断点](/img/20171021/20171021-log-debug002.png)

然而，从log类型看不出来个什么鬼

#### 2. 查看logback配置是怎么加载

![日志断点](/img/20171021/20171021-log-debug003.png)

web.xml中可以看出，LogbackConfigListener配置在ContextLoaderListener之后，所以在Spring Context加载完成之后才会加载logback配置，所以logback配置没有生效。

调整新姿势后，日志正常打印，烦银的debug记录不见了

### 日志的初始化过程

@TODO


