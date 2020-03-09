---
title: Java笔记4：Map接口
tags:
  - Java
abbrlink: 2366166279
date: 2019-07-05 19:46:02
---

## Map接口

&#160;&#160;&#160;&#160;`Map`用于保存键值对（key-value），其中key不允许重复。Map接口的继承树如下：

{% asset_img map.png map %}

<!--more-->

可以看到，`Map`的子类层次与`Set`很像，`Set`接口下有`HashSet`，`LinkedHashSet`，`SorterdSet`，`TreeSet`，`EnumSet`；相应的，`Map`接口下有`HashMap`，`LinkedHashMap`，`SortedMap`，`TreeMap`，`EnumMap`。`Map`的这些实现类中key集的存储形式和对应的`Set`中的元素的存储形式完全相同。

> 实际上，从源码看，Java是先实现了`Map`，然后通过包装一个所有value都为一个空对象的`Map`就实现了`Set`

## Java8改进的HashMap和Hashtable实现类

* 两个实现类的区别：
  * `Hashtable`是一个古老的实现类，虽然线程安全，但是性能较低；相对的，`HashMap`线程不安全，但是性能较高
  * `Hashtable`不允许使用`null`作为key和value，而`HashMap`中key和value都可以为`null`

* key对象的比较：两个实现类判断key相等的标准是，两个key通过`equals()`方法比较返回`true`，两个key的哈希值也相等。即只要两个key对象`equals()`返回`fasle`或者两个对象的哈希值不同，那么这两个对象就能同时保存在同一个`HashMap`中（`Hashtable`也一样）

> 所以在使用可变对象作为key时要十分谨慎，因为如果在存入Map后再修改key对象可能导致其哈希值的改变，从而`HashMap`无法准确访问到该key

## LinkedHashMap实现类

* 使用双向链表维护了键值对的次序，则迭代顺序与键值对的插入顺序一致
* 迭代访问时性能较好

## SortedMap接口和TreeMap实现类

* 基本可以参照`SortedSet`接口和`TreeSet`实现类

## WeakHashMap

&#160;&#160;&#160;&#160;`WeakHashMap`与`HashMap`基本相似，区别在与`HashMap`对于key保存的是强引用，而`WeakHashMap`对key保留的只是弱引用。这就意味着只要`HashMap`对象不被销毁，其中所有的key所引用的对象也不会被垃圾回收；而在`WeakHashMap`中的key所引用的对象如果没有被其他强引用变量所引用，就有可能被垃圾回收，`WeakHashMap`也会自动删除对应的键值对。强弱引用可参考{% post_link jvm2-gc %}

## IdentityHashMap实现类

&#160;&#160;&#160;&#160;`IdentityHashMap`与`HashMap`基本相似，区别在于只有当`key1==key2`时，`IdentityHashMap`才认为两个key是相等的。
