---
title: Java笔记3：Collection接口
tags:
  - Java
abbrlink: 3337726739
date: 2019-07-04 15:12:09
---

## Java集合

&#160;&#160;&#160;&#160;Java中集合类主要用于保存、盛装其他类型的数据，因此集合类也称为容器类。所有的集合类都位于`java.util`包下，容器类只能保存引用类型。Java中的集合类主要有两个接口：`Collection`和`Map`，本文主要讨论`Collection`接口。`Collection`接口的继承数如下：

{% asset_img Collection.png Collection %}

<!--more-->

## Set接口

* Set接口与Collection基本相同，没有提供额外的方法
* Set接口的所有实现类都不允许包含重复元素

### HashSet类

* `HashSet`按照哈希码来存储集合中的元素，因此具有很好的存取和查找性能。
* 不保证元素的排列顺序
* 不是线程安全的
* 元素值可以是`null`

&#160;&#160;&#160;&#160;向HashSet中存入一个新元素时，HashSet会调用该对象的`hashCode()`方法来得到该对象的哈希值，然后根据哈希值决定该对象在集合中的存储位置。如果两个元素`equals()`方法返回`true`，但是哈希值不等，仍然可以添加成功；反过来，如果两个对象哈希值相同但是`equals()`返回`false`，也仍然可以添加成功。

> 如果需要把某个类型的对象保存到`HashSet`中，重写该类的`hashCode()`方法和`equals()`方法时，尽量保证如果`equals()`方法返回`true`，那么其哈希值也要相等。

&#160;&#160;&#160;&#160;由于`HashSet`的存取和查找都是依赖于哈希值，因此在存储可变对象时会有风险，如果修改HashSet集合中的对象，有可能导致该对象的哈希值变化，从而导致`HashSet`无法准确访问该对象。

> `HashSet`底层实现是基于`HashMap`的，具体可参照`HashMap`的实现。

### LinkedHashSet类

&#160;&#160;&#160;&#160; `LinkedHashSet`是`HashSet`的子类，内部使用了一个链表维护了元素的插入顺序，因此性能略低于`HashSet`，但是在迭代时有很好的性能。

### TreeSet类

&#160;&#160;&#160;&#160; `TreeSet`是`SortedSet`接口的实现类，可以确保集合元素处于排序状态。支持两种排序方法：
* 自然排序：直接调用元素的compareTo方法比较元素之间的大小（实现了Comparable接口的对象），然后将集合元素按升序排列
* 定制排序：在构造`TreeSet`实例时，提供一个`Comparator`对象用于对元素进行比较。

### Set接口性能分析

* `HashSet`的性能总是比`TreeSet`好，因为排序需要额外的开销。
* `LinkedHashSet`一般情况下比`HashSet`略慢，但是在遍历访问时，`LinkedHashSet`会更快。

## List接口

* `List`接口根据整数索引访问其中的元素。

### ArrayList类和Vector类

* 相同点：
  * 长度动态可变
  * 内部用`Object[]`数组实现
  * 不指定初始容量时，默认为10；如果需要一次性添加大量元素，可使用`ensureCapacity(int minCapacity)`方法，减少重分配的次数提高性能。

* 不同点：
  * `Vector`是线程安全的，而`ArrayList`不是线程安全的
  * `Vector`的性能低于`ArrayList`

### 固定长度的List

* `Arrays.asList(Object... a)`方法可以把一个数组或一些对象转换成一个`List`集合，但是返回的是`Arrays.ArrayList`的实例，其长度固定，只能遍历访问该集合里的元素，不可增加、删除。

## Queue接口

* 用于模拟队列，是一个“先进先出”（FIFO）的容器。

### PriorityQueue实现类

* 即数据结构中“优先队列”的实现
* 排序方式与`TreeSet`一致

### Deque接口与ArrayDeque类

* `Deque`接口是`Queue`接口的子接口，表示一个双端队列
* 该接口有一个实现类`ArrayDeque`，用数组`Object[]`实现

### LinkedList实现类

* 内部用链表的像是保存集合中的元素
* 随机访问性能价差，插入删除元素的性能较好

## 各种线性表的性能分析

&#160;&#160;&#160;&#160; `List`接口是一种线性表接口，其不同的实现在不同的应用场景中有性能差异

* 如果需要遍历集合元素，对于`ArrayList`对象和`Vector`对象应该使用索引来访问；而对于`LinkedList`对象，应该采用迭代器来遍历
* 如果需要经常执行插入、删除操作，可考虑使用`LinkedList`。
* 如果有多个线程需要同时访问`List`集合中的元素，可考虑使用`Collections`将集合包装成线程安全的。 
