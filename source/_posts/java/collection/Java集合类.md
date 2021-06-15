---
title: Java集合类
mermaid: true
date: 2021-06-14 00:21:17
cover: /img/cover/Java.jpg
tags:
- 集合类
categories:
- Java
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [3W 字详解 Java 集合](https://mp.weixin.qq.com/s/2yke6e-bfEYGtTHxH-31HQ)

###  集合框架

> 所有集合实现类的最顶层接口为`Iterable`和`Collection`接口，再向下`Collection`分为了三种不同的形式，分别是`List`，`Queue`和`Set`接口，然后就是对应的不同的实现方式。

<img src="http://www.chenjunlin.vip/img/java/collection/集合类总类图.png" alt="img" style="zoom:80%;" />

#### **顶层接口`Iterable`**

```java
public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

##### `Iterator `与` ListIterator`详解

###### `Iterator`

* Iterator是一个接口，它是集合的迭代器。集合可以通过Iterator去遍历集合中的元素。

* Iterator提供的API接口如下：
  * `boolean hasNext()`：判断集合里是否存在下一个元素。如果有，`hasNext()`方法返回 true。	
  * `Object next()`：返回集合里下一个元素。
  * `void remove()`：删除集合里上一次next方法返回的元素。
* `Iterator`只能单向移动。
* `Iterator.remove()`是唯一安全的方式来在迭代过程中修改集合；如果在迭代过程中以任何其它的方式修改了基本集合将会产生未知的行为。而且每调用一次`next()`方法，`remove()`方法只能被调用一次，如果违反这个规则将抛出一个异常。

###### `ListIterator`

* **`ListIterator`是一个功能更加强大的迭代器, 它继承于Iterator接口，**只能用于各种List类型的访问。可以通过调用`listIterator()`方法产生一个指向List开始处的`ListIterator`, 还可以调用`listIterator(n)`方法创建一个一开始就指向列表索引为n的元素处的`ListIterator`。
* 由以上定义我们可以推出`ListIterator`可以:
  - 双向移动（向前/向后遍历）.
  - 产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引.
  - 可以使用`set()`方法替换它访问过的最后一个元素.
  - 可以使用`add()`方法在`next()`方法返回的元素之前或`previous()`方法返回的元素之后插入一个元素.

#### `Collection`

```java
public interface Collection<E> extends Iterable<E> {
    // Query Operations

    int size();
    
    boolean isEmpty();

    boolean contains(Object o);

    Iterator<E> iterator();

    Object[] toArray();

    <T> T[] toArray(T[] a);

    // Modification Operations

    boolean add(E e);

    boolean remove(Object o);

    // Bulk Operations

    boolean containsAll(Collection<?> c);

    boolean addAll(Collection<? extends E> c);
  
    boolean removeAll(Collection<?> c);

    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    boolean retainAll(Collection<?> c);

    void clear();

    // Comparison and hashing

    boolean equals(Object o);

    int hashCode();

    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}

```

#### Set、List和Map可以看做集合的三大类

- List集合是有序集合，集合中的元素可以重复，访问集合中的元素可以根据元素的索引来访问。
- Set集合是无序集合，集合中的元素不可以重复，访问集合中的元素只能根据元素本身来访问（也是集合里元素不允许重复的原因）。
- Map集合中保存Key-value对形式的元素，访问时只能根据每项元素的key来访问其value。

### `List`

> List表示一串有序的集合，和Collection接口含义不同的是List突出有序的含义。

```java
public interface List<E> extends Collection<E> {
    // Query Operations

    int size();

    boolean isEmpty();

    boolean contains(Object o);

    Iterator<E> iterator();

    Object[] toArray();

    <T> T[] toArray(T[] a);

    // Modification Operations

    boolean add(E e);

    boolean remove(Object o);

    // Bulk Modification Operations

    boolean containsAll(Collection<?> c);

    boolean addAll(Collection<? extends E> c);
    
    boolean removeAll(Collection<?> c);    
    
    boolean retainAll(Collection<?> c);
    
    void clear();

    // Comparison and hashing
  
    boolean equals(Object o);

    int hashCode();
    
    //============================
    // 		List独有方法
    //============================
    boolean addAll(int index, Collection<? extends E> c);

    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }

    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    // Positional Access Operations

    E get(int index);

    E set(int index, E element);

    void add(int index, E element);
 
    E remove(int index);

    // Search Operations

    int indexOf(Object o);

    int lastIndexOf(Object o);

    // List Iterators

    ListIterator<E> listIterator();

    ListIterator<E> listIterator(int index);

    // View

    List<E> subList(int fromIndex, int toIndex);

    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }
}

