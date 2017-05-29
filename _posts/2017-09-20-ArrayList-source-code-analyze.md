---
layout: post
title: ArrayList源码浅析
tags:
- Java
- 集合
categories: Java
description: 
---
##### 概述
ArrayList是基于数组实现的List，支持快速随机访问，其容量能“自动”增长。本文分析的ArrayList代码来源与Android中的jdk7，与Oracle的java存在不少差异。


#### 成员变量
    private static final int MIN_CAPACITY_INCREMENT = 12; //最小增量
    transient Object[] array; //不可序列化的内部数组，所有操作都基于它
    int size; //实际元素个数
    protected transient int modCount; //基类AbstractList中的变量，修改次数，用于并发控制

#### 构造器
public ArrayList(int capacity) ：构造指定初始容量的数组
public ArrayList()：构造一个空数组
public ArrayList(Collection<? extends E> collection)：构造一个包含指定集合元素的数组
#### 读取
E get(int index)：直接读取array的指定索引值
#### 添加
（1）add(E e)：将指定的元素添加到列表的尾部。

    public boolean add(E object) {
        Object[] a = array;
        int s = size;
        if (s == a.length) { //数组已存满则先扩容
            //现有元素个数小于6时加12，否则扩至1.5倍
            Object[] newArray = new Object[s +
                    (s < (MIN_CAPACITY_INCREMENT / 2) ?
                     MIN_CAPACITY_INCREMENT : s >> 1)];
            System.arraycopy(a, 0, newArray, 0, s);//拷贝到新数组
            array = a = newArray;
        }
        a[s] = object;
        size = s + 1;
        modCount++; 
        return true;
    }
（2）add(int index, E element)：在index位置插入element。

    public void add(int index, E object) {
        Object[] a = array;
        int s = size;
        if (index > s || index < 0) {
            throwIndexOutOfBoundsException(index, s);
        }

        if (s < a.length) { //空间充足，index位置开始的元素后移一步
            System.arraycopy(a, index, a, index + 1, s - index);
        } else {
            // 空间不够，先生成新的数组再转移
            Object[] newArray = new Object[newCapacity(s)];
            System.arraycopy(a, 0, newArray, 0, index);
            System.arraycopy(a, index, newArray, index + 1, s - index);
            array = a = newArray;
        }
        a[index] = object;
        size = s + 1;
        modCount++;
    }
（3）addAll(Collection<? extends E> c)：将特定Collection中的元素添加到Arraylist末尾，原理类似add单个元素
（4）addAll(int index, Collection<? extends E> c)：将特定Collection中的元素添加到index位置，原理类似add单个元素


#### 设置
E set(int index, E object)：将新元素放入array[Index]，返回原先此处的元素

#### 清空

    public void clear() {
        if (size != 0) {
            Arrays.fill(array, 0, size, null); //用null填充array
            size = 0;//设置array的size为0
            modCount++;
        }
    }

#### 删除

    public E remove(int index) {
        Object[] a = array;
        int s = size;
        if (index >= s) {
            throwIndexOutOfBoundsException(index, s);
        }
        E result = (E) a[index];
         //把array中index以后的元素前移一位，最后一位设为null防内存泄露
        System.arraycopy(a, index + 1, a, index, --s - index);
        a[s] = null;  // Prevent memory leak
        size = s;
        modCount++;
        return result; //返回index位置原元素
    }

    @Override public boolean remove(Object object) {
        。。。
        //遍历array删除object，原理同上，返回成功失败，注意object为null时的处理
    }

    @Override protected void removeRange(int fromIndex, int toIndex) {
        。。。
        //删除指定范围元素，不包含toIndex所在位置，把后面不要的位置全部设为null
    }
#### 迭代器
    public Iterator<E> iterator() {
        return new ArrayListIterator();
    }

    private class ArrayListIterator implements Iterator<E> {
        private int remaining = size; //当前迭代剩余元素个数
        private int removalIndex = -1; //可以被删除的元素的索引，即next方法遍历的当前元素
         private int expectedModCount = modCount; //迭代中的modCount

        public boolean hasNext() {
            return remaining != 0; //是否还有未迭代的元素
        }

        @SuppressWarnings("unchecked") public E next() {
            ArrayList<E> ourList = ArrayList.this;
            int rem = remaining;
            //当前List的modCount与迭代的计数不一致，可能被其他线程并发修改，抛异常
            if (ourList.modCount != expectedModCount) { 
                throw new ConcurrentModificationException();
            }
            if (rem == 0) { 
                throw new NoSuchElementException();
            }
            remaining = rem - 1;
            //返回当前遍历的元素，并标记它的索引为可删除
            return (E) ourList.array[removalIndex = ourList.size - rem];
        }

        public void remove() {
            Object[] a = array;
            int removalIdx = removalIndex;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            if (removalIdx < 0) {
                throw new IllegalStateException();
            }
            System.arraycopy(a, removalIdx + 1, a, removalIdx, remaining); //剩余元素前移一位
            a[--size] = null;  // Prevent memory leak
            removalIndex = -1;
            expectedModCount = ++modCount; //算作一次修改
        }
    }
#### 其他方法
1、ensureCapacity(int)：确保容量不低于一个最小值

    public void ensureCapacity(int minimumCapacity) {
        Object[] a = array;
        //如果当前容量低于入参，按入参创建新数组
        if (a.length < minimumCapacity) {
            Object[] newArray = new Object[minimumCapacity];
            System.arraycopy(a, 0, newArray, 0, size);
            array = newArray;
            modCount++;
        }
    }
2、clear()：清空List

    public void clear() {
        if (size != 0) { //将数组元素全部置为null，size归零
            Arrays.fill(array, 0, size, null); 
            size = 0;
            modCount++;
        }
    }
3、trimToSize：调整List大小，使容量和元素个数相同。

    public void trimToSize() {
        int s = size;
        if (s == array.length) {
            return;
        }
        if (s == 0) {
            array = EmptyArray.OBJECT;
        } else {
            Object[] newArray = new Object[s];
            System.arraycopy(array, 0, newArray, 0, s); //转移到新数组
            array = newArray;
        }
        modCount++;
    }

### Fail-Fast机制
ArrayList采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。