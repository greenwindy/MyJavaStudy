# ArrayList

线性表的一个重要特征：除了头尾元素，每个元素都有确定的唯一前驱和唯一后继。

## 特点以及源码分析

ArrayList 是List的一个具体实现，底层结构是一个序列化的数组。它的初始容量是10，每次扩容成原数组长度的1.5倍，可以通过构造方法给定初始容量。

```java
class ArrayList<E> {
    
    transient Object[] elementData; // ArrayList底层所维护的数组
    
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};// 空数组
    
    int size;// 元素个数
    
    private static final int DEFAULT_CAPACITY = 10;// 默认初始容量
    
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    
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
    
    
    public boolean add(E e) {
        // 数组容量检查
        ensureCapacityInternal(size + 1); 
                
        elementData[size] = e;
        size++;
        
        return true;
    }
    
    private void ensureCapacityInternal(int minCapacity) {
         
         // 如果数组为空，容量就设为给定容量和默认初始容量中的最大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;// 记录这个对象(ArrayList)被修改的次数
		
        // 如果目标容量大于现有的数组长度，扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;// 设定的最大数组长度

    private void grow(int minCapacity) {
        // 旧容量是数组长度
        int oldCapacity = elementData.length;
        
        // 新容量设为旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        
        // 如果计算的新容量值溢出，设为ArrayList元素个数+1(传入的newCapacity)
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        
		// 如果计算的新容量值超过设定的最大数组长度，设为最大数组长度
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        
        // 将原来的数据复制进新数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```

ArrayList允许重复, 允许null, 有序，线程不安全。

## ListIterator

- ArrayList的toString()方法通过迭代器iterator遍历容器中的元素，而ArrayList自己带有返回实现了Iterator接口的ListIterator类的方法`listIterator()`和`listIterator(int index)`。
- ListIterator有两个成员变量cursor和lastRet。cursor初始是0，指向当前位置和下一个要遍历的位置中间的部分。lastRet初始是1，指向前一个遍历过的元素的位置。

- ListIterator较Iterator多了向前遍历的方法和添加元素、修改元素的方法。next()方法和previous()方法会使lastRet指向前一个遍历过的位置。，使用`remove()`方法、 `add(int index, E element)` 方法会令lastRet = -1，而`remove()`方法和`set(int index, E element)`方法要用到lastRet，故不能连续使用`remove()`或在添加元素后使用删除或修改方法。
- 迭代器在迭代时会检查源集合的修改次数和迭代器的记录的修改次数是否相同，如果不同，抛出并发修改异常`ConcurrentModificationException`，但用迭代器的修改方法不会改变修改次数。

## 常用方法

|           作用           | 返回值类型 |                      方法                       |
| :----------------------: | :--------: | :---------------------------------------------: |
|      添加元素至表尾      | `boolean`  |                   `add(E e)`                    |
|    添加元素至指定位置    |   `void`   |           `add(int index, E element)`           |
|    返回指定位置的元素    |    `E`     |                `get(int index)`                 |
|     删除指定位置元素     |    `E`     |               `remove(int index)`               |
|      指定元素的位置      |   `int`    |               `indexOf(Object o)`               |
|    删除第一个指定元素    | `boolean`  |               `remove(Object o)`                |
|           修改           |    `E`     |           `set(int index, E element)`           |
|         元素个数         |   `int`    |                    `size()`                     |
|       是否包含元素       | `boolean`  |              `contains(Object o)`               |
|           清空           |   `void`   |                    `clear()`                    |
|      添加集合内元素      | `boolean`  |       `addAll(Collection<? extends E> c)`       |
| 添加集合内元素至指定位置 | `boolean`  | `addAll(int index,  Collection<? extends E> c)` |

- `subList(int fromIndex, int toIndex)`方法获取的是List的视图，对它修改不会影响到原List。

## ArrayList和Vector区别

|            | ArrayList | Vector |
| :--------: | :-------: | :----: |
| 线程安全性 |  不安全   |  安全  |
|  扩容机制  |   1.5倍   |  2倍   |

Stack的线程安全性使得单线程访问它的效率更低，而现在的事务不需要这么原子操作级别低，颗粒度细的线程安全了。

## Stack

Stack是Vector的子类，是操作受限的线性表（一端进出），不能设置初始容量，没有增量，常用方法有`push()`,`pop()`,`peek()`。要实现栈的数据结构，更常用的是LinkedList类。

# LinkedList

## LinkedList充当线性表

- LinkedList不仅是List接口的链表实现, 还是Deque接口的链表实现。
- LinkedList底层结构是一个双向链表 ，没有初始容量, 也不需要扩容。
- 有序, 允许null, 允许重复
- 线程不安全
- 可以根据下标增删改查，效率比ArrayList低。

## LinkedList充当其他数据结构

- 充当栈：`push()`,`pop()`,`peek()`
- 充当队列：`offer()`,`poll()`,`peek()`
- 双端队列：`offerFirst()`,`offerLast()`,`pollFirst()`,`pollLast()`,`peekFirst()`,`peekLast()`