```

#### `Collection`与`List`的区别

* `Collection`和`List`最大的区别是`Collection`是无序的,不支持索引操作,而`List`是有序的.`Collection`没有顺序的概念;
* `List`中`Iterator`为`ListIterator`;
* `List`可以进行排序,`List`接口支持使用`sort`方法;
* 两者的`spliterator`操作不一样;

#### `ArrayList`

> `ArrayList`是List接口最常用的一个实现类，支持List接口的一些列操作

<img src="http://www.chenjunlin.vip/img/java/collection/ArrayList类图.png" alt="img" style="zoom:80%;" />

##### `ArrayList`常量

```ajva
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

	// 真正存储元素的数组
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```

##### `ArrayList`构造函数

```java
    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
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

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

##### ``ArrayList``添加元素

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


```

> 注意`ArrayList`中有一个`modCount`的属性，表示该实例修改的次数。（所有集合中都有`modCount`这样一个记录修改次数的属性），每次增改添加都会增加一次该`ArrayList`修改次数，而上边的add(E e)方法是将新元素添加到list尾部。

##### ``ArrayList``扩容

```java
 	private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

	private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // 可见当初始化的list是一个空ArrayList的时候，会直接扩容到DEFAULT_CAPACITY，该值大小是一个默认值10。而当添加进ArrayList中的元素超过了数组能存放的最大值就会进行扩容。
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
        // 采用右移运算，就是原来的一般，所以是扩容1.5倍。
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

##### **为什么`elementData`用`transient`修饰？**

* `transient`的作用是该属性不参与序列化。
2. `ArrayLis`t继承了标示序列化的`Serializable`接口
3. 对`ArrayList`序列化的过程中进行了读写安全控制。

```java
    /**
     * Save the state of the <tt>ArrayList</tt> instance to a stream (that
     * is, serialize it).
     *
     * @serialData The length of the array backing the <tt>ArrayList</tt>
     *             instance is emitted (int), followed by all of its elements
     *             (each an <tt>Object</tt>) in the proper order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

> 在序列化方法`writeObject()`方法中可以看到，先用默认写方法，然后将size写出，最后遍历写出`elementData`，因为该变量是transient修饰的，所有进行手动写出，这样它也会被序列化了。那是不是多此一举呢？
>
> ```java
> protected transient int modCount = 0;
> ```
>
> 当然不是，其中有一个关键的`modCount`, 该变量是记录list修改的次数的，当写入完之后如果发现修改次数和开始序列化前不一致就会抛出异常，序列化失败。这样就保证了序列化过程中是未经修改的数据,保证了序列化安全。（Java集合中都是这样实现）

#### `LinkedList`

<img src="http://www.chenjunlin.vip/img/java/collection/LinkedList类图.png" alt="img" style="zoom:80%;" />

> `LinkedList`是一种链表结构,`LinkedList`既是List接口的实现也是Queue的实现（`Dequ`e），故其和`ArrayList`相比`LinkedList`支持的功能更多，其可视作队列来使用

##### `LinkedList`常量

> `LinkedList`由一个头节点和一个尾节点组成，分别指向链表的头部和尾部。

```java
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

##### `LinkedList`构造函数

