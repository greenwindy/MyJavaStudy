[TOC]
# HashMap

## 特点以及源码分析

### 1. HashMap的底层结构

HashMap的底层维护了一个数组。数组默认初始容量16， 默认的加载因子是0.75。HashMap能存储的元素数量 <= 底层数组长度 * 加载因子，超过了就要扩容，扩容机制: 扩为原来的2倍。

```java	
	transient Node<K,V>[] table;// hashmap底层所维护的数组, 是一个Node类型的数组

	static final float DEFAULT_LOAD_FACTOR = 0.75f;// 默认加载因子
	final float loadFactor;// 加载因子(饱和度)

	public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }
```

```java
	int threshold;// 标记阈值 = 数组长度 * 加载因子
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
```

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        ......
        if (++size > threshold)
            resize();
        ......
     }

	final Node<K,V>[] resize() {
        ......
        int oldThr = threshold;
        
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
              
            }
            // newCap = 2旧容量 = 32
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // newThr = 2旧阈值 = 24
                newThr = oldThr << 1; 
        }
        ......
    }
```

加载因子可以通过构造方法给定，建议0.5~1。如果在创建HashMap的时候给定长度, 那么HashMap会根据给定的长度, 计算一个大于等于该长度的最小的2的幂值, 用来充当底层数组的长度。

为什么要求HashMap的底层数组长度是2的幂值？HashMap的散列方式是取模。取模(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。” 并且采用二进制位操作 &，相对于%能够提高运算效率。源代码：`tab[i = (n - 1) & hash]`。

### 2. 元素k值的计算及插入规则

HashMap的底层结构还有链表和红黑树，其中红黑树是jdk1.8加入的。当要存储的位置冲突时，该位置就以链表和红黑树的结构扩展。HashMap无序，因为插入的位置要通过散列重新计算。可以存储null key，把它当作0来看待，但不能存储重复的key。HashMap判断key是否重复比较复杂，首先它要计算key的hash值，通过key的hashCode异或上hashCode向右移位16次的结果得到。如果hash值相同，并且key相等或者equals，那么就判断key重复，会把旧值用新值替换掉。此时`put()`方法会返回旧值（没有被替换会返回null）。HashMap插入的位置通过hash值对数组长度进行取模运算得到。链表中元素大于8个以后，再添加新元素会把链表转化成红黑树。

```java
	static final int TREEIFY_THRESHOLD = 8;//树化阈值

	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        ......
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        
        // i = (n - 1) & hash:  取模
        // tab[i] :  key值经过hash计算, 又经过取模之后的对应数组下标位置
        // p =  tab[i]
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 如果这个散列位置, 没有内容, 存进去
            tab[i] = newNode(hash, key, value, null);
        
        // 散列位置如果有元素
        else {
            Node<K,V> e; 
            K k;
            
            // (h = key.hashCode) ^ (h >>> 16): 有可能导致, 两个完全不同的对象hash值一样
            // p.hash == hash :  1, 两个key-value数据的, hash值是否一样
            // (k = p.key) == key || (key != null && key.equals(k))
            // key是否直接相等 或者 相equals
            
            // (hash是否一样   &&   (key直接相等 ||  key相equals  ))
            // 如果判断条件为真:  把新值 替换 旧值
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            
            // 判断条件为假，当前重复位置不是重复key
            // p是不是红黑树的根节点
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 已经存储的元素是一个单链表
                // p是链表中第一个元素，而从p的next开始计数，从0计数到6，binCount>=7时树化
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
     }	
