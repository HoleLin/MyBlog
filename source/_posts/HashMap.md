---
title: HashMap
date: 2021-05-25 22:46:55
index_img: /img/cover/Java.jpg
tags:
- Java
- JDK
- 源码
categories: 
- Java
---

#### HashMap源码(JDK8)

* 参考文献
  * [HashMap源码分析（jdk1.8，保证你能看懂）](https://zhuanlan.zhihu.com/p/79219960)

#### 理论

> HashMap最早是在JDK1.2中出现,直到JDK1.7一直没太大变化,但是到了JDK1.8突然进行了一个很大的改动.其中一个显著的改动就是:
>
> * 之前JDK1.7的存储结构是数组+链表
>
> * 到了JDK1.8变成了数组+链表+红黑树.
>
> 另外,HashMap是非线程安全的,也就是说多线程同时对HashMap中的某个元素进行增删改操作时,是不能保证数据的一致性的.

![image-20210417194922322](http://www.chenjunlin.vip/img/hashmap/image-20210417194922322.png)

> 在JDK1.7中,首先是把元素放在一个个数组里面,后来存放的数据元素越来越多,于是就出现了链表,对数组中的每个元素,都可以有一条链表来存储元素.这就是有名的"拉链式"存储方法
>
> 后来存储的元素越来越多,链表也越来越长,查找一个元素的时候效率不仅没有提高(链表不适合查找,适合增删),反而下降了不少,于是在JDK1.8中对这条链表进行了改进--使用红黑树. 将链表结构编程红黑树,原来JDK1.7的优点是增删效率提高,在JDK1.8中不仅增删的效率提高了,查找的效率也提升了.
>
> 核心是基于哈希值的**桶和链表**
>
> - 一般用数组实现桶
> - 发生哈希碰撞时，用链表来链接发生碰撞的元素
> - O(1)的平均查找、插入、删除时间
> - 致命缺点是哈希值的碰撞(collision)
>   - 哈希碰撞：元素通过哈希函数计算后，被映射到同一个桶中。例如上图中的桶1, 5中的元素就发生了哈希碰撞

![image-20210417195442294](http://www.chenjunlin.vip/img/hashmap/image-20210417195442294.png)

* **链表变成红黑树的条件:只有链表的长度不小于8,而且数组的长度不小于64的时候才会将链表转化为红黑树.**

##### 为什么选择红黑树?

* 红黑树是一个自平衡的二叉查找树,也就是说红黑树的查找效率是非常高的,查找效率会从链表的O(n)降低为O(logn);
* 引入红黑树就是为了**查找数据快**O(log n)，解决链表查询深度的问题
* 但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当**桶中元素大于8并且桶的个数大于64**的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢

> **红黑树性质**
>
> - 每个节点非红即黑
> - 根节点（root）是黑的
> - **不能有两个红色的节点连在一起**（黑色可以）
> - 每个叶节点（叶节点即树尾端NULL指针或NULL节点）都是黑的
> - 如果一个节点是红的，那么它的两儿子都是黑的
> - 对于任意节点而言，其到叶子点树NULL指针的每条路径都包含相同数目的黑节点
> - 每条路径**都包含相同的黑节点**

##### 为什么不一下子把整个链表转变为红黑树?

* 构造红黑树要比构造链表复杂,在链表的节点不多的时候,从整体的性能上看,数组+链表+红黑树的结构可能不一定比数组+链表的结构性能高;
* HashMap频繁的扩容,会造成底部红黑树不断的进行拆分和重组,这是非常耗时的,因此,也就是链表长度比较长的时候转变为红黑树才会效率显著;
* **链表变为红黑树的条件**：元素个数大于8同时桶的个数大于64
  - 当某个桶中的**元素个数大于8**时，会调用treeifyBin()方法，但并不一定就会变为红黑树
  - 当哈希表中**桶的个数大于64**时，才会真正进行让其转化为红黑树

#####  HashMap初始容量

* initialCapacity初始容量

  > 官方要求我们要输入一个2的N次幂的值，比如说2、4、8、16等等这些，但是我们忽然一个不小心，输入了一个20怎么办？没关系，虚拟机会根据你输入的值，找一个离20最近的2的N次幂的值，比如说16离他最近，就取16为初始容量。

##### HashMap负载因子

* loadFactor负载因子

  > 负载因子，默认值是0.75。负载因子表示一个散列表的空间的使用程度，有这样一个公式：initailCapacity*loadFactor=HashMap的容量。 所以负载因子越大则散列表的装填程度越高，也就是能容纳更多的元素，元素多了，链表大了，所以此时索引效率就会降低。反之，负载因子越小则链表中的数据量就越稀疏，此时会对空间造成烂费，但是此时索引效率高。
  >
  > 泊淞分布
  >
  > 当桶中元素到达8个的时候，概率已经变得非常小，也就是说用0.75作为加载因子，每个碰撞位置的链表长度超过８个是几乎不可能的。当桶中元素到达8个的时候，概率已经变得非常小，也就是说用0.75作为加载因子，每个碰撞位置的链表长度超过８个是几乎不可能的。

#### 构造方法

```java
    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
    
    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
        /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param initialCapacity the initial capacity 初始容量 要求要输入一个2的N次幂的值
     * @param loadFactor      the load factor 负载因子，默认值是0.75。
     * @throws IllegalArgumentException if the initial capacity is negative
     *                                  or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        // 如果传入的初始化容量小于0，则抛出异常
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        // 如果传入的容量大于最大容量，就初始化为最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        // 如果负载因子小于0，或者是非法的浮点数，抛出异常
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // 负载因子表示一个散列表的空间的使用程度 initialCapacity*loadFactor=HashMap的容量
        // 所以负载因子越大则散列表的装填程度越高，也就是能容纳更多的元素，元素多了，链表大了，
        // 所以此时索引效率就会降低。反之，负载因子越小则链表中的数据量就越稀疏，此时会对空间造成烂费，但是此时索引效率高。
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

   /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        // 经过下面的 或 和位移 运算， n最终各位都是1。
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        // 判断n是否越界，返回 2的n次方作为 table（哈希桶）的阈值
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

```java
    /**
     * Implements Map.putAll and Map constructor.
     *
     * @param m the map
     * @param evict false when initially constructing this map, else
     * true (relayed to method afterNodeInsertion).
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        // 获取该map的实际长度
        int s = m.size();
        if (s > 0) {
            // 判断table是否初始化
            if (table == null) { // pre-size
                /**
                 * 求出需要的容量，因为实际使用的长度=容量*0.75得来的，+1是因为小数相除，基本都不会是整数，容量大小不能为小数的，后面转换为int，多余的小数就要被丢掉，所以+1，
                 * 例如，map实际长度22，22/0.75=29.3,所需要的容量肯定为30，有人会问如果刚刚好除得整数呢，除得整数的话，容量大小多1也没什么影响
                 **/
                float ft = ((float)s / loadFactor) + 1.0F;
                //判断该容量大小是否超出上限。
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                //对临界值进行初始化，tableSizeFor(t)这个方法会返回大于t值的，且离其最近的2次幂，例如t为29，则返回的值是32
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            //如果table已经初始化，则进行扩容操作，resize()就是扩容。
            else if (s > threshold)
                resize();
            //遍历，把map中的数据转到hashMap中。
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

#### 存储元素(put)

![image-20210417201703380](http://www.chenjunlin.vip/img/hashmap/image-20210417201703380.png)

```java
public class Main {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("k1", "v1");
    }
}
```

##### 第一步：调用put方法传入键值对

```java
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        // 1.先根据键计算Hash
        return putVal(hash(key), key, value, false, true);
    }
```

##### 第二步：使用hash算法计算hash值

```java
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
        // hash值其实就是通过hashcode与16异或计算的
        // 通过异或运算能够是的计算出来的hash比较均匀，不容易出现冲突
        // 在数据结构中，我们处理hash冲突常使用的方法有：开发定址法、再哈希法、链地址法、建立公共溢出区。而hashMap中处理hash冲突的方法就是链地址法。
        // 这种方法的基本思想是将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。链地址法适用于经常进行插入和删除的情况。
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

```java
    /**
     * Implements Map.put and related methods.
     *
     * @param hash         hash for key 调用了hash方法计算得到的hash值
     * @param key          the key 传入的key的值
     * @param value        the value to put 传入的value的值
     * @param onlyIfAbsent if true, don't change existing value true表示: 当键相同时,不修改已存在的值
     * @param evict        if false, the table is in creation mode. false表示: 数组就处于创建模式,一般为true
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // Node<K,V>[] tab表示的就是数组
        // Node<K,V> p表示的是当前插入的节点
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 若数组是空的,则调用resize()方法来创建一个新的数组
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 第三步：根据hash值确定存放的位置，判断是否和其他键值对位置发生了冲突
        // i表示在数组中插入的位置,计算的方式为(n-1)&hash,在这里要判断插入的位置是否是冲突的
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 第四步：若没有发生冲突，直接存放在数组中即可
            // 若不冲突,就直接newNode,插入到数组中即可
            tab[i] = newNode(hash, key, value, null);
        // 出现冲突
        else {
            Node<K,V> e; K k;
            // 第五步：若发生了冲突，还要判断此时的数据结构是什么？
            // 这里判断table[i]中的元素是否与插入的key一样,若相同那么就使用插入的值p替换掉就的值e;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 判断插入的数据结构是红黑树还是链表
            else if (p instanceof TreeNode)
                // 第六步：若此时的数据结构是红黑树，那就直接插入红黑树中
                // 若是红黑树,那么直接调用putTreeVal(),将节点加到红黑树中
                //为红黑树的节点，则在红黑树中进行添加，如果该节点已经存在，则返回该节点（不为null），该值很重要，用来判断put操作是否成功，如果添加成功返回null
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 若是链表,遍历链表
                for (int binCount = 0; ; ++binCount) {
                    // 如果找到尾部，则表明添加的key-value没有重复，在尾部进行添加
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 第七步：若此时的数据结构是链表，判断插入之后是否大于等于8
                        // 插入的时候会判断当前链表的长度是否大于阀值TREEIFY_THRESHOLD - 1(8,从0开始)
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            // 第八步：插入之后大于8了，就要先调整为红黑树，在插入
                            // 转变为红黑树
                            treeifyBin(tab, hash);
                        // 第九步：插入之后不大于8，那么就直接插入到链表尾部即可。
                        break;
                    }
                    //如果链表中有重复的key，e则为当前重复的节点，结束循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 如果存在了直接使用新的value替换掉旧的
                    p = e;
                }
            }
            // 有重复的key,则用待插入的值进行覆盖,返回旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        // 修改次数+1
        ++modCount;
        // 插入成功之后,还要判断一下实际存在的键的数量size是否大于阀值threshold,若大于则需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

```java
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;

        // 如果当前哈希表中桶的数目，小于最小树化容量，就调用resize()方法进行扩容
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();

        // 当桶中元素个数大于8，且桶的个数大于64时，进行树化
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```



#### 扩容(resize)

![image-20210417204637243](http://www.chenjunlin.vip/img/hashmap/image-20210417204637243.png)

> HashMap扩容就是先计算,新的hash表容量和新的容量阀值,然后初始化新的hash表,将旧的键值对重新映射到新的hash表里面.若在就的hash表里面涉及到红黑树,那么在映射到新的hash表中还涉及红黑树的拆分.

```java
    /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        // 把没插入之前的哈希数组做oldTab
        Node<K,V>[] oldTab = table;
        //o ldTab的长度
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // oldTab的临界值
        int oldThr = threshold;
        // 初始化new的长度和临界值
        int newCap, newThr = 0;
        // oldCap > 0也就是说不是首次初始化，因为hashMap用的是懒加载
        if (oldCap > 0) {
            // 若旧的容量大于数组最大容量,那么就将阀值设置为整数的最大值
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 临界值为整数的最大值
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 若没有超过,则扩容为原来的两倍(oldCap << 1,右移一位)
            // 标记##，其它情况，扩容两倍，并且扩容后的长度要小于最大值，oldTab长度也要大于16
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                //临界值也扩容为oldTab的临界值2倍
                newThr = oldThr << 1; // double threshold
        }
        // 设置阀值
        // 如果oldCap<0，但是已经初始化了，像把元素删除完之后的情况，那么它的临界值肯定还存在，
        // 如果是首次初始化，它的临界值则为0
        else if (oldThr > 0) // initial capacity was placed in threshold
            // 若阀值已经初始化过了,那么就直接使用旧的阀值
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 没有初始化,初始化一个新的数组容量和新的阀值
            newCap = DEFAULT_INITIAL_CAPACITY;
            // 临界值等于容量*加载因子
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 此处的if为上面标记##的补充，也就是初始化时容量小于默认值16的，此时newThr没有赋值
        if (newThr == 0) {
            // new的临界值
            float ft = (float)newCap * loadFactor;
            // 判断是否new容量是否大于最大值，临界值是否大于最大值
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 把上面各种情况分析出的临界值，在此处真正进行改变，也就是容量和临界值都改变了。
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 旧数组不为空,将旧数据迁移到新数组中
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                // 临时变量
                Node<K,V> e;
                // 当前哈希桶的位置值不为null，也就是数组下标处有值，因为有值表示可能会发生冲突
                if ((e = oldTab[j]) != null) {
                    // 把已经赋值之后的变量置位null，当然是为了好回收，释放内存
                    oldTab[j] = null;
                    // 只有一个节点,通过索引位置直接映射
                    if (e.next == null)
                        //把该变量的值存入newCap中，e.hash & (newCap - 1)并不等于j
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果是红黑树,需要进行树拆分然后映射
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 若是多个节点的链表,将原来链表拆分为两个链表
                        // 因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位=  low位+原哈希桶容量
                        // 低位链表的头结点、尾节点
                        Node<K,V> loHead = null, loTail = null;
                        // 高位链表的头节点、尾节点
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 扩容后长度为原hash表的2倍，于是把hash表分为两半，分为低位和高位，如果能把原链表的键值对， 一半放在低位，一半放在高位，而且是通过e.hash & oldCap == 0来判断，
                            // n = 16，二进制为10000，第5位为1，e.hash & oldCap 是否等于0就取决于e.hash第5 位是0还是1，这就相当于有50%的概率放在新hash表低位，50%的概率放在新hash表高位。
                            // 扩容后,若hash值新增参与运算的位=0,那么元素在扩容后的位置=原始位置
                            // hash值新增参与运算的位: 把hash值转变成二进制数字，新增参与运算的位就是倒数第五位。
                            // 这里又是一个利用位运算 代替常规运算的高效点： 利用哈希值 与 旧的容量，可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 扩容后,若hash值新增参与运算的位=1,那么元素在扩容后的位置=原始位置+扩容后的旧位置
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 低位链表存于原索引
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 高位链表存于原索引加上原hash桶长度偏移量
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

#### 移除元素(remove)

```java
    /**
     * Removes the mapping for the specified key from this map if present.
     *
     * @param  key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V remove(Object key) {
        // 临时变量
        Node<K,V> e;
        // 调用removeNode(hash(key), key, null, false, true)进行删除，
        // 第三个value为null，表示，把key的节点直接都删除了，不需要用到值，如果设为值，则还需要去进行查找操作
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
```

```java
    /**
     * Implements Map.remove and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to match if matchValue, else ignored
     * @param matchValue if true only remove if value is equal 为true的话，则表示删除它key对应的value，不删除key
     * @param movable if false do not move other nodes while removing 为false，则表示删除后，不移动节点
     * @return the node, or null if none
     */
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        // tab 哈希数组，p 数组下标的节点，n 长度，index 当前数组下标
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        // 哈希数组不为null，且长度大于0，然后获得到要删除key的节点所在是数组下标位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            // node 存储要删除的节点，e 临时变量，k 当前节点的key，v 当前节点的value
            Node<K,V> node = null, e; K k; V v;
            // 如果数组下标的节点正好是要删除的节点，把值赋给临时变量node
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            // 也就是要删除的节点，在链表或者红黑树上，先判断是否为红黑树的节点
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    // 遍历红黑树，找到该节点并返回
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        //表示为链表节点，一样的遍历找到该节点
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        // 注意，如果进入了链表中的遍历，那么此处的p不再是数组下标的节点，而是要删除结点的上一个结点
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            // 找到要删除的节点后，判断!matchValue，我们正常的remove删除，!matchValue都为true
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                // 如果删除的节点是红黑树结构，则去红黑树中删除
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                // 如果是链表结构，且删除的节点为数组下标节点，也就是头结点，直接让下一个作为头
                else if (node == p)
                    tab[index] = node.next;
                // 为链表结构，删除的节点在链表中，把要删除的下一个结点设为上一个结点的下一个节点
                else
                    p.next = node.next;
                // 修改计数器
                ++modCount;
                // 长度减一
                --size;
                // 此方法在hashMap中是为了让子类去实现，主要是对删除结点后的链表关系进行处理
                afterNodeRemoval(node);
                // 返回删除的节点
                return node;
            }
        }
        // 返回null则表示没有该节点，删除失败
        return null;
    }
```

#### 获取元素(get)

```java
    /**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
     * key.equals(k))}, then this method returns {@code v}; otherwise
     * it returns {@code null}.  (There can be at most one such mapping.)
     *
     * <p>A return value of {@code null} does not <i>necessarily</i>
     * indicate that the map contains no mapping for the key; it's also
     * possible that the map explicitly maps the key to {@code null}.
     * The {@link #containsKey containsKey} operation may be used to
     * distinguish these two cases.
     *
     * @see #put(Object, Object)
     */
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

```java
    /**
     * Implements Map.get and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        // first 头结点，e 临时变量，n 长度,k key
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 头结点也就是数组下标的节点
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 如果是头结点，则直接返回头结点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            // 不是头结点
            if ((e = first.next) != null) {
                // 判断是否是红黑树结构
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 去红黑树中找，然后返回
                do {
                    // 链表节点，一样遍历链表，找到该节点并返回
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        // 找不到，表示不存在该节点
        return null;
    }
```