```java
    /**
     * Constructs an empty list.
     */
    public LinkedList() {
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param  c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

##### Node结构

```java
    private static class Node<E> {
        // 当前值item
        E item;
        // 指向下个节点next
        Node<E> next;
        // 指向上一个节点prev
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

##### `LinkedList`尾插法和头插法

```java
    /**
     * Links e as first element.
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }

    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

##### `LinkedList`查询方法

* **按照下标获取某一个节点**: get方法，获取第index个节点。

  ```java
      /**
       * Returns the element at the specified position in this list.
       *
       * @param index index of the element to return
       * @return the element at the specified position in this list
       * @throws IndexOutOfBoundsException {@inheritDoc}
       */
      public E get(int index) {
          checkElementIndex(index);
          return node(index).item;
      }
      private void checkElementIndex(int index) {
          if (!isElementIndex(index))
              throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
      }
      /**
       * Tells if the argument is the index of an existing element.
       */
      private boolean isElementIndex(int index) {
          return index >= 0 && index < size;
      }
      /**
       * Returns the (non-null) Node at the specified element index.
       */
      Node<E> node(int index) {
          // assert isElementIndex(index);
  		// 判断index是更靠近头部还是尾部，靠近哪段从哪段遍历获取值。
          if (index < (size >> 1)) {
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

##### `LinkedList`设置值

* 查询索引修改方法，先找到对应节点，将新的值替换掉老的值。

  ```java
      /**
       * Replaces the element at the specified position in this list with the
       * specified element.
       *
       * @param index index of the element to replace
       * @param element element to be stored at the specified position
       * @return the element previously at the specified position
       * @throws IndexOutOfBoundsException {@inheritDoc}
       */
      public E set(int index, E element) {
          checkElementIndex(index);
          Node<E> x = node(index);
          E oldVal = x.item;
          x.item = element;
          return oldVal;
      }
  ```

##### `LinkedList`添加元素

* 直接添加元素

  ```java
      /**
       * Appends the specified element to the end of this list.
       *
       * <p>This method is equivalent to {@link #addLast}.
       *
       * @param e element to be appended to this list
       * @return {@code true} (as specified by {@link Collection#add})
       */
      public boolean add(E e) {
          linkLast(e);
          return true;
      }
  ```

* 根据索引位置添加元素

  ```java
      /**
       * Inserts the specified element at the specified position in this list.
       * Shifts the element currently at that position (if any) and any
       * subsequent elements to the right (adds one to their indices).
       *
       * @param index index at which the specified element is to be inserted
       * @param element element to be inserted
       * @throws IndexOutOfBoundsException {@inheritDoc}
       */
      public void add(int index, E element) {
          checkPositionIndex(index);
  
          if (index == size)
              linkLast(element);
          else
              linkBefore(element, node(index));
      }
      /**
       * Inserts element e before non-null Node succ.
       */
      void linkBefore(E e, Node<E> succ) {
          // assert succ != null;
          final Node<E> pred = succ.prev;
          final Node<E> newNode = new Node<>(pred, e, succ);
          succ.prev = newNode;
          if (pred == null)
              first = newNode;
          else
              pred.next = newNode;
          size++;
          modCount++;
      }
  ```

##### `ArrayList`和`LinkedList`

- ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
- 对于随机访问get和set，ArrayList绝对优于LinkedList，因为LinkedList要移动指针。
- 对于新增和删除操作add和remove，`LinedList`比较占优势，因为`ArrayList`要移动数据。

#### **Vector**

> **Vector是线程安全的动态数组**

<img src="http://www.chenjunlin.vip/img/java/collection/Vector.png" alt="img" style="zoom:80%;" />

##### `Vector`常量

```java
    /**
     * The array buffer into which the components of the vector are
     * stored. The capacity of the vector is the length of this array buffer,
     * and is at least large enough to contain all the vector's elements.
     * 存放元素的数组
     * <p>Any array elements following the last element in the Vector are null.
     *
     * @serial
     */
    protected Object[] elementData;

    /**
     * The number of valid components in this {@code Vector} object.
     * Components {@code elementData[0]} through
     * {@code elementData[elementCount-1]} are the actual items.
     * 有效元素数量，小于等于数组长度
     * @serial
     */
    protected int elementCount;

    /**
     * The amount by which the capacity of the vector is automatically
     * incremented when its size becomes greater than its capacity.  If
     * the capacity increment is less than or equal to zero, the capacity
     * of the vector is doubled each time it needs to grow.
     * 容量增加量，和扩容相关
     * @serial
     */
    protected int capacityIncrement;
```

##### `Vector`构造函数

```java
/**
     * Constructs an empty vector with the specified initial capacity and
     * capacity increment.
     *
     * @param   initialCapacity     the initial capacity of the vector
     * @param   capacityIncrement   the amount by which the capacity is
     *                              increased when the vector overflows
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    /**
     * Constructs an empty vector with the specified initial capacity and
     * with its capacity increment equal to zero.
     *
     * @param   initialCapacity   the initial capacity of the vector
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }

    /**
     * Constructs an empty vector so that its internal data array
     * has size {@code 10} and its standard capacity increment is
     * zero.
     */
    public Vector() {
        this(10);
    }

    /**
     * Constructs a vector containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this
     *       vector
     * @throws NullPointerException if the specified collection is null
     * @since   1.2
     */
    public Vector(Collection<? extends E> c) {
        Object[] a = c.toArray();
        elementCount = a.length;
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, elementCount, Object[].class);
        }
    }
```

##### `Vector`扩容

```java
    /**
     * This implements the unsynchronized semantics of ensureCapacity.
     * Synchronized methods in this class can internally call this
     * method for ensuring capacity without incurring the cost of an
     * extra synchronization.
     *
     * @see #ensureCapacity(int)
     */
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 若没有指定capacityIncrement,则扩容两倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

##### `Vector`添加元素

```java
    /**
     * Appends the specified element to the end of this Vector.
     *
     * @param e element to be appended to this Vector
     * @return {@code true} (as specified by {@link Collection#add})
     * @since 1.2
     */
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }

    /**
     * Inserts the specified element at the specified position in this Vector.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     * @since 1.2
     */
    public void add(int index, E element) {
        insertElementAt(element, index);
    }
    /**
     * Inserts the specified object as a component in this vector at the
     * specified {@code index}. Each component in this vector with
     * an index greater or equal to the specified {@code index} is
     * shifted upward to have an index one greater than the value it had
     * previously.
     *
     * <p>The index must be a value greater than or equal to {@code 0}
     * and less than or equal to the current size of the vector. (If the
     * index is equal to the current size of the vector, the new element
     * is appended to the Vector.)
     *
     * <p>This method is identical in functionality to the
     * {@link #add(int, Object) add(int, E)}
     * method (which is part of the {@link List} interface).  Note that the
     * {@code add} method reverses the order of the parameters, to more closely
     * match array usage.
     *
     * @param      obj     the component to insert
     * @param      index   where to insert the new component
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index > size()})
     */
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }
```

##### `Vector`移除元素

```java
    /**
     * Removes the element at the specified position in this Vector.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).  Returns the element that was removed from the Vector.
     *
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     * @param index the index of the element to be removed
     * @return element that was removed
     * @since 1.2
     */
    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            // 复制数组，假设数组移除了中间某元素，后边有效值前移1位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--elementCount] = null; // Let gc do its work

        return oldValue;
    }
