---
title: Java笔记1：Object类源码
tags:
  - Java
abbrlink: 3140146257
date: 2019-07-02 00:00:00
---


Object类是Java类层次结构的根节点，定义了一些最抽象的方法。

## registerNatives()

```java
private static native void registerNatives();

static {
    registerNatives();
}
```
&#160; &#160; &#160; &#160;是一个本地方法，并且该方法在静态代码块中，因此所有对象在创建时，都会先调用该方法。该方法的主要作用就是将其他语言实现的（主要是C/C++）一些环境相关的方法映射到Java中的本地方法，实现方法名的解耦。

<!--more-->

## getClass()

```java
public final native Class<?> getClass();
```

&#160; &#160; &#160; &#160;该方法也是一个本地方法，返回该对象的**运行时类型**，不可重写。返回的Class对象的具体类型为`Class<? extends X>`，其中X是变量调用该方法的变量的被擦除的静态类型。

## hashCode()

```java
public native int hashCode();
```
&#160; &#160; &#160; &#160;返回值为对象的哈希值，用于支持哈希表。对于该方法的返回值，有一些通用的约定：

* 在同一个Java应用中调用同一个对象的`hashCode()`函数，只要`equals()`方法中比较的信息没有改变，那么无论调用多少次，返回值都应该是相等的。
* 如果根据`equals()`方法，两个对象是相等的，那么这两个对象的哈希值也应该是相等的。
* 虽然并没有要求两个不相等的对象（根据`equals()`方法）的哈希值必须不等，但是程序应该保证哈希值尽量稀疏以提高哈希表的性能。

## equals()

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

&#160; &#160; &#160; &#160;用于比较另一个对象是否与当前对象“相等”，可由子类重写。该方法应具有自反性，传递性，对称性，幂等性。在`Object`类中的默认实现比较的内存地址，即只有当两个变量指向同一个对象时该方法才返回`true`，在实际应用时通常需要重写该方法以保证逻辑上的正确性。

## clone()

```java
protected native Object clone() throws CloneNotSupportedException;
```

&#160; &#160; &#160; &#160;创建并返回该对象的拷贝，“拷贝”的准确含义与该对象的类型有关。通常来说，该方法应该使得`x.clone() != x`，`x.clone().getClass() == x.getClass()`，`x.clone().equals(x)`的结果均为`true`，但是并不是强制的要求。
&#160; &#160; &#160; &#160;该方法对对象实现的是“浅拷贝(shallow copy)”，而非“深拷贝（deep copy）”
* 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
* 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

&#160; &#160; &#160; &#160;对于一个具体类，如果想要调用其`clone()`方法，该类需要实现`Clonable`接口，否则会抛出`CloneNotSupportedException`异常。

## toString()

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

&#160; &#160; &#160; &#160;发挥该对象的字符串表示，通常该字符串是对该对象的一个简明、易读的总结，所有的子类都应该重写该方法。`Object`类中的`toString()`方法的返回值格式是：类型名@哈希值，其中哈希值是十六进制表示。

## notify(), notyfyAll()

```java
public final native void notify();
public final native void notifyAll();
```

&#160; &#160; &#160; &#160;`notify()`随机唤醒一个等待该对象的线程，`notifyAll()`唤醒全部等待该对象的线程。线程如果想要调用这两个方法，必须拥有该对象的监视器，线程可以通过以下几种方法拥有对象的监视器：
* 执行对象的synchronized实例方法
* 执行一个synchronized代码块，用该对象作为同步对象
* 如果是一个Class对象，那么只要执行一个该类的同步静态方法

## wait(long timeout), wait(long timeout, int nanos), wait()

```java
public final native void wait(long timeout) throws InterruptedException;

public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}

public final void wait() throws InterruptedException {
    wait(0);
}
```

&#160; &#160; &#160; &#160;让线程进入等待状态，直到其他线程调用了该对象的`notify()`方法或者`nitifyAll()`方法或者经过了一定的时长。

## finalize()

```java
protected void finalize() throws Throwable { }
```

&#160; &#160; &#160; &#160;当垃圾回收器（Garbage Collector）确定不再有对该对象的引用时，由垃圾回收器在对象上调用该方法。该方法只会被调用一次。