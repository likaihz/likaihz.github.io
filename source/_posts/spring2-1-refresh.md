---
title: Spring探秘2.1：容器启动中的refresh方法
tags:
  - Spring
  - 设计模式
abbrlink: 2300181694
date: 2019-12-14 11:16:01
---

&#160;&#160;&#160;&#160;在前一篇文章{% post_link spring2-application-context %}中提到了Spring容器启动的最后一步是refresh，即配置的刷新，这是容器启动过程中一个核心的步骤，实现了启动容器的主要的功能。本文会简单介绍一下`AbstractApplicationContext#refresh()`方法的流程，其中涉及到的比较复杂的过程还需要另外详细分析。首先其源码如下：

<!--more-->

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 让容器准备好刷新。
        prepareRefresh();

        // 通知子类刷新其内部的Bean工厂
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // 为容器准备好Bean工厂
        prepareBeanFactory(beanFactory);

        try {
            // 目前是一个空方法
            postProcessBeanFactory(beanFactory);

            // 调用容器中已注册为Bean的所有BeanFactoryPostProcessor
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册所有的BeanPostProcessor（拦截Bean创建过程）
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

## prepareRefresh()

* 让容器准备好刷新，设置启动时间和活动标识
* 初始化property源，验证环境中的property是否合法（即判断其可解析）

## obtainFreshBeanFactory()

&#160;&#160;&#160;&#160;该方法是用于从子类中获取实际的Bean工厂对象。

* 先调用refreshBeanFactory()通知子类刷新Bean工厂
  * 在GenericApplicationContext#refreshBeanFactory()中用了一个CAS操作将容器的refreshed属性置为true
* 再调用抽象方法getBeanFactory()获取Bean工厂，**这个抽象方法就是留给子类实现的工厂方法**

> 因此这里就是一个工厂模式的实例

## prepareBeanFactory()

&#160;&#160;&#160;&#160;该方法是在获取到了容器中的Bean工厂后，配置其中的一些工具与属性。

* 设置类加载器
* 设置Bean表达式解析器，bean初始化完成后填充属性时会用到
* 添加一个PropertyEditorRegistrar，属性编辑器注册器，用于将属性编辑器(PropertyEditor)注册到容器中
* 添加一个BeanPostProcessor (ApplicationContextAwareProcessor)，其功能是为各种实现了Aware接口的bean注入其需要的容器中的信息
* 设置各种Aware实现类为忽略自动装配
* 设置自动装配的类及自动装配的对象
* 根据需要添加动态织入的功能
* 注册各种默认的环境bean（Environment, SystemProperty, SystemEnvironment）

## postProcessBeanFactory()

&#160;&#160;&#160;&#160;该方法在框架中是一个空方法，是预留给子类去实现的，用于在Bean工厂准备好后的一些处理工作。

## invokeBeanFactoryPostProcessors()

&#160;&#160;&#160;&#160;该方法是刷新过程中**最核心**的一个方法，调用了容器中的BeanFactoryPostProcessor（即Bean工厂后置处理器），这是Spring框架向外暴露的一个扩展点，开发者可以通过实现该接口来对Bean工厂进行一些增强。这个方法的流程简单来说是这样：
* 先找到容器中所有的BeanDefinitionRegistryPostProcessor接口（它是BeanFactoryPostProcessor的子接口）的实现类，按顺序调用其postProcessBeanDefinitionRegistry()方法
* 再调用这些BeanDefinitionRegistryPostProcessor的postProcessBeanFacory()方法
* 最后再调用普通的BeanFactoryPostProcessor的postProcessBeanFactory()方法

具体的流程还需要单独再进行分析。

&#160;&#160;&#160;&#160;在默认情况下，此时容器中只有一个ConfigurationClassPostProcessor，它实现了BeanDefinitionRegistryPostProcessor接口。顾名思义该类的功能是在Bean工厂准备好后，处理容器中处理配置类。它的两个主要功能如下：

* postProcessBeanDefinitionRegistry()方法：找到所有的配置类，并解析这些配置类（这里所说的配置类是广义上的配置类，只要有相关的配置注解的类都会被解析）。主要处理了以下的注解：
  * @PropertySource
  * @ComponentScan：扫描了该注解指定的包，并将包内所有@Component注解的类都生成BeanDefinition并注册到容器中
  * @Import: 根据Import注解传入的类不同（ImportSelector, ImportBeanDefinitionRegistar, 其他）有不同的处理流程，处理结束后该注解引入的类的BeanDefinition应该已经注册到容器中了
  * @ImportResource
  * 配置类中的@Bean方法
  * 接口中的默认方法
  * 如果配置类有父类，还要处理其父类
* postProcessBeanFactory()方法：对于有@Configuration注解的配置类的BeanDefinition，会将他们替换为CGLib增强的子类。

> 这里要为什么要做CGLib代理？看了一些源码可知道，最根本的原因是为了增强配置类中的@Bean方法，使得即使多次调用这些方法也能够得到同一个Bean对象，实现Bean的单例模式。

ConfigurationClassPostProcessor是Spring框架中**最重要**的一个后置处理器，实现以上功能的原理以及具体的流程也需要单独分析。

## registerBeanPostProcessor()

&#160;&#160;&#160;&#160;会将所有实现了BeanPostProcessor接口的Bean注册到容器中

## initMessageSource()

&#160;&#160;&#160;&#160;在容器中初始化消息源

## initApplicationEventMulticaster()

&#160;&#160;&#160;&#160;初始化事件广播器

## onRefresh()

&#160;&#160;&#160;&#160;空方法，也是为子类预留的，令子类能够通过重写该方法来初始化一些特殊的Bean。

## registerListeners()

&#160;&#160;&#160;&#160;将实现了ApplicationListener接口的Bean注册到容器中作为监听器

## finishBeanFactoryInitialization()

&#160;&#160;&#160;&#160;完成Bean工厂的初始化，生成所有单例Bean。即实例化Bean就是主要在该方法中进行的。

## finishRefresh()：

&#160;&#160;&#160;&#160;完成容器的刷新过程

* 初始化容器的生命周期处理器并通知其容器正在刷新
* 发布容器刷新完成的事件