```

#### `Stack`

<img src="http://www.chenjunlin.vip/img/java/collection/Stack.png" alt="img" style="zoom:80%;" />

```java
package java.util;

/**
 * The <code>Stack</code> class represents a last-in-first-out
 * (LIFO) stack of objects. It extends class <tt>Vector</tt> with five
 * operations that allow a vector to be treated as a stack. The usual
 * <tt>push</tt> and <tt>pop</tt> operations are provided, as well as a
 * method to <tt>peek</tt> at the top item on the stack, a method to test
 * for whether the stack is <tt>empty</tt>, and a method to <tt>search</tt>
 * the stack for an item and discover how far it is from the top.
 * <p>
 * When a stack is first created, it contains no items.
 *
 * <p>A more complete and consistent set of LIFO stack operations is
 * provided by the {@link Deque} interface and its implementations, which
 * should be used in preference to this class.  For example:
 * <pre>   {@code
 *   Deque<Integer> stack = new ArrayDeque<Integer>();}</pre>
 *
 * @author  Jonathan Payne
 * @since   JDK1.0
 */
public
class Stack<E> extends Vector<E> {
    /**
     * Creates an empty Stack.
     */
    public Stack() {
    }

    /**
     * Pushes an item onto the top of this stack. This has exactly
     * the same effect as:
     * <blockquote><pre>
     * addElement(item)</pre></blockquote>
     *
     * @param   item   the item to be pushed onto this stack.
     * @return  the <code>item</code> argument.
     * @see     java.util.Vector#addElement
     */
    public E push(E item) {
        addElement(item);

        return item;
    }

