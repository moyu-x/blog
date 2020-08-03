---
title: Arraylist
date: 2020-08-02T18:13:04+08:00
tags:
  - Java
  - JUC
categories:
  - JUC
---

底层使用`Object[]`实现，操作受到底层数组实现的限制，在指定位置插入和删除元素的操作的时候会大量消耗性能，在末尾插入唯一消耗性能的是扩容，所以最好能在使用前预估好大小。

因为是基于数组实现的，其更符合计算机底层，其在内存上是连续排布，所以在获得内存地址加偏移后能很快的获取指定元素，所以支持高效的随机元素访问。

## RandomAccess

这个接口的主要作用就是标识实现此接口的类具有随机访问能力（也表明其更贴近计算机的底层实现），当容器类实现此接口的时候，`fori`循环的速度会快于`iterator`的遍历，因为其相当于直接在内存上递增查找。

例如使用`Collections.binarySearch()` 方法时候，容器类本身实现`RandomAccess`，则会走`indexedBinarySearch()`获取更高的性能，否则则会走`iteratorSearch()`

## 扩容

默认的元素大小是 10，在不指定大小的情况下调用重载的构造器构造默认大小为 10 的空数组`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，在指定大小为 0 的时候直接使用用于共享的空数组实例`EMPTY_ELEMENTDATA`，所以在默认情况下，只有在插入数据的时候才会构建，不然都是使用方法本身提供的共享空数组实例。从`Collection`对象中构建，先使用`toArray`方法直接转换，如果容量为 0 则用`EMPATY_ELEMENTDATA`替换原实例，否则在不是`ArrayList`对象则用`Arrays.copyOf()`方法转换后替代并保持原顺序。

### 1.8 的实现

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

		private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

		private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

		private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

		private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

默认直接空构造`ArrayList`，首先调用`ensureCapacityInternal`进行容量大小的判断；当添加第一个元素的时候，`minCapacity`为 1，`DEFAULT_CAPACITY`为 10，最后此方法会返回 10；调用`ensureExplicitCapacity`方法，`minCapacity - elementData.legth > 10`成立，调用`grow`方法，第二个元素的时候这两个值相同，不会调用`grow`方法；grow 的方法直接将容量进行右移一位的操作（右移 n 位相当于$原数据\times2^n$），此时新的容量为此前的 1.5 倍左右，小于最小值的时候直接用最小值替代扩容值（这个地方一般出现在指定初始化容量小于`DEFAULT_CAPACITY`或者是空构造添加第一个元素的时候）

`hugeCapacity`对于溢出的限制，当前`minCapacity`还是比数组最大值小的时候为数组最大值，否则为整数最大值；当扩容到`INTEGER.MAX_VALUE`的情况下，在下一次增加元素的情况下，`minCapacity`已经溢出，这个时候余下两个判段都会进入，最后抛出`OutOfMemoryError`错误。

`Arrays.copyOf`的内部调用`System.arrayCopy`进行实现，`copyOf`的目的是拷贝原数组指定容量的数据，并返回新数组；而`System.arrayCopy`是一个`native`方法，主要是拷贝指定数组的指定数据到目标数组，所以`copyof`内部用此来实现高效的数组拷贝。]

`modCount`的作用：这是来源于`AbstractList`的一个变量，用来表示当前列表被操作的次数，使用这个数据可以进行快速的判错，入`iterator`用此来判断是否被修改而抛出异常。

`ensureCapacity`这个方法是提供给使用者使用的，如果在初始化的时候没有创建合适的容量，可以在向其添加数据前调用此方法用以扩容到合适的容量，以减少增加过程中的扩容消耗，其也是调用`ensureExplicitCapacity`来实现扩容

### 1.11 实现

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }

		private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }

```

核心的扩容原理没有改变，只是前置条件改了，当插入位置已经和原有列表长度一直的时候就扩容，减少了判断次数，删除了`ensureExplicitCapacity`和`ensureCapacityInternal`方法。
