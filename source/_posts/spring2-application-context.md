---
title: Spring探秘2：ApplicationContext启动流程
tags:
  - Spring
  - 设计模式
abbrlink: 1219753109
date: 2019-12-12 14:28:46
---

&#160;&#160;&#160;&#160; `ApplicationContext`是Spring框架中最基础的接口之一，可以认为其实现类就是一个Spring的环境（容器），而一个简单的Spring应用的启动过程就是一个`ApplicationContext`的实现类的实例化过程，是研究Spring源码的很好的切入点。这里研究的实现类是`AnnotationConfigApplicationContext`，运行的代码为前文{% post_link spring1-build %}中的测试代码，启动类的代码如下：
```java
public class DemoApp {
  public static void main(String[] args) {
        ApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

        DemoComponent component = context.getBean(DemoComponent.class);
        component.foo();

    }
}
```

<!--more-->

> 要区分的一个概念就是“容器”，本文中提到的容器都是指Spring框架的容器，即管理Service，Dao以及Component这些Bean的一个“环境”。与SpingMVC的容器不同，SpringMVC的容器是管理Controller的环境。与Tomcat这些Web容器就更不一样了。

## 相关的类与接口

### 继承结构

&#160;&#160;&#160;&#160; `AnnotationConfigApplicationContext`这个类从类名就可以看出，它是一个通过注解来配置的Spring容器，实例化该容器时，可以传入一个配置类，上面代码中的`AppConfig`就是我自定义的一个配置类，其中唯一的配置项就是通过`@ComponentScan`注解配置了扫描的包。
&#160;&#160;&#160;&#160;`AnnotationConfigApplicationContext`这个类的继承结构如下图（图中只包含了与启动容器关系较大的类和接口，实际的继承层次要复杂得多）：

{% asset_img applicationcontext.svg applicationcontext %}

从中可以看出
1. `ApplicationContext`最终还是要实现`BeanFactory`，而这个父接口的含义可以参考{% post_link spring1-BFandFB 前一篇文章 %}。因此可以说`ApplicationContext`是`BeanFactory`功能上的增强。
2. `AnnotationConfigApplicationContext`的直接父类`GenericApplicationContext`其实还有其他的一些子类，如`GenericXmlApplicationContext`等，分别用于表示不同配置方式的容器。

### 启动容器时涉及到的类

&#160;&#160;&#160;&#160; 容器的启动过程就是一个`ApplicationContext`的实例化过程，本文研究的容器实现类为`AnnotationConfigApplicationContext`，启动的过程中会涉及到许多其他的类，包括容器中的实例对象的实例化，向容器中注册的一些初始的处理器等。这里通过类图的形式总结了部分类以及这些类之间的关系。

{% asset_img startclass.svg startclass %}

### 各类的简介

自顶向下地简单介绍相关的类：
* `BeanDefinition`: 用于描述一个Bean实例的信息；根据Bean的来源不同有具体的实现类，如`AnnotatedGenericBeanDefinition`，`RootBeanDefinition`，`ScannedGenericBeanDefinition`等。
* `AbstractApplicationContext`: 是`ApplicationContetxt`接口的抽象实现类，该类没有指定容器的配置类型，只实现了容器的一些通用功能，这些功能增强了简单的`BeanFactory`。与简单的`BeanFactory`相比，`ApplicationContext`应该能够主动探测到容器内的一些特殊的bean。因此该类自动注册了一些`BeanFactoryPostProcessor`，`BeanPostProcessor`，`ApplicationListener`，在一些特殊的时期能够自动调用这些对象的功能。
* `BeanFactoryPostProcessor`: 这就是一种`ApplicationContext`能够主动调用的特殊的Bean，从类名不难猜到，实现了该接口的Bean会在Bean工厂初始化完成后被调用（称为后置处理器）。因此我们可以通过实现该接口来插手Bean工厂初始化过程，从而扩展Spring的功能。值得注意的是，该接口的实现类只能去处理、修改bean definitions，而不应该对Bean实例作任何处理，因为调用时，bean都还没实例化。
* `BeanDefinitionRegistryPostProcessor`，它是`BeanFactoryPostProcessor`的子接口，即除了实现`BeanFactoryPostProcessor`的功能之外，还能够实现动态地向容器中添加`BeanDefinition`的功能。该功能会在一般的`BeanFactoryPostProcessor`之前被调用。
* `PostProcessorRegistrationDelegate`：`AbstractApplicationContext`将调用后置处理器的功能委派给该类，应该只是为了减少每个类的代码量？
* `BeanDefinitionRegistry`：registry意为“注册处”，所以这个接口也就是表示`BeanDefinition`注册的地方，一般都是由Bean工厂实现，用于管理`BeanDefinition`，包括注册、移除、获取等。对`BeanDefinition`有管理需要的地方就可以调用该接口的方法。
* `GenericApplicationContext`: 是`AbstractApplicationContext`的子类，主要维护了一个`DefaultListableBeanFactory`实例，通过该实例实现了`BeanFactory`接口（装饰者模式）。另外也实现了`BeanDefinitionRegistry`接口，给有需要的类注册`BeanDefinition`。
* `DefaultListableBeanFactory`：这就是Spring内`BeanFactory`接口和`BeanDefinitionRegistry`的一个默认实现。
* `AnnotationConfigApplicationContext`: 是`GenericApplicationContext`的子类，是一个独立的应用上下文（容器），构造时可以接收多个组件类作为参数（一般是配置类）。
* `ClassPathBeanDefinitionScanner`: 是`AnnotationConfigApplicationContext`中一个重要的工具，用于扫描classpath上所有的Bean，并将相应的`BeanDefifnition`注册到给定的`BeanDefinitionRegistry`中。（一般registry就是一个`BeanFactory`或者是`ApplicationContext`，即当前的Spring容器）
* `AnnotatedBeanDefinitionReader`：是`AnnotationConfigApplicationContext`中另一个重要的工具，也是用于接收一个Bean类，转换为`BeanDefifnition`并将其注册到registry中，但是不同点在于它只用于读取显式注册的类，即与`ClassPathBeanDefinitionScanner`相比没有扫描classpath的功能。




