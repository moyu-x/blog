---
title: ConcurrentHashMap源码阅读
date: 2020-08-05T23:48:20+08:00
tags:
  - Java
  - Collection
categories:
  - Collection
---


线程安全，在类注释中就直接表面这个是为了替代`HashTale`来实现的，并且兼容`HashTable`的操作方法。

## 1.7版本

内部维护了一个`Segment`的内部类，通过继承`ReentrantLock`来实现，为了简化一些锁操作，避免单独的构造。`Segment`维护了一个`entry`列表来保持一致性，所以可以在读的时候无锁，然后再`segment`的内部使用`hashentry`数组来维护数据。

### 初始化

大体步骤如下：

1. 参数校验
2. 计算掩码，偏移量操作，默认构造掩码是15，偏移量28
3. 初始化，segment的容量是`sszie`,是通过循环左移1，直到找到一个小于`concurrentLevel`附近的数,
    1. `Segment[0]`的`HashEntry[]`大小是`cap`,通过`initialCapacity / ssize`计算得到`c`，然后移循环左移找到数据，其是又是一个小于`c`附近的2的幂次值
    2. `Segment[]`的大小是`sszie`
    3. 将s0放到ss中是通过`unsafe`类的`putOrderedObject`操作实现的

初始化`segment[0]`，默认为2，负载因子是0.75，扩容的阈值就是1.5，只有插入第二个数值的时候才会扩容。

### put操作

```java
    /**
     * Applies a supplemental hash function to a given hashCode, which
     * defends against poor quality hash functions.  This is critical
     * because ConcurrentHashMap uses power-of-two length hash tables,
     * that otherwise encounter collisions for hashCodes that do not
     * differ in lower or upper bits.
     */
    private static int hash(int h) {
        // Spread bits to regularize both segment and index locations,
        // using variant of single-word Wang/Jenkins hash.
        h += (h <<  15) ^ 0xffffcd7d;
        h ^= (h >>> 10);
        h += (h <<   3);
        h ^= (h >>>  6);
        h += (h <<   2) + (h << 14);
        return h ^ (h >>> 16);
    }
```

在此版本中的hash算法，当然也就要传入调用`hashcode`方法后的值，也是遵循着冲突容易发生在低位，所以用移位，异或，求和操作得到一个值，最后再与自己右移16的数进行异或操作得到最终的哈希值。

对于存储在哪个segment位置的计算，在使用默认的构造情况下，相当于将hash值右移28位后再与掩码15进行与运算。

在发现插入位的segment为null的时候，调用ensureSegment操作进行初始化，对于这个初始化，流程大体如下：

1. 从0中获取初始化长度作为cap
2. 从0中获取负载因子
3. 计算扩容阈值
4. 创建一个cap容量的`HashEntry`数组

这个操作类似构造器初始化的时候创建`segment[0]`的数据，当然不同的是，在这个操作中，会利用unsafe类中的自旋操检测是否还是null，然后再使用unsafe类的cas操作。

在最后插入值的时候，首先是使用`tryLock`获取锁，未活得则调用`scanAndLockForPut`方法自旋的获取锁，超过`Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1`的时候就使用`lock()`强制获取锁。

使用`(tab.length - 1) & hash`计算出插入的位置，对这个`HashEntry`链表进行遍历：

1. key存在，直接替换
2. 不存在，接着遍历
    1. 容量不够，扩容，也就是移动到了扩容位
    2. 移动到最后，也就是key不存在，也扩容结束，就使用头插法插入数据

### 扩容rehash

从注释和实际代码中，我们可以看到，扩容是直接扩容为原来的2倍，老数组里的数据移动到新的数组中的时候，要么不变，要么是index+oldsize，最后用头插法插入新节点。

## 1.8版本

当然和同版本的HashMap一样，修改成了链表变红黑树的实现，对于位置从原来的`segment`修改成了`node`数组。对于构造器，只是申明的大小，只有在插入数据的时候才会去开辟空间。

在操作前面有`tabAt`，`casTabAt`和`setTabAt`三个前置操作，进行数据可见性的操作，`setTabAt`操作始终锁定在锁定区域内进行，因此只要求发布顺序。

对于`hash`函数，其与`HashMap`不同，是在右移16，减少冲突后，然后在与本身进行异或操作，这个适合会出现负数请求，所以再与`0x7fffffff`进行并操作后，就能获取正数。

```java
    /**
     * Spreads (XORs) higher bits of hash to lower and also forces top
     * bit to 0. Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```

### get操作

在get操作的适合，使用`tabAt`来进行可见性操作。其原理比较简单，根据哈希值和tabat找到node节点，头简单就是就返回，是红黑树就用find函数查找，是链表就遍历查找。

### put操作

当添加到`table`表空间点的时候：

1. 为null，也就刚申明好，这个时候使用`initTable`调用进行空间的开辟
2. 桶内为空，也就是刚初始化好，进行的第二次循环，这个时候直接使用cas操作放入数据
3. 当当前位置的`hashcode == MOVED == -1`就进行扩容，和之前的tabat操作有关
4. 代码顺序，在tabat操作的时候已经找到了需要插入node节点的位置，在下一次循环的时候就可以进行真实的数据插入操作；
    1. 直接使用synchronized锁住当前节点
    2. 再次使用tabat进行同步，fh值是通过hash得到的，为正数的时候说明还是最初的列表，然后在次节点链表中进行遍历，同时进行bincount的自增操作，key相同则替换数据，移动到末尾则插入数据
    3. 如果是红黑树，则使用红黑树的插入操作。
5. 最后检查bincount的值，大于树化阈值则进行书化操作

### remove操作

通过浏览remove的代码，可以看到大体逻辑和put操作类似，但是可以发现，在红黑树删除的代码里面，有这么一行`setTabAt(tab, i, untreeify(t.first));`，也就是说当冲突减小到阈值的时候，又会变为链表。