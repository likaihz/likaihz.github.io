---
title: Spring Bean的生命周期
tags:
  - Spring
  - 源码
abbrlink: 2299005582
date: 2020-01-18 22:58:57
---

## 什么是Bean的生命周期

&#160; &#160; &#160; &#160;用最简单的说法，Bean的生命周期就是一个Bean实例从产生到灭亡所经历的阶段，包括每个阶段的状态和阶段之间转移的时机。本文主要讨论Spring框架中最常见的单例Bean的生命周期。

<!--more-->
&#160; &#160; &#160; &#160;Spring中，单例的Bean在容器启动过程中就已经实例化并放入容器了（除了懒加载的），从源码上来看，在容器启动时，首次实例化单例Bean的调用路径是这样的：
AbstractApplicationContext # refresh()
-> AbstractApplicationContext # finishBeanFactoryInitialization()
-> DefaultListableBeanFactory # preInstantiateSingletons()
-> DefaultListableBeanFactory # getBean()
-> AbstractBeanFactory # doGetBean()
-> AbstractAutowireCapableBeanFactory # createBean()
-> AbstractAutowireCapableBeanFactory # doCreateBean()
最终主要的实例化逻辑就在doCreateBean()这个方法中，Bean的生命周期也从这里开始。

### Spring Bean的生命周期有那些阶段

&#160;&#160;&#160;&#160;总的来说，Bean的生命周期分为**四个阶段**，前三个阶段的开始分别对应在`doCreateBean()`方法中的一个方法调用：
1. 实例化阶段（Instantiation）：createBeanInstance()
2. 属性赋值阶段（Populate）：populateBean()
3. 初始化阶段（Initialization）：initializeBean()
4. 销毁阶段（Destruction）：在容器关闭时调用。


## 了解Bean的生命周期有什么用

&#160;&#160;&#160;&#160;Spring提供了许多的扩展点，其中有一些是在创建Bean的过程中起作用的，在其生命周期的不同阶段都会有不同的接口起作用，了解Bean的生命周期就能够对这些扩展点的生效的阶段有一个更准确的理解。

### BeanPostProcessor接口

&#160;&#160;&#160;&#160;在Spring框架中最常见的一个扩展点就是BeanPostProcessor接口，它还有一个很重要的子接口InstantiationAwareBeanPostProcessor，这两个接口的实现类能够在其他的Bean的生命周期的不同阶段起作用。它们起作用的时机如下图：

{% asset_img bean-lifecycle.svg bean-lifecycle %}

### Aware接口

&#160;&#160;&#160;&#160;Aware接口顾名思义，就是能够“获知”某些信息的接口。
&#160;&#160;&#160;&#160;Aware接口有很多子接口，是要Bean自己去实现的，实现了Aware接口的Bean在创建过程中根据实现的不同子接口，会在不同的阶段调用其接口方法，向其通知对应的信息。**所有的Aware接口都是在Bean初始化之前调用的**。
&#160;&#160;&#160;&#160;Aware接口可以分为两类，这两类Aware接口的方法是分开调用的：

* 第一类通知的信息是跟Bean有关的，如：
  * BeanNameAware
  * BeanClassLoaderAware
  * BeanFactoryAware
* 第二类通知的信息跟容器有关，如：
  * EnvironmentAware
  * ApplicationContextAware

&#160;&#160;&#160;&#160;这些接口的名称很直观，实现了名为XXXAware的接口，容器就会在创建Bean的时候将这个XXX通过调用接口方法的方式传递给Bean实例。而这些接口方法的调用是在`initializeBean()`发生的，具体代码如下（无关代码已经隐去）：

{% asset_img aware.png aware %}

### 生命周期接口

&#160;&#160;&#160;&#160;默认情况下，初始化阶段除了调用上面提到的Aware接口方法以外就没有什么其他的逻辑了；销毁阶段也是几乎没有做什么。Spring提供了途径让开发者能够在这两个阶段执行一些自定义的逻辑：
* `InitializingBean`接口：实现该接口的Bean在初始化阶段会调用其接口方法。
* `DisposableBean`接口：实现该接口的Bean在销毁阶段会调用其接口方法。