---
layout: post
title: SparseArray初探
tags:
- Android
categories: Android
description: 
---
#### 概述 
SparseArray是一种类似HashMap的映射结构，以int为键，查询时使用二分法，不适合存储大量数据。

#### 成员与构造器 
首先看成员：
{% highlight c++ linenos %}
    private static final Object DELETED = new Object();  //已删除标记
    private boolean mGarbage = false; //是否需要进行gc
    private int[] mKeys; //键集
    private Object[] mValues; //值集
    private int mSize； //元素数目
{% endhighlight %}
再看构造器：
{% highlight c++ linenos %}
    public SparseArray() {
        this(10); //默认初始容量10
    }

    public SparseArray(int initialCapacity) { //建议按需初始化大小
        if (initialCapacity == 0) {
            mKeys = EmptyArray.INT;
            mValues = EmptyArray.OBJECT;
        } else {
            mValues = ArrayUtils.newUnpaddedObjectArray(initialCapacity);
            mKeys = new int[mValues.length];
        }
        mSize = 0; //初始无任何元素
    }
{% endhighlight %}
#### 主要方法 
1、put：把指定key和value放到集合里
{% highlight c++ linenos %}
    public void put(int key, E value) {
        //二分查找键集中是否有对应的key，有则返回数组的索引，否则返回lo按位取反的值，该值为负数
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key); 

        if (i >= 0) { //键集中已有该key，直接覆盖value
            mValues[i] = value;
        } else { //键集中不含该key，把返回的lo再次按位取反
            i = ~i;
            //如果该位置已被删除，则用新的key和value替换
            if (i < mSize && mValues[i] == DELETED) {
                mKeys[i] = key;
                mValues[i] = value;
                return;
            }
            //如需gc先gc
            if (mGarbage && mSize >= mKeys.length) {
                gc(); //数据前移

                // Search again because indices may have changed.
                i = ~ContainerHelpers.binarySearch(mKeys, mSize, key);
            }
            //插入key和value，不够就扩容翻倍
            mKeys = GrowingArrayUtils.insert(mKeys, mSize, i, key);
            mValues = GrowingArrayUtils.insert(mValues, mSize, i, value);
            mSize++;
        }
    }
{% endhighlight %}
2、get：根据键查找值
{% highlight c++ linenos %}
    public E get(int key) {
        return get(key, null); //设置默认值为null，调下面的方法
    }

    public E get(int key, E valueIfKeyNotFound) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key); //二分查找
        //未找到或者已删除则返回默认值，找到返回具体值
        if (i < 0 || mValues[i] == DELETED) {
            return valueIfKeyNotFound;
        } else {
            return (E) mValues[i];
        }
    }
{% endhighlight %}
3、delete：删除。此方法并未马上删除键值，只是先标记为可删除和待回收
{% highlight c++ linenos %}
    public void delete(int key) {
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        //查到key对应的索引，将value置为删除，标记可gc
        if (i >= 0) {
            if (mValues[i] != DELETED) {
                mValues[i] = DELETED;
                mGarbage = true;
            }
        }
    }
{% endhighlight %}
#### 其它方法 
1、gc()，名为垃圾回收，实则是将可用的数组元素前移，并得出数组存储元素的数目，在其它依赖元素确切数目的方法中都要先行调用此方法。

2、remove系列方法，可根据key、数组索引删除一个或多个元素。

3、keyAt/valueAt（int），根据数组索引获取键/值，需先gc

4、size()，获取大小，需先gc

5、clear()，将values数组元素全部置为null，mSize置0，mGarbage置为false

6、append(int key, E value)，插入新元素到最后