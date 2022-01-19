---
title: 数据结构(八)-Map
date: 2021-12-21 15:02:12
cover: /img/cover/DataStructure.png
tags:
- Map
categories:
- DataStructure
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

### Map

```java
package com.holelin.map;

/**
 * Map接口
 * @param <K> 键
 * @param <V> 值
 */
public interface Map<K, V> {
	/**
	 * 添加元素
	 *
	 * @param key   键
	 * @param value 值
	 */
	void add(K key, V value);

	/**
	 * 根据key移除元素
	 *
	 * @param key 键
	 * @return 删除的元素
	 */
	V remove(K key);

	/**
	 * 判断Map中是否包含key
	 *
	 * @param key 键
	 * @return 包含返回true, 反之返回false
	 */
	boolean contains(K key);

	/**
	 * 根据key获取value
	 *
	 * @param key 键
	 * @return key对应value
	 */
	V get(K key);

	/**
	 * 修改key对应的value值
	 *
	 * @param key   键
	 * @param value 值
	 */
	void set(K key, V value);

	/**
	 * 获取Map中元素的个数
	 *
	 * @return
	 */
	int getSize();

	/**
	 * 判断Map是否为空
	 *
	 * @return 为空返回true, 反之返回false;
	 */
	boolean isEmpty();

}
```

#### LinkedListMap

```java
package com.holelin.map;

/**
 * ClassName: LinkedListMap
 * 基于链表实现的Map
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/11
 */

public class LinkedListMap<K, V> implements Map<K, V> {
	private Node dummyHead;
	private int size;

	public LinkedListMap() {
		dummyHead = new Node();
		size = 0;
	}

	private Node getNode(K key) {
		Node cur = dummyHead.next;
		while (cur != null) {
			if (cur.key.equals(key)) {
				return cur;
			}
			cur = cur.next;
		}
		return null;
	}

	@Override
	public void add(K key, V value) {
		Node node = getNode(key);
		if (node == null) {
			dummyHead.next = new Node(key, value, dummyHead.next);
			size++;
		} else {
			// 如果重复的key,则修改当前的key对应value
			node.value = value;
		}
	}

	@Override
	public V remove(K key) {
		Node prev = dummyHead;
		while (prev.next != null) {
			if (prev.next.key.equals(key)) {
				break;
			}
			prev = prev.next;
		}
		if (prev.next != null) {
			Node delNode = prev.next;
			prev.next = delNode.next;
			delNode.next = null;
			size--;
			return delNode.value;
		}
		return null;

	}

	@Override
	public boolean contains(K key) {
		return get(key) != null;
	}

	@Override
	public V get(K key) {
		Node node = getNode(key);
		return node == null ? null : node.value;
	}

	@Override
	public void set(K key, V value) {
		Node node = getNode(key);
		if (node == null) {
			throw new IllegalArgumentException(key + " doesn't exist");
		}
		node.value = value;
	}

	@Override
	public int getSize() {
		return 0;
	}

	@Override
	public boolean isEmpty() {
		return false;
	}

	private class Node {
		/**
		 * 键
		 */
		K key;
		/**
		 * 值
		 */
		V value;
		/**
		 * 指针域
		 */
		Node next;

		public Node(K key, V value, Node next) {
			this.key = key;
			this.value = value;
			this.next = next;
		}

		public Node(K key, V value) {
			this.key = key;
			this.value = value;
			this.next = null;
		}

		public Node() {
			this(null, null);
		}

		@Override
		public String toString() {
			return key.toString() + " : " + value.toString();
		}
	}
}

```

#### AVLMap

```java
package com.holelin.map;

import com.holelin.tree.AVLTree;

/**
 * ClassName: AVLMap
 * 基于AVL树实现的Map
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/17
 */

public class AVLMap<K extends Comparable<K>, V> implements Map<K, V> {
	private AVLTree<K, V> mAVLTree;

	public AVLMap() {
		mAVLTree = new AVLTree<>();
	}

	@Override
	public void add(K key, V value) {
		mAVLTree.add(key, value);
	}

	@Override
	public V remove(K key) {
		return mAVLTree.remove(key);
	}

	@Override
	public boolean contains(K key) {
		return mAVLTree.contains(key);
	}

	@Override
	public V get(K key) {
		return mAVLTree.get(key);
	}

	@Override
	public void set(K key, V value) {
		mAVLTree.set(key, value);
	}

	@Override
	public int getSize() {
		return mAVLTree.getSize();
	}

	@Override
	public boolean isEmpty() {
		return mAVLTree.isEmpty();
	}
}

```

