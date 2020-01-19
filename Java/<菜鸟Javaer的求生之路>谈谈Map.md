# 谈谈Map



作为Javaer，对于Map这个单词绝对不会陌生，无论是开发过程中还是出去面试的时候，都会经常遇到，而最频繁使用和面试提问的无非这么几个，HashMap,  HashTable, ConcurrentHashMap。那么本文就针对这几个知识点做一个归纳和总结。

 

## 从HashMap说起

HashMap是上面提到的几个Map中使用频率最高的了，毕竟需要考虑到多线程并发的场景并不算太多。下面是Map的一个关系图，大家了解一下即可。

![Map](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/UTOOLS1579397021808.png)

HashMap在Java8之前和之后有很大差别，在Java8以前，它的数据结构是数组+链表的形式，8以后就变成了数组+链表+红黑树的结构。它的key是保存在一个Set里面的，也就是有去重的功能，values是存在一个Collections里面。

![HashMap_Java7](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/UTOOLS1579231511991.png)

![HashMap_Java8](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/UTOOLS1579231597435.png)

HashMap里的数组每个元素存放的是key-value形式的实例，Java7里面叫做Entry，8里面叫Node。这个Node里面包含了hash值，键值对，下一个节点next这几个属性组成。数组被分为一个个bucket，也就是桶，通过hash值决定了键值对在这个数组中的寻址，hash值相同的则以链表的形式存储，链表长度超过阈值就转成红黑树。那么先来看看HashMap的put操作，

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //为空则初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 算出键值对在table中的具体位置，没有就new一个node
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // 如果存在
        Node<K,V> e; K k;
        //一样就替换
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //树化了就用树的形式保存
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //链表的形式插入元素
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
        // 存在就更新
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //超过阈值就扩容
    if (++size > threshold)
        resize();
    // 这是为了继承HashMap的LinkedHashMap类服务的，用来回调移除最早放入Map的对象
    afterNodeInsertion(evict);
    return null;
}
```



那么总结一下就是：

1. 若HashMap未被初始化，则进行初始化操作
2. 对Key求Hash值，依据Hash值计算下标
3. 若未发生碰撞，则直接放入桶中
4. 若发生碰撞，则以链表的方式链接到后面
5. 若链表长度超过阈值，且HashMap元素超过最低树化容量，则将链表转成红黑树
6. 若节点已经存在，则用新值替换旧值
7. 若桶满了，就需要resize（扩容2倍后重排）

这个put的操作引申出几个知识点，首先，

> **HashMap的初始容量是多少？为什么设置成这个值呢？**

翻看源码我们可以看到有这么一个变量DEFAULT_INITIAL_CAPACITY，

```Java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

这个就是HashMap的初始容量，也就是16，为啥用位运算这么骚的写法是因为位运算比算数计算的效率要高。那么为啥用16？看看上面说的下标计算的公式: index = HashCode（Key） & （Length- 1）,当长度为16时候，Length-1的二进制就是1111,是一个所有位都为1的数，而且看上述注释，建议的HashMap的初始长度都是2的幂次方，这种情况下，index的结果等同于HashCode后几位的值。那么**只要输入的HashCode本身分布均匀，Hash算法的结果就是均匀的**。

> 另一个问题，Java8里面引入了红黑树，**当链表达到一定长度的时候会转换成红黑树，引入红黑树的好处是什么？这个变换的阈值是多少，为什么是这个值？**

当元素put的时候，首先是要根据哈希函数和长度计算下标的，但**即使哈希函数取得再好，也很难达到元素百分百均匀分布**，那么就有可能**导致 HashMap 中有大量的元素都存放到同一个桶中时，这个桶下有一条长长的链表**，这个时候 HashMap 就相当于一个单链表，假如单链表有 n 个元素，遍历的时间复杂度就是 O(n)，完全失去了它的优势。