    /**
     * Removes the object at the top of this stack and returns that
     * object as the value of this function.
     *
     * @return  The object at the top of this stack (the last item
     *          of the <tt>Vector</tt> object).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }

    /**
     * Looks at the object at the top of this stack without removing it
     * from the stack.
     *
     * @return  the object at the top of this stack (the last item
     *          of the <tt>Vector</tt> object).
     * @throws  EmptyStackException  if this stack is empty.
     */
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }

    /**
     * Tests if this stack is empty.
     *
     * @return  <code>true</code> if and only if this stack contains
     *          no items; <code>false</code> otherwise.
     */
    public boolean empty() {
        return size() == 0;
    }

    /**
     * Returns the 1-based position where an object is on this stack.
     * If the object <tt>o</tt> occurs as an item in this stack, this
     * method returns the distance from the top of the stack of the
     * occurrence nearest the top of the stack; the topmost item on the
     * stack is considered to be at distance <tt>1</tt>. The <tt>equals</tt>
     * method is used to compare <tt>o</tt> to the
     * items in this stack.
     *
     * @param   o   the desired object.
     * @return  the 1-based position from the top of the stack where
     *          the object is located; the return value <code>-1</code>
     *          indicates that the object is not on the stack.
     */
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 1224463164541339165L;
}

```

#### `Queue`

> **队列的操作不会因为队列为空抛出异常，而集合的操作是队列为空抛出异常。**

```java
public interface Queue<E> extends Collection<E> {
    /**
     * Inserts the specified element into this queue if it is possible to do so
     * immediately without violating capacity restrictions, returning
     * {@code true} upon success and throwing an {@code IllegalStateException}
     * if no space is currently available.
     * 增加一个元素。如果队列已满，则抛出一个IIIegaISlabEepeplian异常
     * @param e the element to add
     * @return {@code true} (as specified by {@link Collection#add})
     * @throws IllegalStateException if the element cannot be added at this
     *         time due to capacity restrictions
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this queue
     * @throws NullPointerException if the specified element is null and
     *         this queue does not permit null elements
     * @throws IllegalArgumentException if some property of this element
     *         prevents it from being added to this queue
     */
    boolean add(E e);

    /**
     * Inserts the specified element into this queue if it is possible to do
     * so immediately without violating capacity restrictions.
     * When using a capacity-restricted queue, this method is generally
     * preferable to {@link #add}, which can fail to insert an element only
     * by throwing an exception.
     * 添加一个元素并返回true。如果队列已满，则返回false
     * @param e the element to add
     * @return {@code true} if the element was added to this queue, else
     *         {@code false}
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this queue
     * @throws NullPointerException if the specified element is null and
     *         this queue does not permit null elements
     * @throws IllegalArgumentException if some property of this element
     *         prevents it from being added to this queue
     */
    boolean offer(E e);

    /**
     * Retrieves and removes the head of this queue.  This method differs
     * from {@link #poll poll} only in that it throws an exception if this
     * queue is empty.
     * 移除元素，当集合为空，抛出异常
     * @return the head of this queue
     * @throws NoSuchElementException if this queue is empty
     */
    E remove();

