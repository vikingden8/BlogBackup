---
title: 数据结构中的二叉树
date: 2017-08-27 22:00:57
categories: "数据结构与算法"
---

### 二叉树的定义

二叉树(binary tree)是结点的有限集合，这个集合或者空，或者由一个根及两个互不相交的称为这个根的左子树或右子树构成.

从定义可以看出，二叉树包括：1.空树; 2.只有一个根节点; 3.只有左子树;  4.只有右子树;  5.左右子树都存在. 有且仅有这5中表现形式

### 二叉树与一般树的区别

* 一般树的子树不分次序，而二叉树的子树有左右之分;

* 由于二叉树也是树的一种，所以大部分的树的概念，对二叉树也适用;

* 二叉树的存储：每个节点只需要两个指针域（左节点，右节点）,有的为了操作方便也会增加指向父级节点的指针，除了指针域以外，还会有一个数据域用来保存当前节点的信息.

<!--more-->

### 二叉树的特点

* 性质1：在二叉树的第i层上至多有2^(i-1)个节点(i >= 1)
* 性质2：深度为k的二叉树至多有2^(k-1)个节点(k >=1)
* 性质3：对于任意一棵二叉树T而言，其叶子节点数目为N0,度为2的节点数目为N2，则有N0 = N2 + 1。
* 性质4：具有n个节点的完全二叉树的深度 。

### 二叉树的遍历

二叉树的遍历分为三种：前序遍历,中序遍历,后序遍历.

* **前序遍历**：按照“根左右”,先遍历根节点，再遍历左子树 ，再遍历右子树.

* **中序遍历**：按照“左根右“,先遍历左子树，再遍历根节点，最后遍历右子树.

* **后续遍历**：按照“左右根”，先遍历左子树，再遍历右子树，最后遍历根节点 .

其中前，后，中指的是每次遍历时候的根节点被遍历的顺序.

### 二叉树遍历的java实现

```java
public class BinaryTreeTraverse {

	private int[] array = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
	private static List<Node> nodeList = null;

	private static class Node {
		Node leftChild;
		Node rightChild;
		int data;

		Node(int newData) {
			leftChild = null;
			rightChild = null;
			data = newData;
		}
	}

	public void createBinTree() {
		nodeList = new LinkedList<Node>();
		// 将一个数组的值依次转换为Node节点
		for (int nodeIndex = 0; nodeIndex < array.length; nodeIndex++) {
			nodeList.add(new Node(array[nodeIndex]));
		}
		// 对前lastParentIndex-1个父节点按照父节点与孩子节点的数字关系建立二叉树
		for (int parentIndex = 0; parentIndex < array.length / 2 - 1; parentIndex++) {
			// 左孩子
			nodeList.get(parentIndex).leftChild = nodeList
					.get(parentIndex * 2 + 1);
			// 右孩子
			nodeList.get(parentIndex).rightChild = nodeList
					.get(parentIndex * 2 + 2);
		}
		// 最后一个父节点:因为最后一个父节点可能没有右孩子，所以单独拿出来处理
		int lastParentIndex = array.length / 2 - 1;
		// 左孩子
		nodeList.get(lastParentIndex).leftChild = nodeList
				.get(lastParentIndex * 2 + 1);
		// 右孩子,如果数组的长度为奇数才建立右孩子
		if (array.length % 2 == 1) {
			nodeList.get(lastParentIndex).rightChild = nodeList
					.get(lastParentIndex * 2 + 2);
		}
	}

	/**
	 * 先序遍历
	 *
	 * 这三种不同的遍历结构都是一样的，只是先后顺序不一样而已
	 *
	 * @param node
	 *            遍历的节点
	 */
	public static void preOrderTraverse(Node node) {
		if (node == null)
			return;
		System.out.print(node.data + " ");
		preOrderTraverse(node.leftChild);
		preOrderTraverse(node.rightChild);
	}

	/**
	 * 中序遍历
	 *
	 * 这三种不同的遍历结构都是一样的，只是先后顺序不一样而已
	 *
	 * @param node
	 *            遍历的节点
	 */
	public static void inOrderTraverse(Node node) {
		if (node == null)
			return;
		inOrderTraverse(node.leftChild);
		System.out.print(node.data + " ");
		inOrderTraverse(node.rightChild);
	}

	/**
	 * 后序遍历
	 *
	 * 这三种不同的遍历结构都是一样的，只是先后顺序不一样而已
	 *
	 * @param node
	 *            遍历的节点
	 */
	public static void postOrderTraverse(Node node) {
		if (node == null)
			return;
		postOrderTraverse(node.leftChild);
		postOrderTraverse(node.rightChild);
		System.out.print(node.data + " ");
	}

	public static void main(String[] args) {
		BinaryTreeTraverse binTree = new BinaryTreeTraverse();
		binTree.createBinTree();
		// nodeList中第0个索引处的值即为根节点
		Node root = nodeList.get(0);

		System.out.println("先序遍历：");
		preOrderTraverse(root);
		System.out.println();

		System.out.println("中序遍历：");
		inOrderTraverse(root);
		System.out.println();

		System.out.println("后序遍历：");
		postOrderTraverse(root);
	}

}
```
