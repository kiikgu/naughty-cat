---
title: Spring5之Event剖析
date: 2018-05-20 19:18:33
categories: [spring5学习]
tags: [spring,EventListener,事件发布处理机制]
toc: true
---

ApplicationContext的实现类也实现了生命周期接口，如：LifeCycle和Closable，通常执行这些接口方法，如：close(), stop()，表示ApplicationContext生命周期的终结，此时需要销毁context中的所有bean并释放bean已经加载的资源。Spring抽象并实现了一套消息发布和订阅机制，很好地完成了这一工作。在调用生命周期相关方法时发布消息，当我们的代码需要在context关闭时释放资源，就可以订阅这些消息，进行相应的处理，很容易实现扩展。<br>

本章主要介绍Spring的event机制并深入剖析其实现。
<!--more-->

Spring event相关接口包括：ApplicationEvent，ApplicationListener，ApplicationEventPublisher, ApplicationEventPublisherAware。ApplicationEvent是对消息通知的抽象，表示消息事件；ApplicationEventListner是事件接收者。一个事件发布后，注册到ApplicationContext中的listener都能收到消息通知，并做处理。这是一个典型的观察者模式。 <br>

通常定义新的Event需要实现ApplicationEvent接口，Spring4.2之后提供了注解方式，使得Event对象和Spring Api解耦。

#### 标准事件
Spring提供了几种内部事件，在ApplicationContext的生命周期中触发。

**ContextRefreshEvent**

当ApplicationContext完成初始化或refresh方法调用之后，触发该事件。

* 完成初始化：所有Bean配置被加载，post-processor bean被检测和加载，singleton bean初始化完成
* refresh: 只要context没有关闭，就可以多次触发refresh；

![refresh事件触发 1](/img/20180520/event_context_refresh.png)

![refresh事件触发 2](/img/20180520/event_context_refresh2.png)

注意：GenericApplicationContext不支持多次"hot" refresh

**ContextStartedEvent和ContextStoppedEvent**

当context调用start和stop方法时，会发布ContextStartedEvent和ContextStoppedEvent事件。

![start和stop事件触发](/img/20180520/event_context_start_stop.png)

**ContextClosedEvent**

当context调用close方法时，发布ContextClosedEvent事件。

![close事件触发 1](/img/20180520/event_context_closed.png)

![close事件触发 2](/img/20180520/event_context_closed2.png)

**RequestHandledEvent**

当一次Http请求处理结束时，发布RequestHandledEvent事件。仅限于使用Spring DispatchServlet的程序。

![request handler事件触发](/img/20180520/event_context_request.png)

![request handler事件触发](/img/20180520/event_context_request2.png)


#### 自定义事件

spring event机制允许用户扩展新的事件及相应的处理，自定义事件需要实现ApplicationEvent接口，事件处理类需要实现ApplicationEventListener接口。

**自定义event**

![自定义事件示例](/img/20180520/event_custom_event.png)

**自定义event listener**

![自定义事件处理示例](/img/20180520/event_custom_listener.png)

**事件发布**

![事件发布示例](/img/20180520/event_custom_publish.png)

注入到EmailService的ApplicationEventPublisher是context本身，context实现了ApplicationEventPublisher接口。

#### event机制剖析

##### ApplicationListener接口

继承了java.util.EventListener接口，Spring3.0之后，可以指定listener关注的event类型。

##### ApplicationEventMulticaster

管理多个ApplicationListener，并向它们发布事件消息。

![ApplicationEventMulticaster接口](/img/20180520/event_multicaster1.png)

![ApplicationEventMulticaster接口](/img/20180520/event_multicaster2.png)

spring提供了实现类：SimpleApplicationEventMulticaster。

##### listener的检测

在Context启动过程中，类ApplicationListenerDetector用于检测ApplciationListener bean，并添加到ApplicationListener列表。 <br>

ApplicationListenerDetector实现了BeanPostProcessor接口。

![ApplicationListenerDetector](/img/20180520/event_listener_detector1.png)

![ApplicationListenerDetector](/img/20180520/event_listener_detector2.png)

#### 基于注解的event listener

##### @EventListener注解使用

**标注到任意方法上，即可实现事件监听**

![@EventListener使用示例](/img/20180520/event_annotation_use1.png)

**指定接收类型**