    /**
     * Retrieves and removes the head of this queue,
     * or returns {@code null} if this queue is empty.
     * 移除队列头部元素并返回，如果为空，返回null
     * @return the head of this queue, or {@code null} if this queue is empty
     */
    E poll();

    /**
     * Retrieves, but does not remove, the head of this queue.  This method
     * differs from {@link #peek peek} only in that it throws an exception
     * if this queue is empty.
     * 查询集合第一个元素，如果为空，抛出异常
     * @return the head of this queue
     * @throws NoSuchElementException if this queue is empty
     */
    E element();

    /**
     * Retrieves, but does not remove, the head of this queue,
     * or returns {@code null} if this queue is empty.
     * 查询队列中第一个元素，如果为空，返回null
     * @return the head of this queue, or {@code null} if this queue is empty
     */
    E peek();
}
```

##### `Deque`

> Deque英文全称是Double ended queue，也就是俗称的双端队列.对于这个队列容器，既可以从头部插入也可以从尾部插入，既可以从头部获取，也可以从尾部获取.

##### `PriorityQueue`

###### `PriorityQueue`常量

```java
    private static final int DEFAULT_INITIAL_CAPACITY = 11;

    /**
	 * 存放元素的数组    
     * Priority queue represented as a balanced binary heap: the two
     * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
     * priority queue is ordered by comparator, or by the elements'
     * natural ordering, if comparator is null: For each node n in the
     * heap and each descendant d of n, n <= d.  The element with the
     * lowest value is in queue[0], assuming the queue is nonempty.
     */
    transient Object[] queue; // non-private to simplify nested class access

    /**
     * 队列中存放了多少元素 
     * The number of elements in the priority queue.
     */
    private int size = 0;

    /**
     * 自定义的比较规则，有该规则时优先使用，否则使用元素实现的Comparable接口方法。
     * The comparator, or null if priority queue uses elements'
     * natural ordering.
     */
    private final Comparator<? super E> comparator;

    /**
     * 队列修改次数，每次存取都算一次修改
     * The number of times this priority queue has been
     * <i>structurally modified</i>.  See AbstractList for gory details.
     */
    transient int modCount = 0; // non-private to simplify nested class access
```

###### `PriorityQueue`添加元素

```java
    /**
     * Inserts the specified element into this priority queue.
     *
     * @return {@code true} (as specified by {@link Collection#add})
     * @throws ClassCastException if the specified element cannot be
     *         compared with elements currently in this priority queue
     *         according to the priority queue's ordering
     * @throws NullPointerException if the specified element is null
     */
    public boolean add(E e) {
        return offer(e);
    }

    /**
     * Inserts the specified element into this priority queue.
     *
     * @return {@code true} (as specified by {@link Queue#offer})
     * @throws ClassCastException if the specified element cannot be
     *         compared with elements currently in this priority queue
     *         according to the priority queue's ordering
     * @throws NullPointerException if the specified element is null
     */
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1);
        size = i + 1;
        if (i == 0)
            queue[0] = e;
        else
            siftUp(i, e);
        return true;
    }