```

但是当数组长度小于64的时候, 会优先扩容, 而不是转化为红黑树，如果数组长度超过64, 必定由链表转化为红黑树。

### 3. 扩容和红黑树转化成链表

扩容时一个位置上只有一个元素，那么会重新计算新位置。链表和红黑树会进行拆分，一部分留在原来的位置，一部分下标加原本数组长度。

```java
	final Node<K,V>[] resize() {
        ......
        // 创建一个新数组
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
         
        
         // 把旧数组的数据 转移到新数组中去
         if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                
                Node<K,V> e;
                
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 单个点：对新数组长度重新取模运算得到新位置
                    if (e.next == null)  
                        newTab[e.hash & (newCap - 1)] = e;
                    
                   	// 一个树拆分成两部分：原本下标位置/原本下标 + 原本数组长度的位置
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    
                    // 链表同样拆成两部分
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

树在经过分裂后，如果树的元素个数元素个数 <= 6 ，就会由红黑树转化为链表。此外删除元素时导致红黑树的根节点, 根节点的左右结点, 根节点的左节点的左节点, 这四个结点，有一个变为null，那么就会由红黑树转化为链表。

### 4. 其他

HashMap是线程不安全的。不要通过引用去修改存入HashMap中的key，那样会使得HashMap无法对这一个键值对进行操作。

## 常用方法

|       作用       |      返回值类型       |                   方法                    |
| :--------------: | :-------------------: | :---------------------------------------: |
|  获取键对应的值  |          `V`          |             `get(Object key)`             |
|       插入       |          `V`          |          `put(K key,  V value)`           |
|    是否包含键    |       `boolean`       |         `containsKey(Object key)`         |
|    是否包含值    |       `boolean`       |       `containsValue(Object value)`       |
| 返回键值映射视图 | `Set<Map.Entry<K,V>>` |               `entrySet()`                |
|    返回键视图    |       `Set<K>`        |                `keySet()`                 |
| 删除指定键的映射 |          `V`          |           `remove(Object key)`            |
|   删除指定映射   |       `boolean`       |    `remove(Object key, Object value)`     |
|       替换       |       `boolean`       | `replace(K key,  V oldValue, V newValue)` |

# LinkedHashMap

LinkedHashMap 是HashMap的子类，特性和HashMap几乎一样。不同的是LinkedHashMap是有序的: 他的底层结构,在HashMap基础上, 又额外维护了一个双向链表。

# TreeMap

他是Map的一个树实现，底层结构是红黑树，最重要特征是key大小有序，不能存储’’重复’’元素key，不允许存储null-- key，线程不安全。如果使用默认的构造方法，那要求存储元素的顺序, 应该实现 ‘自然顺序’。也可以在构造方法的参数中，提供比较器，如果比较器和自然顺序同时存在, 优先使用比较器。

## 比较顺序

- 默认比较顺序：字典序
- key的类实现Comparator接口
- 传入Comparator对象。例：

```java
TreeMap<User, String> map = new TreeMap<>(new Comparator<User>() {
            @Override
            public int compare(User user1, User user2) {
                int com = user2.name.compareTo(user1.name);
                com = com == 0 ? user2.age.compareTo(user1.age): com;
                return com;
            }
        });
```

# HashMap与HashTable

|                      | HashMap                                               | HashTable                              |
| -------------------- | ----------------------------------------------------- | -------------------------------------- |
| 线程是否安全         | 不安全                                                | 安全                                   |
| 效率                 | 高                                                    |                                        |
| Null Key和Null Value | 可以有1个Null key以及多个Null Value                   | NullPointerException                   |
| 容量和容量扩充机制   | 默认初始容量16，使用大于等于给定初始容量的最小2的幂次 | 默认初始容量11，直接使用给定的初始容量 |
| 底层数据结构         | jdk1.8后加入红黑树来解决哈希冲突，机制是……            | 没有这个机制                           |

# Set

- Set是collection集合子接口，不允许重复。

- 子类底层持有对应的Map对象，拥有相同的特点。

|              |   HashSet   |   LinkedHashSet   |   TreeSet   |
| :----------: | :---------: | :---------------: | :---------: |
|     底层     | HashMap对象 | LinkedHashMap对象 | TreeMap对象 |
|   是否有序   |    无序     |       有序        |  大小有序   |
| 线程是否安全 |   不安全    |      不安全       |   不安全    |
| 是否允许null |    允许     |       允许        |   不允许    |