引入红黑树后，但链表长度大于8时，就会转换成红黑树，若链表元素个数小于等于6时，树结构还原成链表。至于为什么是8，我看到过两个说法，一个是**因为红黑树的平均查找长度是log(n)，长度为8的时候，平均查找长度为3，如果继续使用链表，平均查找长度为8/2=4，这才有转换为树的必要。链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短**。另一个说法是**根据泊松分布，在负载因子默认为0.75的时候，单个hash槽内元素个数为8的概率小于百万分之一，所以将7作为一个分水岭，等于7的时候不转换，大于等于8的时候才进行转换，小于等于6的时候就化为链表**。两种都有道理我觉得哪一种都是可以的。

> **当桶满了的时候，HashMap会进行扩容resize，它是何时并且如何扩容的呢？**

当桶的容量达到长度乘以负载因子的时候就会进行扩容，默认的负载因子为0.75。

```Java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

首先，它会创建一个新的Entry空数组，长度是原数组的2倍。然后遍历原Entry数组，把所有的Entry重新Hash到新数组。这里要进行ReHash的原因是我们知道下标的计算是跟长度有关的，长度不一样了，那么index计算的结果自然也不一样，因此需要重新Hash到新数组，rehash是一个比较耗时的过程。

> **接下来还是插入相关的问题，新的Entry节点在插入链表的时候，是怎么插入的？**

这个问题我是在一篇博客上看到的，之前的确从未考虑过这个问题。**Java8之前是头插法**，就是说新来的值会取代原有的值，原有的值就顺推到链表中去，就像上面的例子一样，因为写这个代码的作者认为后来的值被查找的可能性更大一点，提升查找的效率，**在Java8之后，都是所用尾部插入了。** 由于在扩容的时候会存在条件竞争,如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。用头插法的话，假设原来链表是A指向B指向C，新的链表可能出现B指向A但A同时也指向B。用尾插的方法扩容保持链表元素原油的顺序，就不会出现这种链表成环的问题了。

> put的时候会先判断是否碰撞，那么如何减少碰撞呢？

一般有两个方法，一个是使用扰动函数，让不同对象返回不同hashcode；一个是使用final对象，防止键值改变，并采用合适的equeals方法和hashCode方法，减少碰撞的发生。

那么对于get方法因为比较简单就不做太多详细解释，其实就是根据key的hashcode算出元素在数组中的下标,之后遍历Entry对象链表,直到找到元素为止。

### SynchronizedMap

这里额外再提一个Map，也是解决HashMap多线程安全的一种方案。那就是Collections.synchronizedMap(Map)。它会返回一个线程安全的SynchronizedMap的实例。它里面维护了一个排斥锁mutex。对于里面的public方法，使用了synchronized对mutex进行加锁。多线程环境下串行化执行，效率低下。

![SynchronizedMap](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/UTOOLS1579228791046.png)

上面就是一些关于HashMap的一些简单的知识点，我这里整理的其实也不算太多但还是很实用的（我就知道这么多）。

## HashTable

关于HashTable其实说不了太多，因为说实话反正我是从来没用过。都知道它线程安全，但它用的手段很简单粗暴。涉及到修改的地方使用了synchronized修饰，以串行化方式运行，效率比较低下。它和上面说的SynchronizedMap实现线程安全的方式很接近，只是锁的对象不一样。

## ConcurrentHashMap

那么还是来谈谈另一个还挺常见的ConcurrentHashMap,它现在的数据结构和原来的也是不一样的，早期也是数组+链表，现在是数组+链表+红黑树。 

在Java8以前，由Segment数组、HashEntry组成，通过分段锁Segment来实现线程安全，ConcurrentHashMap内部维护了Segment内部类，继承了RetrantLock。它将锁一段一段的存储，给每一段数据分配一个锁，也就是segment，当一个线程访问一个锁时，其他线程也可以访问其他segment的数据，不会被阻塞，默认分配16个segment。也就是理论上它的效率比HashTable提高了16倍。而HashEntry跟HashMap差不多，只是它用volatile修饰了数据的value还有下一个节点next。

到了Java8，它就不再是使用Segment分段锁，而是使用了CAS+synchronized来保证线程安全。

![ConcurrentHashMap_Java8](https://leafw-blog-pic.oss-cn-hangzhou.aliyuncs.com/UTOOLS1579229640162.png)

synchronized锁住当前链表或者红黑树的首节点，这样只要哈希不冲突，就不会出现并发问题。

```java
/**
 * Table initialization and resizing control.  When negative, the
 * table is being initialized or resized: -1 for initialization,
 * else -(1 + the number of active resizing threads).  Otherwise,
 * when table is null, holds the initial table size to use upon
 * creation, or 0 for default. After initialization, holds the
 * next element count value upon which to resize the table.
 */