```

##### `ArrayDeque`

### `Set`

> Set是一种不包括重复元素的Collection。它维持它自己的内部排序，所以随机访问没有任何意义。与List一样，它同样允许null的存在但是仅有一个。由于Set接口的特殊性，**所有传入Set集合中的元素都必须不同**，同时要注意任何可变对象，如果在对集合中元素进行操作时，导致`e1.equals(e2)==true`，则必定会产生某些问题。Set接口有三个具体实现类，分别是散列集`HashSet`、链式散列集`LinkedHashSet`和树形集`TreeSet`。
>
> **Set是一种不包含重复的元素的Collection，无序，即任意的两个元素e1和e2都有e1.equals(e2)=false，Set最多有一个null元素。**
>
> 需要注意的是：虽然Set中元素没有顺序，但是元素在set中的位置是由该元素的hashCode决定的，其具体位置其实是固定的。

#### `HashSet`

> `HashSet` 是一个没有重复元素的集合。它是由`HashMap`实现的，不保证元素的顺序(这里所说的没有顺序是指：元素插入的顺序与输出的顺序不一致)，而且`HashSet`允许使用null 元素。`HashSet`是非同步的，如果多个线程同时访问一个哈希set，而其中至少一个线程修改了该set，那么它必须保持外部同步。 **`HashSet`按Hash算法来存储集合的元素，因此具有很好的存取和查找性能。**

##### **`HashSet`使用和理解中容易出现的误区**

* **`HashSet`中存放null值。**`HashSet`中是允许存入null值的，但是在`HashSet`中仅仅能够存入一个null值。

* **`HashSet`中存储元素的位置是固定的。**`HashSet`中存储的元素的是无序的，这个没什么好说的，但是由于`HashSet`底层是基于Hash算法实现的，使用了`hashCode`，所以`HashSet`中相应的元素的位置是固定的。

* **必须小心操作可变对象**（`Mutable Object`）。如果一个Set中的可变元素改变了自身状态导致`Object.equals(Object)=true`将导致一些问题。

#### `LinkedHashSet`

> `LinkedHashSet`继承自`HashSet`，其底层是**基于`LinkedHashMap`来实现的**，有序，非同步。`LinkedHashSet`集合同样是根据元素的`hashCode`值来决定元素的存储位置，但是它同时使用链表维护元素的次序。这样使得元素看起来像是以插入顺序保存的，也就是说，当遍历该集合时候，**`LinkedHashSet`将会以元素的添加顺序访问集合的元素。**

#### `TreeSet`

> `TreeSet`是一个有序集合，其底层是基于`TreeMap`实现的，非线程安全。`TreeSet`可以确保集合元素处于排序状态。**`TreeSet`支持两种排序方式，自然排序和定制排序，其中自然排序为默认的排序方式。**当我们构造`TreeSet`时，若使用不带参数的构造函数，则`TreeSet`的使用自然比较器；若用户需要使用自定义的比较器，则需要使用带比较器的参数。

> 注意：`TreeSet`集合不是通过`hashCode`和equals函数来比较元素的.它是通过`compare`或者`comparaeTo`函数来判断元素是否相等.`compare`函数通过判断两个对象的id，相同的id判断为重复元素，不会被加入到集合中。

### Map

> Map与List、Set接口不同，它是由一系列键值对组成的集合，提供了key到Value的映射。同时它也没有继承Collection。在Map中它保证了key与value之间的一一对应关系。也就是说一个key对应一个value，所以它**不能存在相同的key值，当然value值可以相同**。

#### `HashMap` 

> [HashMap分析](http://www.chenjunlin.vip/2021/05/25/java/HashMap/)

#### `LinkedHashMap`

> `LinkedHashMap`是`HashMap`的一个子类，它保留插入的顺序，如果需要输出的顺序和输入时的相同，那么就选用`LinkedHashMap`。
>
> **`LinkedHashMap`是Map接口的哈希表和链接列表实现，具有可预知的迭代顺序。**此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。
>
> `LinkedHashMap`实现与`HashMap`的不同之处在于，后者维护着一个运行于所有条目的双重链接列表。此链接列表定义了迭代顺序，该迭代顺序可以是插入顺序或者是访问顺序。
>
> 根据链表中元素的顺序可以分为：按插入顺序的链表，和按访问顺序(调用get方法)的链表。默认是按插入顺序排序，如果指定按访问顺序排序，那么调用get方法后，会将这次访问的元素移至链表尾部，不断访问可以形成按访问顺序排序的链表。

> 注意，此实现不是同步的。如果多个线程同时访问链接的哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。由于`LinkedHashMap`需要维护元素的插入顺序，因此性能略低于`HashMap`的性能，但在迭代访问Map里的全部元素时将有很好的性能，因为它以链表来维护内部顺序。

#### `TreeMap`

> **`TreeMap`是一个有序的key-value集合，非同步，基于红黑树（Red-Black tree）实现，每一个key-value节点作为红黑树的一个节点。**`TreeMap`存储时会进行排序的，会根据key来对key-value键值对进行排序，其中排序方式也是分为两种，一种是自然排序，一种是定制排序，具体取决于使用的构造方法。
>
> **自然排序：**`TreeMap`中所有的key必须实现Comparable接口，并且所有的key都应该是同一个类的对象，否则会报`ClassCastException`异常。
>
> **定制排序：**定义`TreeMap`时，创建一个comparator对象，该对象对所有的`TreeMap`中所有的key值进行排序，采用定制排序的时候不需要`TreeMap`中所有的key必须实现Comparable接口。
>
> `TreeMap`判断两个元素相等的标准：两个key通过`compareTo()`方法返回0，则认为这两个key相等。

### 常见问题

#### `HashTable与HashMap`

* **相同点**：
  * 都实现了`Map、Cloneable、java.io.Serializable`接口。
  * 都是存储"键值对(key-value)"的散列表，而且都是采用拉链法实现的。

* **不同点**：

  * **历史原因：**`HashTable`是基于陈旧的Dictionary类的，`HashMap`是Java 1.2引进的Map接口的一个实现 。

  * **同步性：**`HashTable`是线程安全的，也就是说是同步的，而`HashMap`是线程序不安全的，不是同步的 。

  * **对null值的处理：**`HashMap`的key、value都可为null，`HashTable`的key、value都不可为null 。

  * **基类不同：**`HashMap`继承于`AbstractMap`，而`Hashtable`继承于Dictionary。
  * **支持的遍历种类不同：**`HashMap`只支持Iterator(迭代器)遍历。而`Hashtable`支持Iterator(迭代器)和Enumeration(枚举器)两种方式遍历。
  * `HashMap`提供对key的Set进行遍历，因此它是fail-fast的，但`HashTable`提供对key的Enumeration进行遍历，它不支持fail-fast。

#### `Fail-Fast`与`Fail-Safe`有什么区别?

* `Iterator`的`Fail-Fast`属性与当前的集合共同起作用，因此它不会受到集合中任何改动的影响。
* `Java.util`包中的所有集合类都被设计为`Fail-Fast`的，而`java.util.concurrent`中的集合类都为`Fail-Safe`的。

* `Fall-fast`迭代器抛出`ConcurrentModificationException`;

* `Fall-safe`迭代器从不抛出`ConcurrentModificationException`。

#### `ArrayList`和`Vector`有何异同点?

* **相同点:**
  * 两者都是基于索引的，内部由一个数组支持。
  * 两者维护插入的顺序，我们可以根据插入顺序来获取元素。
  * `ArrayList`和`Vector`的迭代器实现都是fail-fast的。
  * `ArrayList`和`Vector`两者允许null值，也可以使用索引值对元素进行随机访问。
* **不同点:**
  * `Vector`是同步的，而`ArrayList`不是。然而，如果你寻求在迭代的时候对列表进行改变，你应该使用`CopyOnWriteArrayList`。
  * `ArrayList`比Vector快，它因为没有同步，不会过载。
  * `ArrayList`更加通用，因为我们可以使用Collections工具类轻易地获取同步列表和只读列表。

#### `Array`和`ArrayList`有何区别？什么时候更适合用`Array`？

* Array可以容纳基本类型和对象，而`ArrayList`只能容纳对象;
* Array是指定大小的，而`ArrayList`大小是固定的;
* Array没有提供`ArrayList`那么多功能，比如`addAll`、`removeAll`和`Iterator`等;
* 如果列表的大小已经指定，大部分情况下是存储和遍历它们。
* 对于遍历基本数据类型，尽管Collections使用自动装箱来减轻编码任务，在指定大小的基本类型的列表上工作也会变得很慢。
* 如果你要使用多维数组，使用[][]比List<List<>>更容易。
