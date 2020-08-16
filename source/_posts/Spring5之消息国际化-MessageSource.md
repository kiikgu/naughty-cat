---
title: Spring5之消息国际化(MessageSource)
date: 2018-05-11 07:59:34
categories: [spring5学习]
tags: [spring,MessageSource,消息国际化]
toc: true
---

ApplicationContext继承了MessageSource接口，并提供了两个MessageSource接口的实现类：ResourceBundleMessageSource和ReloadableResourceBundleMessageSource，来实现消息国际化。
<!--more-->

#### 使用ResourceBundleMessageSource

**1. 配置messageSource bean**

![messageSource bean配置](/img/20180511/messageSource_bean配置.png)

上图所示，basenames指定消息资源文件路径，上图所示指定了3个资源文件，format.properties、exceptions.properties、windows.properties, 都位于classpath路径下的messagesource文件夹下。

**2. 消息格式**

![message 1](/img/20180511/message_format.png)

![message 2](/img/20180511/message_exception.png)

**3. 使用示例**

![无参](/img/20180511/message_use_without_param.png)

![有参](/img/20180511/message_use_param.png)

![国际化](/img/20180511/message_use_international.png)

#### ResourceBundleMessageSource
从上面的使用中可以总结ResourceBundleMessageSource的实现包含以下功能点：

* 根据指定Locale找到Resource文件
* 从Resource文件中找到指定code的字符串value值
* 根据参数进行字符串格式化，得到最终结果

##### 类继承结构

![ResourceBundleMessageSource类继承结构图](/img/20180511/bundle_class_inher.png)

MessageSource接口: 声明了三个获取格式化消息的方法。

![MessageSource接口](/img/20180511/messageSource_interface.png)

MessageSourceSupport:  为MessageSource的实现类提供功能支持，包含的方法在MessageSource实现中会用到。

##### AbstractMessageResource

![AbstractMessageSource类](/img/20180511/abstract_message_source.png)

getMessageInternal方法处理主要工作：

![getMessageInternal方法](/img/20180511/get_message_internal.png)

resolveCode方法根据code和locale从对应的Resource文件中搜索到相应的字符串，此时返回的是一个MessageFormat对象而不是字符串，是为了让子类对MessageFormat对象进行合理的缓存。

MessageFormat：格式化消息

##### resolveCode方法实现

![resolveCode方法](/img/20180511/resolve_code.png)

resolveCode先根据basename和locale找到ResourceBundle对象，再从ResourceBundle中搜索code对应的字符串。

ResourceBundle: 一个basename和一个Locale指定Resource对应一个ResourceBundle对象，如：

![ResourceBundle示例](/img/20180511/resource_bundle_format.png)

上图所示，有两个ResourceBundle对象，他们的basename相同，Locale不同，属于同一个Resource Bundle家族。
ResourceBundle类用来获取相应basename和Locale指定的Resource，它提供的多个静态方法getBundle实现了这一逻辑。

#### ReloadableResourceBundleMessageSource

ResourceBundleMessageSource通过bundleClassLoader加载ResourceBundle，Resource一旦被加载就会被永久缓存。而
ReloadableResourceBundleMessageSource基于Spring的Property加载机制，实现了Resource文件的重复加载，它可以根据文件更改时间和字符类型进行Resource文件的动态加载。

##### 类继承结构

![ReloadableResourceBundleMessageSource类继承结构](/img/20180511/ReloadableResourceBundle_class_inher.png)

##### resolveCode方法实现

![resolveCode方法](/img/20180511/reloadable_resolve_code.png)

**1. PropertiesHolder**

用来保存加载的Properties，文件修改时间，最后一次加载时间和一个重新加载锁。

**2. Resource文件搜索顺序**

caculateAllFilenames决定了文件搜索顺序

![resource搜索顺序](/img/20180511/resource_search_1.png)

![resource搜索顺序](/img/20180511/resource_search_2.png)

**3. Resource文件加载**

![resource加载](/img/20180511/resource_load1.png)

![resource加载](/img/20180511/resource_load2.png)

#### 总结和得到：

* 使用MessageResource机制实现消息的国际化
* 熟悉Spring MessageResource机制实现原理
* xxxSupport抽象 (一些公用的功能可以抽象xxxSupport类中)
* 学会一种实现文件修改立即生效的加载方法

#### 参考资料

[ https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-messagesource]( https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-messagesource)

[ https://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-java.util.Locale-java.lang.ClassLoader-java.util.ResourceBundle.Control-]( https://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-java.util.Locale-java.lang.ClassLoader-java.util.ResourceBundle.Control-)

Spring源码




