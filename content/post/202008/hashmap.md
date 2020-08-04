---
title: HashMap源码阅读
date: 2020-08-04T21:19:24+08:00
tags:
  - Java
  - Collection
categories:
  - Collection
---


1.8之前是数组+链表构成，1.8改成了当阈值为8的时候，将链表转换为红黑树（当链表长度小于64的时候是会进行扩容而不是直接转红黑树），在键值上运行使用null作为键值，

## 1.7版本

创建的默认值是16，如果指定大小，则从1开始向左移位移循环，直到找到一个$2^n$数字大于或等于初始化容量大小作为构建的容量。

对于哈希函数，要求传入的是对象的`hashcode()`方法的值，对其进行一个补充的哈希计算，防止出现劣质的哈希值（也就是冲突较大，哈希后的数值分布不均匀），在注释中可以看到hashmap用的2的幂次方的哈希表，容易在地位出现冲突，`null`的作为key用于返回`0`哈希值。其在使用异或操作和右移操作确保每个位置仅有一定数量的冲突。

```java
    /**
     * Applies a supplemental hash function to a given hashCode, which
     * defends against poor quality hash functions.  This is critical
     * because HashMap uses power-of-two length hash tables, that
     * otherwise encounter collisions for hashCodes that do not differ
     * in lower bits. Note: Null keys always map to hash 0, thus index 0.
     */
    static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * Returns index for hash code h.
     */
    static int indexFor(int h, int length) {
        return h & (length-1);
    }
```

因为1.7的代码中解决冲突的方式很暴力，就直接使用链地址法（也就是拉链法），再用`indexFor`找到要放置的链表位置，遍历查找是否有已经存在，最后在数组中指定位置插入数据，如果超过数组的存储阈值就直接扩大两倍然后拷贝到新数组中。

删除操作也是也是直接通过`indexFor`函数查出存在哪个bucket中，然后再其中进行遍历查找出数据进行删除。

## 1.8版本

对于使用默认无参的构造器进行构建，则在初始化容量的时候会使用`tableSizeFor`进行操作。`tableSizeFor`使用移位和异或操作进行实现的，相比于1.7的版本，其在获取数据的时候是常数级别的。

因为冲突经常发生在最低位，所以1.8处理的方式比较简单，直接右移16位，将高位移动到地位，这样在哈希算法寻找位置的时候冲突较低。

在添加元素的时候，当第一次添加元素的时候才会在添加位置创建链表分配空间，然后才进行数据的插入，此时被分为了三种情况：

1. 元素本身就是链表头节点，那直接进行后续的操作
2. 需要插入的节点是红黑树，那走红黑树的插入
3. 插入节点还是普通列表，会遍历该节点，查找是否存在同key的，有就跳出进行后续的操作，否则继续寻找，直到走到末尾节点，此时如果已经超过了默认的8个元素，就会调用`treeifyBin`进行数化（在此函数中会判断`table`数组大小是否超过64，只有超过64的时候才会进行当前树化，否则只会进行当前节点的扩容）

在扩容的`resize`的代码注释上写着，扩容后，要么保持在相同的索引位置，要么就要移动到新表中2次幂的位置

```java
    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

    /**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
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
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```