#### BSTMap

```java
package com.holelin.map;

/**
 * ClassName: BSTMap
 *
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/11
 */

public class BSTMap<K extends Comparable<K>, V> implements Map<K, V> {
	private Node root;
	private int size;

	public BSTMap() {
		root = null;
		size = 0;
	}

	@Override
	public void add(K key, V value) {
		root = add(root, key, value);
	}

	private Node add(Node node, K key, V value) {
		if (node == null) {
			size++;
			return new Node(key, value);
		}
		if (key.compareTo(node.key) < 0) {
			node.left = add(node.left, key, value);
		} else if (key.compareTo(node.key) > 0) {
			node.right = add(node.right, key, value);
		} else {
			node.value = value;
		}
		return node;
	}

	private Node getNode(Node node, K key) {
		if (node == null) {
			return null;
		}
		if (node.key.compareTo(key) == 0) {
			return node;
		} else if (node.key.compareTo(key) < 0) {
			return getNode(node.left, key);
		} else {
			return getNode(node.right, key);
		}
	}

	@Override
	public V remove(K key) {
		Node node = getNode(root, key);
		if (node != null) {
			root = remove(root, key);
			return root.value;
		}
		return null;
	}


	private Node remove(Node node, K key) {
		if (node == null) {
			return null;
		}
		if (key.compareTo(node.key) < 0) {
			// 到node左子树寻找
			node.left = remove(node.left, key);
			return node;
		} else if (key.compareTo(node.key) > 0) {
			node.right = remove(node.right, key);
			return node;
		} else {
			// 待删除节点左子树为空的情况
			if (node.left == null) {
				Node rightNode = node.right;
				node.right = null;
				size--;
				return rightNode;
			}
			// 待删除节点右子树为空的情况
			if (node.right == null) {
				Node leftNode = node.left;
				node.left = null;
				size--;
				return leftNode;
			}
			// 带删除节点左右子树均不为空的情况
			// 找到比带删除节点大的最小的节点,即待删除节点右子树的最小节点
			// 用这个节点顶替待删除节点的位置
			Node successor = minimum(node.right);
			successor.right = removeMin(node.right);
			successor.left = node.left;
			node.left = node.right = null;
			return successor;
		}
	}

	/**
	 * 返回以node为根的二分搜索树的最小值所在的结点
	 *
	 * @param node 以node为根的二分搜索树
	 * @return 以node为根的二分搜索树的最小值所在的结点
	 */
	private Node minimum(Node node) {
		if (node.left == null) {
			return node;
		}
		return minimum(node.left);
	}

	/**
	 * 删除掉以node为根的二分搜索树中的最小节点
	 * 返回删除节点后的新的二分搜索树的根
	 *
	 * @param node 以node为根的二分搜索树
	 * @return 返回删除节点后的新的二分搜索树的根
	 */
	private Node removeMin(Node node) {
		if (node.left == null) {
			Node rightNode = node.right;
			node.right = null;
			size--;
			return rightNode;
		}
		node.left = removeMin(node.left);
		return node;
	}

	@Override
	public boolean contains(K key) {
		return getNode(root, key) != null;
	}

	@Override
	public V get(K key) {
		Node node = getNode(root, key);
		return node == null ? null : node.value;
	}

	@Override
	public void set(K key, V value) {
		Node node = getNode(root, key);
		if (node == null) {
			throw new IllegalArgumentException(key + "doesn't exist!");
		}
		node.value = value;
	}

	@Override
	public int getSize() {
		return size;
	}

	@Override
	public boolean isEmpty() {
		return size == 0;
	}

	private class Node {
		/**
		 * 键
		 */
		public K key;
		/**
		 * 值
		 */
		public V value;
		/**
		 * 左子树地址域
		 */
		public Node left;
		/**
		 * 右子树地址域
		 */
		public Node right;

		public Node(K key, V value) {
			this.key = key;
			this.value = value;
			this.left = null;
			this.right = null;
		}
	}

}
```

#### 测试用例