![指定接收类型](/img/20180520/event_annotation_use2.png)


**根据条件过滤事件**

![条件过滤](/img/20180520/event_annotation_use3.png)

##### @EventListener实现剖析

EventListerMethodProcessor负责检测@EventListener标注方法，并创建代理ApplicationListener对象，最后添加到context的ApplicationListener列表。具体实现步骤如下：

* 在所有singleton bean初始化后，遍历每个bean对象
* 检测到所有被@EventListener标注的方法
* 对每个@EventListener方法，创建一个ApplicationListener对象
* 把ApplicationListener对象加入context的ApplicationListener序列

**检测@EventListener**

![EventListnerMethodProcessor类继承结构](/img/20180520/event_listener_mothod_processor1.png)

EventListenerMethodProcessor实现了SmartInitializingSingleton接口，SmartInitializingSingleton接口在所有singleton bean被初始化后紧接着被调用，可以作为InitializingBean接口的替代方案。

SmartInitializingSingleton接口及调用点

![SmartInitializingSingleton接口](/img/20180520/event_SmartInitializingSingleton1.png)

![SmartInitializingSingleton接口调用点](/img/20180520/event_SmartInitializingSingleton2.png)

**EventListenerMethodProcessor调用**

![EventListenerMethodProcessor调用](/img/20180520/event_listener_mothod_processor_invoke.png)

![EventListenerMethodProcessor调用](/img/20180520/event_listener_mothod_processor_invoke2.png)

当创建对象为ApplicationListenerAdapter时会进行初始化，添加EL表达式处理类EventExpressionEvaluator，用来处理@EventListener中的condition参数

**ApplicationListnerAdapter**

* 类继承图

![ApplicationListnerAdapter类继承图](/img/20180520/event_adapter_hier.png)


* 处理Event事件

![处理event事件](/img/20180520/event_adapter_process1.png)

![处理event事件](/img/20180520/event_adapter_process2.png)

上图所示，rocessEvent方法包含以下处理过程：
	
	1. 处理event参数
	2. 根据EL表达式判断是否处理消息
	3. 调用event处理方法
	4. 处理方法返回


![参数处理](/img/20180520/event_adapter_process_param.png)

![条件处理](/img/20180520/event_adapter_process_condition.png)

![调用event处理方法](/img/20180520/event_adapter_process_method.png)

方法调用中使用了桥接方法，关于桥接方法的介绍参考：JSL桥接方法

![调用event处理方法](/img/20180520/event_adapter_process_return.png)

上图可以看到，如果方法产生新的事件，那么会继续发布新事件

@EventListner注解实现采用了适配器模式，每个@EventListener注解方法都会被适配成一个ApplicationListnerAdapter对象。

#### 事件异步处理

添加@Async标签实现事件的异步处理。

![事件异步处理](/img/20180520/event_listener_process_async.png)

使用异步事件处理需要注意：

* event listener抛出的Exception传播不到caller
* event listener不能自动发布返回的事件消息，只能通过注入ApplicationEventPublisher手动发布

#### listener排序

通过@Order调整listener的调用顺序

![listener排序](/img/20180520/event_listener_process_order.png)

#### 泛型事件

![泛型事件](/img/20180520/event_generic1.png)

![泛型事件](/img/20180520/event_generic2.png)

* ResolableType: 封装了java.lang.reflect.Type对象，提供了一些便利方法，访问类型信息。
* ResolvableTypeProvider: 返回一个RelovableType，用来在运行时判断对象是否是泛型类型，因为Java在运行时会擦出泛型信息。

#### 观察者模式

spring事件处理机制实现是典型的观察者模式，观察者模式可参考此博客内容。

#### 总结和得到

* 熟悉了Spring事件处理机制及实现原理
* 复习观察者模式
* 可参考@EventListener注解检测和解析方式实现自定义注解的检测和解析
* 可参考@EventListener EL表达式的解析实现添加EL表达式功能

#### 参考资料

[https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-events-annotation
https://www.jianshu.com/p/250030ea9b28
https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html#bridgeMethods](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-functionality-events-annotation)

[https://www.jianshu.com/p/250030ea9b28](https://www.jianshu.com/p/250030ea9b28)

[https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html#bridgeMethods](https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html#bridgeMethods)

spring源码





