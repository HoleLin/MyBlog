---
title: Java集合类
mermaid: true
date: 2021-06-12 00:21:17
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

#### **Vector**