## 启动流程原理分析

`AnnotationConfigApplicationContext`的构造方法的源码如下：
```java
/**
	 * Create a new AnnotationConfigApplicationContext that needs to be populated
	 * through {@link #register} calls and then manually {@linkplain #refresh refreshed}.
	 */
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
/**
	 * Create a new AnnotationConfigApplicationContext, deriving bean definitions
	 * from the given component classes and automatically refreshing the context.
	 * @param componentClasses one or more component classes &mdash; for example,
	 * {@link Configuration @Configuration} classes
	 */
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    register(componentClasses);
    refresh();
}
```

&#160;&#160;&#160;&#160; 下图总结了构造方法的调用层次：

{% asset_img constructor.png constructor %}

### 构造父类GenericApplicationContext

* 再递归地构造父类`AbstractApplicationContext`，该父类中的`BeanFactoryPostProcessor`的list就是在此时实例化的
* 实例化beanFactory属性为DefaultListableBeanFactory。


> 工厂模式：`AbstractApplicationContext`类定义了一个抽象方法`getBeanFactory()`，通过该方法实现了`BeanFactory`接口。该方法就是需要子类去实现的工厂方法，不同的子类可能有不同的`BeanFactory`实现。

### 实例化reader属性

* 设置reader中的registry，conditionEvaluator属性
* 调用 `AnnotationConfigUtils.registerAnnotationConfigProcessors()` 向容器注册一些后置处理器的`BeanDefinition`（并且其实现类均为`RootBeanDefinition`），包括：
  * `ConfigurationClassPostProcessor`：是一个`BeanDefinitionRegistryPostProcessor`实现类，用于处理配置类
  * `AutowiredAnnotationBeanPostProcessor`：用于处理自动注入的类
  * `CommonAnnotationBeanPostProcessor`：支持JSR-250的注解
  * `PersistenceAnnotationBeanPostProcessor`：支持JPA的注解
  * `EventListenerMethodProcessor`
  * `DefaultEventListenerFactory`

* 此时，reader的实例化已经完成，程序运行到这里（指`AnnotatedBeanDefinitionReader`构造方法的最后一行）时，Bean工厂中应该已经存在一些`BeanDefinition`：

{% asset_img bdmap1.png bdmap1 %}

### 实例化scanner属性

* 该实例化过程比较简单，只是在scanner的扫描过滤器中添加了`@Component`这个注解。

### 注册构造方法的参数传入的组件类

* 调用`reader.register()`，追踪调用，最终的逻辑是在`doRegisterBean()`方法实现的

### 刷新 refresh()

* 刷新容器是容器启动的最重要的一个阶段，有很多的工作包括准备Bean工厂、调用`BeanFactoryPostProcessors`、注册`BeanPostProcessors`等等，需要另外进行详细的分析。
