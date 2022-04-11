[TOC]

[参考B站视频](https://b23.tv/9z77Yfm)

[资料](https://www.icloud.com.cn/iclouddrive/0a6FuJdMZfSBXNz1lwbIgW8NA)

## JAVA基础

### 面向对象

#### 什么是面向对象

对比面向过程，是两种不同的处理问题的角度
面向过程更注重事情的每一个步骤及顺序，面向对象更注重事情有哪些参与者(对象)、及各自需要做什么

#### 面向对象的特征

**封装**:内部细节对外部调用透明，外部调用无需修改或者关心内部实现

**继承**:继承基类的方法，并做出自己的改变和/或扩展 ，子类共性的方法或者属性直接使用父类的，而不需要自己再定义，只需扩展自己个性化的

**多态**:基于对象所属类的不同，外部对同一个方法的调用，实际执行的逻辑不同。 继承，方法重写是多态的基础

```java
父类类型 变量名 = new 子类对象 ; 
变量名.方法名();//此处不能调用子类 独有 的方法
```

#### 面向对象的好处

- 封装后的对象可以复用
- 继承和多态易于扩展

## ArrayList和LinkedList区别

### 特征

#### ArrayList

基于动态数组，连续内存存储，适合下标访问(随机访问)，扩容机制:因为数组长度固 定，超出长度存数据时需要新建数组，然后将老数组的数据拷贝到新数组，如果不是尾部插入数据还会 涉及到元素的移动(往后复制一份，插入新元素)，使用尾插法并指定初始容量可以极大提升性能、甚 至超过linkedList(需要创建大量的node对象)

**初始化方式**

不设置初始化容量

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
ArrayList<Object> arrayList1 = new ArrayList<>();
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;//其实是一个空数组
}
```

```java
//在进行添加元素时，如果数组为空，会首先初始化容量为10
private static final int DEFAULT_CAPACITY = 10;
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
```



设置初始化容量

```java
ArrayList<Object> arrayList2 = new ArrayList<>(1);
public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

**扩容方法**

按照原数组大小1.5倍的扩容,既不能太大也不小，太大造成资源的浪费，太小会造成频繁的发生扩容，导致频繁的复制数组（因为数组的扩容是创建一个新的数组，再将原来的元素复制到新数组上去）

```java
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
```

最大扩容为`Integer.MAX_VALUE`，[为什么有的时候是MAX_ARRAY_SIZE](https://blog.csdn.net/w605283073/article/details/109696771)

```java
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
}
```



#### LinkedList

**初始化方式**

```java
LinkedList<Object> linkedList1 = new LinkedList<>();
LinkedList<Object> linkedList2 = new LinkedList<>(Arrays.asList(1,2,3,4));
```

基于链表，可以存储在分散的内存中，适合做数据插入及删除操作，不适合查询:需要逐 一遍历

遍历LinkedList必须使用iterator不能使用for循环，因为每次for循环体内通过get(i)取得某一元素时都需 要对list重新进行遍历，性能消耗极大。

```java
LinkedList<Object> linkedList2 = new LinkedList<>(Arrays.asList(1,2,3,4));
for (int i = 0; i < linkedList2.size(); i++) {
     linkedList2.get(i);
}

public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
}

Node<E> node(int index) {
        // assert isElementIndex(index);
        if (index < (size >> 1)) {//判断索引值是否小于链表大小的一半
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
}
```

​	另外不要试图使用indexOf等返回元素索引，并利用其进行遍历，使用indexOf对list进行了遍历，当结 果为空时会遍历整个列表。

```java
public int indexOf(Object o) {
        int index = 0;
        if (o == null) {//当值为空，遍历整个链表
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

