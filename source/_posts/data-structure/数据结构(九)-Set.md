---
title: 数据结构九-Set
date: 2021-12-21 15:05:43
cover: /img/cover/DataStructure.png
tags:
- Set
categories:
- 数据结构
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

### Set

```java
package com.holelin.set;

public interface Set<E> {
	/**
	 * 添加元素e
	 *
	 * @param e 元素e
	 */
	void add(E e);

	/**
	 * 删除元素e
	 *
	 * @param e 元素e
	 */
	void remove(E e);

	/**
	 * 查询集合中是否包含元素e
	 *
	 * @param e 元素e
	 * @return 包含则返回true, 反之返回false
	 */
	boolean contains(E e);

	/**
	 * 返回集合元素的个数
	 *
	 * @return 集合元素的个数
	 */
	int getSize();

	/**
	 * 判断集合是否为空
	 *
	 * @return 为空返回true, 反之返回false;
	 */
	boolean isEmpty();
}

```

#### LinkedListSet

```java
package com.holelin.set;

import com.holelin.linkedlist.LinkedList;

/**
 * ClassName: LinkedListSet
 *
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/11
 */

public class LinkedListSet<E extends Comparable<E>> implements Set<E> {
	private LinkedList<E> mLinkedList;

	public LinkedListSet() {
		mLinkedList = new LinkedList<>();
	}

	@Override
	public void add(E e) {
		if (!mLinkedList.contains(e)) {
			mLinkedList.addFirst(e);
		}
	}

	@Override
	public void remove(E e) {
		mLinkedList.removeElement(e);
	}

	@Override
	public boolean contains(E e) {
		return mLinkedList.contains(e);
	}

	@Override
	public int getSize() {
		return mLinkedList.getSize();
	}

	@Override
	public boolean isEmpty() {
		return mLinkedList.isEmpty();
	}
}

```

#### AVLSet

```java
package com.holelin.set;

import com.holelin.tree.AVLTree;

/**
 * ClassName: AVLSet
 * 基于AVL树实现的Set
 *
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/17
 */

public class AVLSet<E extends Comparable<E>> implements Set<E> {
	private AVLTree<E, Object> mAVLTree;

	public AVLSet() {
		mAVLTree = new AVLTree<>();
	}

	@Override
	public void add(E e) {
		mAVLTree.add(e, null);
	}

	@Override
	public void remove(E e) {
		mAVLTree.remove(e);
	}

	@Override
	public boolean contains(E e) {
		return mAVLTree.contains(e);
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

#### BSTSet

```java
package com.holelin.set;

import com.holelin.tree.BST;

/**
 * ClassName: BSTSet
 * 基于二分搜索树实现集合
 *
 * @author HoleLin
 * @version 1.0
 * @date 2019/2/11
 */

public class BSTSet<E extends Comparable<E>> implements Set<E> {

	private BST<E> mBst;

	public BSTSet() {
		mBst = new BST<>();
	}

	@Override
	public void add(E e) {
		mBst.add(e);
	}

	@Override
	public void remove(E e) {
		mBst.remove(e);
	}

	@Override
	public boolean contains(E e) {
		return mBst.contains(e);
	}

	@Override
	public int getSize() {
		return mBst.size();
	}

	@Override
	public boolean isEmpty() {
		return mBst.isEmpty();
	}
}
```



