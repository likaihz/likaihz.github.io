---
title: Java笔记2：String，StringBuilder，StringBuffer
abbrlink: 1955439301
date: 2019-07-03 13:24:32
tags:
- Java
---

## String、StringBuilder和StringBuffer的区别

||String|StringBuilder|StringBuffer|
|-|-|-|-|
|是否可变| 不可变 | 可变 | 可变 |
|线程安全| 安全<sup>1</sup> | 不安全 | 安全 |
|拼接方法| `+` 或 `concat`方法 | `append`方法 | `append`方法 |
|拼接性能| 最差<sup>2</sup> | 最好 | 中间<sup>3</sup> |
|适用情况| 很少字符串操作 | 单线程大量字符串操作 | 多线程大量字符串操作 |

<!--more-->

1. 不可变对象天然是线程安全的，因为其只可读，不可写。
2. 在拼接两个`String`对象时，会创建一个新的`String`对象保存拼接的结果，因此性能较差，并且会产生较多的内存垃圾。而如果拼接的都是字符串字面量，会在编译时期优化。
3. `StringBuffer`由于要保证线程安全，因此比`StringBuilder`的性能差。

## String类的不可变性

### 如何保证不可变性
&#160; &#160; &#160; &#160;ava中`String`类的源码前两行如下（版本是jdk_1.8.0_221）：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    // ...
}
```
> 在Java9以前字符串采用`char[]`数组来保存字符，因此字符串的每个字符占2字节；从Java9开始，字符串采用`byte[]`数组再加一个encoding-flag字段来保存字符，因此Java9的字符串更加节省空间。

&#160;&#160;&#160;&#160;Java通过以下措施联合保证了`String`类的不可变性：
* 用`final`修饰value成员字段，保证其不被修改为其他数组对象。
* `String`类的所有方法都不会修改value字段中的任何元素。
* 用`final`修饰`String`类，将其设计为不可继承的，保证了value字段不会被子类获取到。

### 不可变性的优点

* 字符串不变保证了哈希值不变，因此`String`类中缓存了哈希值，当字符串对象作为`HashMap`的键时，能够提高效率。
* 字符串对象作为Java中非常常见的对象，针对字符串对象的空间优化能够大幅降低JVM的内存开销。其不变性让**字符串常量池（String Pool）**成为可能：
  * 采用字面值的方式创建一个字符串时（如`String str = "aaa";`），JVM首先会去字符串池中查找是否存在"aaa"这个对象，如果不存在，则在字符串池中创建"aaa"这个对象，然后将池中"aaa"这个对象的引用地址返回给字符串常量str，这样str会指向池中"aaa"这个字符串对象；如果存在，则不创建任何对象，直接将池中"aaa"这个对象的地址返回，赋给字符串常量。
  * 采用new关键字新建一个字符串对象时（如`String str2 = new String("aaa")`），JVM首先在字符串池中查找有没有"aaa"这个字符串对象，如果有，则不在池中再去创建"aaa"这个对象了，直接在堆中创建一个"aaa"字符串对象，然后将堆中的这个"aaa"对象的地址返回赋给引用str2；如果没有，则首先在字符串池中创建一个"aaa"字符串对象，然后再在堆中创建一个"aaa"字符串对象，然后将堆中这个"aaa"字符串对象的地址返回赋给str2引用。
  * 字符串的常量池能够减少相同字符串的重复创建，优化空间。
* 不可变对象是线程安全的，同一个字符串实例可以被多个线程共享。
* 类加载器要用到字符串，不可变性提供了安全性，以便正确的类被加载。

&#160;&#160;&#160;&#160;总的来说，字符串的不可变设计是考虑到效率和安全性的。