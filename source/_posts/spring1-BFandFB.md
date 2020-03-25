---
title: Spring探秘1：BeanFactory与FactoryBean
tags:
  - Spring
  - 设计模式
abbrlink: 1621531412
date: 2019-12-11 14:43:48
---

&#160; &#160; &#160; &#160; `BeanFactory`和`FactoryBean`都是Spring框架中重要的接口，名字很像，但是功能上是有很大的不同的。

## 先上结论

### 相同点

* 都是接口
* 都没有父接口，是Spring框架中比较基础的接口

### 不同点

* `BeanFactory`正如其接口名的暗示，是一个工厂类，用于管理（生成）Bean。也可以认为其实现类就是一个IOC容器。
* `FactoryBean`是一个Bean, 而该Bean的功能是控制其他Bean实例化的逻辑。
* 因此，`FactoryBean`类型的对象也是由`BeanFactory`管理的

<!--more-->
## BeanFactory

* BeanFactory定义了IOC容器的最基本形式，并提供了IOC容器应遵守的的最基本的接口，也就是Spring IOC容器所遵守的最底层和最基本的编程规范。
* 它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。
* 在Spring源码中，BeanFactory只是个接口，并不是IOC容器的具体实现。但是Spring框架给出了很多种实现，如 DefaultListableBeanFactory，XmlBeanFactory， **ApplicationContext**等，都是附加了某种功能的实现。
* 该接口的方法如下，可以看出该接口实现的主要功能就是从IOC容器中获取Bean：
{% asset_img BeanFactory.png BeanFactory %}


## FactoryBean

&#160; &#160; &#160; &#160;一般情况下，Spring通过反射机制利用<bean>标签的class属性指定实现类实例化Bean，但是在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>标签中提供大量的配置信息。配置方式的灵活性是受限的，这时采用代码的方式可能会得到一个简单的方案。
&#160; &#160; &#160; &#160;另外还有一些情况下，要使用第三方库/框架提供的Bean时，这些Bean的实例化过程不应该由使用者插手，而Spring框架本身又无从得知这些第三方Bean的实例化逻辑，因此需要某种方式将Bean的实例化逻辑引入到Spring框架中。
&#160; &#160; &#160; &#160;Spring为此提供了一个`org.springframework.bean.factory.FactoryBean`的工厂类接口，用户/第三方库设计者可以通过实现该接口定制实例化Bean的逻辑。
&#160; &#160; &#160; &#160;实现 `FactoryBean` 接口需要实现三个方法，分别是`isSingleton()`， `getObject()`， `getObjectType()`，其中 `getObject()` 方法返回的就是它产生的Bean实例。
&#160; &#160; &#160; &#160;FactoryBean的设计体现了“**装饰者模式**”的设计模式，其一个典型的实现是是Mybaits中的`SqlSessionFactory`类。

### FactoryBean生成的Bean的名字

&#160; &#160; &#160; &#160;对于实现了`FactoryBean`接口的Bean，Spring框架对其命名稍有不同。即代码中为实现了`FactoryBean`接口的Bean指定的名字将会命名给该`FactoryBean`生产出的Bean，而这个`FactoryBean`实例的Bean name是在指定的名字前加一个"&"符号。

### 对Bean name的实验

1. 创建一个Bean类Foo
{% asset_img exp1.png exp1 %}
2. 创建一个FactoryBean的实现类CustomFactoryBean，其中getObject()方法就返回一个Foo实例，并通过@Component实例将该类交给IOC容器管理
{% asset_img exp2.png exp2 %}
3. 在启动类中，按名称获取该工厂Bean，并执行其方法。但是却有报错。**说明，在IOC容器中，名为"customFactoryBean"的Bean并不是CustomFactoryBean类型的。**
{% asset_img exp3.png exp3 %}
{% asset_img exp3.2.png exp3.2 %}
4. 那究竟是什么类型？在启动类中用以下代码探究名为"customFactoryBean"的Bean到底是什么类型
{% asset_img exp4.png exp4 %}
输出为：`The class of bean is: class cn.litiezhu.explorespring.dao.Foo`。是在`CustomFactoryBean.getObject()`方法的返回类型。
5. 通过以下代码探索CustomFactoryBean类型的Bean在容器中的名字
{% asset_img exp5.png exp5 %}
输出为：`&customFactoryBean`
6. 那么如果不给CustomFactoryBean指定名字呢？通过实验会发现，结果还是一样的，即将`CustomFactoryBean`的默认名字`customFactoryBean`命名给了其生产的`Foo`类型的Bean。


