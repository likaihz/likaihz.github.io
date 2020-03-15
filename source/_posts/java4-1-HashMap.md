---
title: Java笔记4.1：HashMap源码探索
tags:
  - Java
  - 源码
abbrlink: 487642189
date: 2019-07-06 00:00:00
mathjax: true
---

## 基本数据模型

&#160;&#160;&#160;&#160;在`HashMap`的实现中，最基本的数据模型有两个，分别是用来表示一个键值对的类`Node<K, V>`和用于保存所有键值对的数组`transient Node<K,V>[] table;`。`Node<K, V>`的部分定义如下：

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    // methods
}
```

<!--more-->

在向`HashMap`中添加键值对时，会先根据key的哈希值进行再一次的哈希运算，得到该键值对在`table`中的位置（源码中对`table`中的一个位置称为一个桶，bucket）。该过程不可避免地会产生哈希冲突，即出现计算出的位置上已经有其他键值对存在的情况。为了兼顾时间性能与空间性能，`HashMap`采用了以下两个主要的优化手段：

* 适当的时机**扩容**以降低冲突：当向表中插入的键值对太多时，发生哈希冲突的概率会提高。因此在`HashMap`中的键值对数量超过阈值时进行扩容与重新哈希。
* 当“桶”中对象数太多时，用红黑树组织。`HashMap`采用的解决冲突的办法是拉链法，即将新的值仍然插入到该位置，形成一个链表。如果链表过长，会产生性能问题，因此从jdk1.8开始，又引入了红黑树，即当某个位置上的链表长度超过阈值时，将链表改造为一棵红黑树，这个过程称为“**树化**”。树化过程中，`Node`对象将变为`HashMap.TreeNode`类型(是`Node`的子类)

&#160;&#160;&#160;&#160;综上，`HashMap`内部的结构如下：

{% asset_img hashmap.png hashmap %}

## 哈希与扩容

### 哈希值计算——为什么要右移16位？

&#160;&#160;&#160;&#160;`HashMap`中会根据key对象的hashCode重新计算哈希值，计算哈希值的方法如下：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

从中可以得到以下两方面的信息：
* `HashMap`是允许`null`作为key的，并且`null`的哈希为0
* 一般的key在进行计算哈希时，用其哈希值右移16位的结果与其自身做按位异或。

&#160;&#160;&#160;&#160;注释中简单解释了采用这种哈希函数的原因，大意是说寻址计算时，能够参与到计算的有效二进制位仅仅是右侧和数组长度值对应的那几位，意味着发生冲突的几率会高。通过移位和异或运算让hashCode的高位能够参与到寻址计算中。采用这种方式是为了在性能、实用性、分布质量上取得一个平衡。有很多hashCode算法都已经是分布合理的了，并且大量冲突时，还可以通过树结构来解决查询性能问题。

### 容量与寻址——为什么是2的次方？

&#160;&#160;&#160;&#160;`HashMap`中根据哈希结果计算数组下标（寻址）的方法如下：`hash & (capacity-1)`，其中hash是重新计算过的hash值，capacity是`table`的长度。由于`HashMap`的实现确保了`capacity`是2的次幂，此时`hash & (capacity-1)`的运算效果等同于`hash % capacity`，而位运算的效率高，这样的写法能够提高寻址的性能。

> 这也很好理解，例如length为16时，其二进制为00010000，那么length-1的二进制为00001111，与hash做按位与计算时，结果就是只留下低4位的值，屏蔽了高位，也即是height%length的结果。

&#160;&#160;&#160;&#160;扩容相关的方法`resize()`中的部分代码如下：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
    // something else...
}
```

* 在构造`HashMap`时，`table`为`null`，只有在第一次向表中放入对象时才会触发`resize()`方法，构建一个容量为16（即代码中的`DEFAULT_INITIAL_CAPACITY`）的数组。
* 每一次触发扩容时，都会将数组的大小扩充为原来的两倍。这就保证了数组的大小一定是2的次幂。

## 扩容与负载因子

### 何时触发扩容

&#160;&#160;&#160;&#160;在向表中添加键值对时，如果表内已有的键值对数量大于阈值`threshold`，就会触发一次扩容，扩容的过程上文已有简述。

### 负载因子

&#160;&#160;&#160;&#160;`HashMap`中有一个字段为`loadFactor`，称为负载因子，阈值的计算就是依赖于负载因子。`threshold = capacity * loadFactor`。如果`HashMap` 在构造时没有指定负载因子，默认的负载因子为0.75。如果扩容因子越大，意味着哈希表越满，碰撞的概率也就越大，发生碰撞后的代价也更大，结果导致效率大打折扣。因此扩容因子取0.75也是一个空间换时间的考虑。


## 树化

&#160;&#160;&#160;&#160;前面已经提到过树化的概念，即当table中某个位置出现了太多的冲突时，就要将该位置上的元素由链表排列改为按红黑树排列，以提高查找性能。在`HashMap`中，这个阈值默认为8。

> 在链表查找时，时间复杂度为为$O(n)$，而在红黑树上进行查找时，时间复杂度为$O(\log n)$

### 为什么不直接用红黑树？

&#160;&#160;&#160;&#160;对于这一点，源码注释用也有提到，因为在用红黑树组织对象时，树的节点所用的类为`HashMap.TreeNode`，该类的大小几乎为其祖先类`HashMap.Node`的两倍，而在链表很小的时候，其实其查找性能的差异并不是特别大，因此这是一个空间换时间的决定。

### 为什么树化的默认阈值为8？

&#160;&#160;&#160;&#160;源码的注释中有提到，假设hashCode真的分布很均匀，那么数组中每个位置出现的对象数符合泊松分布，在负载因子为0.75的前提下，分布为：

| 频数 | 概率 |
|-|-|
|0|0.60653066|
|1|0.30326533|
|2|0.07581633|
|3|0.01263606|
|4|0.00157952|
|5|0.00015795|
|6|0.00001316|
|7|0.00000094|
|8|0.00000006|
|大于8|小于一千万分之一|

可见其实当阈值定为8时，需要树化的概率已经相当低了，大于8的概率更是低到可以忽略不及。因此阈值取8已经有很好的时间和空间性能了。