private transient volatile int sizeCtl;
```

ConcurrentHashMap和HashMap的参数差不多，但有些特有的，比如sizeCtl。它是哈希表初始化或扩容时的一个控制位标识量，负数代表正在初始化或正在扩容操作。同样的，我们也看看它的put操作。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        // 键值都不能为null
        if (key == null || value == null) throw new NullPointerException();
        //计算key的hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        // 数组元素的更新，使用CAS，所以需要不断失败重试
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                // 初始化
                tab = initTable();
            //找到f，即链表或者红黑树的头节点，没有就添加
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //CAS添加，失败break
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // 如果正在移动元素，就协助扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                //发生hash碰撞，锁定链表或者红黑树的头节点f
                V oldVal = null;
                synchronized (f) {
                    // 判断f是否时链表的头节点
                    // fh就是头节点的hash值
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            //如果是，初始化链表的计数器
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                //节点存在就更新
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 不更新就插入
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //头节点是红黑树的头，用红黑树的方式插入
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    //链表长度达到了8，则转换成树结构
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // ConcurrentHashMap的size+1
        addCount(1L, binCount);
        return null;
    }
```

上面是整段代码的解释，总结一下就下面几个步骤：

1. 判断Node[]数组是否初始化，没有则进行初始化操作
2. 通过hash定位数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头节点），添加失败则进入下次循环
3. 检查到内部正在扩容，就帮助它一块扩容
4. 如果头节点f！=null,则使用synchronized锁住f元素（链表/红黑二叉树的头元素）
   - 如果是Node（链表结构）则进行链表的添加操作
   - 如果是TreeNode结构则执行树添加操作
5. 判断链表长度已经达到临界值8，这个8可以自己调整，当节点数超过这个值就把链表转换为树结构

使用这种方式相对于Segment而言，锁拆的更细。首先使用无锁操作CAS插入节点，失败则循环重试。若头节点存在，则尝试获取头节点的同步锁再进行操作。至于get操作也比较简单，也是根据hashcode寻址，如果就在桶上就直接返回值，不是的话就按照链表或者红黑树的方式遍历获取值。

## HashMap、HashTable以及ConcurrentHashMap的区别

大致讲述了他们三个的基础知识，那么来总结下它们区别。这里做了个list大家可以看看。

- HashMap线程不安全，数组+链表+红黑树
- HashTable线程安全，锁住整个对象，数组+链表
- ConcurrentHashMap线程安全，CAS+同步锁，数组+链表+红黑树
- HashMap的key，value均可为null，其他两个不可以
  - HashTable使用的是安全失败机制（fail-safe），这种机制会使你此次读到的数据不一定是最新的数据。如果你使用null值，就会使得其无法判断对应的key是不存在还是为空，因为你无法再调用一次contain(key）来对key是否存在进行判断，ConcurrentHashMap同理
- HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75。
- 当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1。
- HashMap 中的 Iterator 迭代器是 fail-fast 的，而 Hashtable 的 Enumerator 不是 fail-fast 的。
  - 快速失败（fail—fast）是java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。

以上就是关于Map相关的一些知识点，里面很多引申的知识点我都没有再往深里说，比如里面使用到的红黑树数据结构，volatile关键字，CAS等等，这个在后面会针对相应的知识点再继续梳理。