```java
package com.holelin.map;

import com.holelin.util.FileOperation;

import java.util.ArrayList;

/**
 * ClassName: MapTest
 * Map测试类
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/17
 */

public class MapTest {
	private static double testMap(Map<String, Integer> map, String filename) {

		long startTime = System.nanoTime();
		ArrayList<String> words = new ArrayList<>();
		if (FileOperation.readFile(filename, words)) {
			System.out.println("Total words: " + words.size());

			for (String word : words) {
				if (map.contains(word)) {
					map.set(word, map.get(word) + 1);
				} else {
					map.add(word, 1);
				}
			}

			System.out.println("Total different words: " + map.getSize());

		}

		long endTime = System.nanoTime();

		return (endTime - startTime) / 1000000000.0;
	}

	public static void main(String[] args) {
		String path = "src/res/Pride-and-prejudice.txt";
		BSTMap<String, Integer> bstMap = new BSTMap<>();
		double time1 = testMap(bstMap, path);
		System.out.println("BST Map: " + time1 + " s");

		System.out.println();

		LinkedListMap<String, Integer> linkedListMap = new LinkedListMap<>();
		double time2 = testMap(linkedListMap, path);
		System.out.println("Linked List Map: " + time2 + " s");

		System.out.println();

		AVLMap<String, Integer> avlMap = new AVLMap<>();
		double time3 = testMap(avlMap, path);
		System.out.println("AVL Map: " + time3 + " s");

	}
}
```

#### HashTable

```java
package com.holelin.tree;

import java.util.TreeMap;

/**
 * ClassName: HashTable
 * 哈希表
 *
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/19
 */

public class HashTable<K, V> {
	private final int[] capacity
			= {53, 97, 193, 389, 769, 1543, 3079, 6151, 12289, 24593,
			49157, 98317, 196613, 393241, 786433, 1572869, 3145739, 6291469,
			12582917, 25165843, 50331653, 100663319, 201326611, 402653189, 805306457, 1610612741};
	private TreeMap<K, V>[] hashtable;
	private int M;
	private int size;

	/**
	 * 平均每个地址承载的元素的上界
	 */
	private static final int upperTol = 10;
	/**
	 * 平均每个地址承载的元素的下界
	 */
	private static final int lowerTol = 2;
	/**
	 * 初始容量
	 */
	private int capacityIndex = 0;

	public HashTable() {
		this.M = capacity[capacityIndex];
		this.size = 0;
		hashtable = new TreeMap[M];
		for (int i = 0; i < hashtable.length; i++) {
			hashtable[i] = new TreeMap<>();
		}
	}

	/**
	 * 计算hash值
	 *
	 * @param key 传入的key
	 * @return 计算后key的hash值
	 */
	private int hash(K key) {
		//  & 0x7fffffff -- 消除负号
		return (key.hashCode() & 0x7fffffff) % M;
	}

	/**
	 * @return
	 */
	public int getSize() {
		return size;
	}

	public void add(K key, V value) {
		TreeMap<K, V> map = hashtable[hash(key)];
		if (map.containsKey(key)) {
			map.put(key, value);
		} else {
			map.put(key, value);
			size++;
		}
		if (size >= upperTol * M && capacityIndex + 1 < capacity.length) {
			capacityIndex++;
			resize(capacity[capacityIndex]);
		}
	}

	private void resize(int newM) {
		TreeMap<K, V>[] newHashTable = new TreeMap[newM];
		for (int i = 0; i < newM; i++) {
			newHashTable[i] = new TreeMap<>();
		}
		int oldM = M;
		this.M = newM;
		for (int i = 0; i < oldM; i++) {
			TreeMap<K, V> map = hashtable[i];
			for (K key : map.keySet()) {
				newHashTable[hash(key)].put(key, map.get(key));
			}
		}
		this.hashtable = newHashTable;
	}

	public V remove(K key) {
		TreeMap<K, V> map = hashtable[hash(key)];
		V ret = null;
		if (map.containsKey(key)) {
			ret = map.remove(key);
			size--;
		}
		if (size < lowerTol * M && capacityIndex - 1 >= 0) {
			capacityIndex--;
			resize(capacity[capacityIndex]);
		}
		return ret;
	}

	public void set(K key, V value) {
		TreeMap<K, V> map = hashtable[hash(key)];
		if (!map.containsKey(key)) {
			throw new IllegalArgumentException(key + "does't exist");
		}
		map.put(key, value);
	}

	public boolean contains(K key) {
		return hashtable[hash(key)].containsKey(key);
	}

	public V get(K key) {
		return hashtable[hash(key)].get(key);
	}


}
```

