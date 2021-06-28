> 人生最精彩的不是成功的那一瞬间，而是回头看，那段漆黑看似没有尽头、苦苦摸索的过程。其实，我只是很在意，在意在我所在意的人的心里，我，在哪个位置。——来源于网络

### **1.树的概述**

树是一种非线性结构，其中的元素被组织成了一个层次结构。 
树由一个包含结点和边的集构成，其中元素被存储在这些结点中，边则将一个结点与另一个结点链接起来。每一个结点是都位于该树层次结构中的某一个特定层次上。树的跟(root)就是就是那个位于树顶层的唯一结点。

根结点是树中唯一一个没有双亲的结点，没有任何孩子的结点称为叶子结点，一个至少有一个孩子的非跟结点称为内部结点。

平衡树：对平衡的定义有很多种，粗略的来说，如果树的叶子都位于同一层或者至少彼此相差不超过一层，那么就称之为平衡的。

完全树：如果某树树平衡的，且底层的所有叶子都位于树的左边，则认为该树是完全的。

满树：如果一棵n元树的所有叶子都位于同一层，而且每一个结点要么是叶子，要么就正好拥有n个孩子，则称此树是满的。

### **2.树的实现**

#### 2.1 树的数组实现

计算策略：对某些特定类型的树，特别是二叉树而言，二叉树的存储设从0开始，左子树为（2*n+1），右子树为(2*n+2). 
链接策略：每一个结点存储的将是每一个孩子（可能还有双亲）的数组索引，而不是指向其孩子（可能还有双亲）的指针对象的引用变量，但是该方式增加了删除树中元素的成本。

#### 2.2 树的链表实现

链表策略：每一结点都可以定义一个TreeNode类，这与我们为连彼岸的LinearNode类所做的处理类似。每一个结点都包含一个指针，它指向将要存储在该结点的元素以及所有可能的孩子的指针，也可以在每一个结点中存储指向其双亲的指针。

### **3.树的遍历**

遍历四棵树有四种基本方法：

- 前序遍历：从根结点开始访问每一结点极其孩子。 
  Visit node 
  Traverse(left child) 
  Traverse(right child)
- 中序遍历：从根节点开始，访问结点的左孩子，然后是该结点，再然后是任何剩余结点。 
  Traverse(left child) 
  Visit node 
  Traverse(right child)
- 后序遍历：从根结点开始，访问结点的孩子，然后是该结点。 
  Traverse(left child) 
  Traverse(right child) 
  Visit node
- 层序遍历：从根结点开始，访问每一层的所有结点，一层一层。 
  Create a queue called *nodes* 
  Create an unordered list called **results** 
  Enqueue the root onto the *nodes* queue 
  While the *nodes* queue is not empty 
  { 
  Dequeue the first element from the *nodes* queue 
  If that element is not null 
  Add that element to the rear of the **results** list 
  Enqueue the children of the element on the *nodes* queue 
  Else 
  Add null on the **result** list 
  }

Return an iterator for the **result** list

### **4.简单二叉树ADT**

```Java
package ds.java.ch10;

import java.util.Iterator;

/** 
 * @author LbZhang
 * @version 创建时间：2015年11月22日 上午10:51:51 
 * @description 二叉树的ADT
 */
public interface BinaryTreeADT<T>{

    /** 
     * Returns a reference to the root element 
     * @return a reference to the root
     */
    public T getRootElement();

    /** 
     * Returns true if this binary tree is empty and false otherwise.
     * @return true if this binary tree is empty, false otherwise
     */
    public boolean isEmpty();

    /** 
     * Returns the number of elements in this binary tree.
     * @return the number of elements in the tree
     */
    public int size();

    /** 
     * Returns true if the binary tree contains an element that matches
     * the specified element and false otherwise. 
     * @param targetElement the element being sought in the tree
     * @return true if the tree contains the target element
     */
    public boolean contains(T targetElement);

    /** 
     * Returns a reference to the specified element if it is found in 
     * this binary tree.  Throws an exception if the specified element
     * is not found.
     * @param targetElement the element being sought in the tree
     * @return a reference to the specified element
     */
    public T find(T targetElement);

    /**  
     * Returns the string representation of this binary tree.
     * @return a string representation of the binary tree
     */
    public String toString();

    /**  
     * Returns an iterator over the elements of this tree.
     * @return an iterator over the elements of this binary tree
     */
    public Iterator<T> iterator();

    /**  
     * Returns an iterator that represents an inorder traversal on this binary tree.  
     * @return an iterator over the elements of this binary tree
     */
    public Iterator<T> iteratorInOrder();

    /**  
     * Returns an iterator that represents a preorder traversal on this binary tree. 
     * @return an iterator over the elements of this binary tree
     */
    public Iterator<T> iteratorPreOrder();

    /**  
     * Returns an iterator that represents a postorder traversal on this binary tree. 
     * @return an iterator over the elements of this binary tree
     */
    public Iterator<T> iteratorPostOrder();

    /**  
     * Returns an iterator that represents a levelorder traversal on the binary tree.
     * @return an iterator over the elements of this binary tree
     */
    public Iterator<T> iteratorLevelOrder();

}
```

### **5. 二叉树类的定义**

#### 5.1 二叉树的结点类的构建

```Java
package ds.java.ch10;

/**
 * @author LbZhang
 * @version 创建时间：2015年11月22日 上午10:54:19
 * @description 二叉树结点类
 */
public class BinaryTreeNode<T> {

    protected T element;
    protected BinaryTreeNode<T> left;
    protected BinaryTreeNode<T> right;

    /**
     * Creates a new tree node with the specified data.
     * 
     * @param obj
     *            the element that will become a part of the new tree node
     */
    public BinaryTreeNode(T obj) {
        this.element = obj;
        this.left = null;
        this.right = null;
    }


    public BinaryTreeNode(T obj, LinkedBinaryTree<T> left,
            LinkedBinaryTree<T> right) {
        element = obj;
        if (left == null)
            this.left = null;
        else
            this.left = left.getRootNode();

        if (right == null)
            this.right = null;
        else
            this.right = right.getRootNode();
    }

    /**
     * Returns the number of non-null children of this node.
     * @return the integer number of non-null children of this node
     * 使用递归的方式
     */
    public int numChildren() {
        int children = 0;

        if (left != null)
            children = 1 + left.numChildren();

        if (right != null)
            children = children + 1 + right.numChildren();

        return children;
    }

    /**
     * Return the element at this node.
     * @return the element stored at this node
     */
    public T getElement() {
        return element;
    }

    public BinaryTreeNode<T> getRight() {
        return right;
    }

    public void setRight(BinaryTreeNode<T> node) {
        right = node;
    }

    public BinaryTreeNode<T> getLeft() {
        return left;
    }

    public void setLeft(BinaryTreeNode<T> node) {
        left = node;
    }

}
```

#### 5.2 二叉树类的构建

```Java
package ds.java.ch10;

import java.util.ConcurrentModificationException;
import java.util.Iterator;
import java.util.NoSuchElementException;

import ds.java.ch05.queueImpl.EmptyCollectionException;
import ds.java.ch06.listImpl.ArrayUnorderedList;
import ds.java.ch06.listImpl.ElementNotFoundException;

/**
 * @author LbZhang
 * @version 创建时间：2015年11月22日 上午11:04:36
 * @description 二叉树的构建
 */
public class LinkedBinaryTree<T> implements Iterable<T>, BinaryTreeADT<T> {

    // 根结点的设置
    protected BinaryTreeNode<T> root;
    protected int modCount;// 修改标记 用于Iterator中使用

    // protected int sizeOfLTree;

    /**
     * 无参构造方法
     */
    public LinkedBinaryTree() {
        root = null;
    }

    public LinkedBinaryTree(T element) {
        root = new BinaryTreeNode<T>(element);
    }

    public LinkedBinaryTree(BinaryTreeNode<T> ltn) {
        this.root = ltn;
    }

    public LinkedBinaryTree(T element, LinkedBinaryTree<T> left,
            LinkedBinaryTree<T> right) {
        root = new BinaryTreeNode<T>(element);
        root.setLeft(left.root);
        root.setRight(right.root);
    }

    /**
     * 获取根节点
     * 
     * @return
     */
    public BinaryTreeNode<T> getRootNode() throws EmptyCollectionException {
        if (isEmpty()) {
            throw new EmptyCollectionException("BinaryTreeNode ");
        }
        return root;
    }

    /**
     * Returns the left subtree of the root of this tree.
     * 
     * @return a link to the right subtree of the tree
     */
    public LinkedBinaryTree<T> getLeft() {
        return new LinkedBinaryTree(this.root.getLeft());
        // To be completed as a Programming Project
    }

    /**
     * Returns the right subtree of the root of this tree.
     * 
     * @return a link to the right subtree of the tree
     */
    public LinkedBinaryTree<T> getRight() {
        return new LinkedBinaryTree(this.root.getRight());
    }

    /**
     * Returns the height of this tree.
     * 
     * @return the height of the tree
     */
    public int getHeight() {
        return 0;
        // To be completed as a Programming Project
    }

    /**
     * Returns the height of the specified node.
     * 
     * @param node
     *            the node from which to calculate the height
     * @return the height of the tree
     */
    private int height(BinaryTreeNode<T> node) {
        return 0;
        // To be completed as a Programming Project
    }

    @Override
    public T getRootElement() throws EmptyCollectionException {
        if (root.getElement().equals(null)) {
            throw new EmptyCollectionException("BinaryTreeNode ");
        }
        return root.getElement();
    }

    @Override
    public boolean isEmpty() {
        return (root == null);
    }

    @Override
    public int size() {
        return 0;
    }

    @Override
    public boolean contains(T targetElement) {
        return false;
    }

    @Override
    public T find(T targetElement) {
        // 获取当前元素
        BinaryTreeNode<T> current = findNode(targetElement, root);

        if (current == null)
            throw new ElementNotFoundException("LinkedBinaryTree");

        return (current.getElement());
    }

    private BinaryTreeNode<T> findNode(T targetElement, BinaryTreeNode<T> next) {
        if (next == null)
            return null;

        if (next.getElement().equals(targetElement))
            return next;
        // 递归调用
        BinaryTreeNode<T> temp = findNode(targetElement, next.getLeft());

        if (temp == null)
            temp = findNode(targetElement, next.getRight());

        return temp;
    }

    @Override
    public Iterator<T> iteratorInOrder() {
        ArrayUnorderedList<T> tempList = new ArrayUnorderedList<T>();
        inOrder(root, tempList);
        return new TreeIterator(tempList.iterator());
    }

    /**
     * Performs a recursive inorder traversal. 中根遍历
     * 
     * @param node
     *            the node to be used as the root for this traversal
     * @param tempList
     *            the temporary list for use in this traversal
     */
    protected void inOrder(BinaryTreeNode<T> node,
            ArrayUnorderedList<T> tempList) {
        if (node != null) {
            inOrder(node.getLeft(), tempList);
            tempList.addToRear(node.getElement());
            inOrder(node.getRight(), tempList);
        }
    }

    /**
     * Performs an preorder traversal on this binary tree by calling an
     * overloaded, recursive preorder method that starts with the root.
     * 
     * @return a pre order iterator over this tree
     */
    @Override
    public Iterator<T> iteratorPreOrder() {
        ArrayUnorderedList<T> tempList = new ArrayUnorderedList<T>();
        preOrder(root, tempList);
        return new TreeIterator(tempList.iterator());
    }

    /**
     * Performs a recursive preorder traversal.
     * 
     * @param node
     *            the node to be used as the root for this traversal
     * @param tempList
     *            the temporary list for use in this traversal
     */
    private void preOrder(BinaryTreeNode<T> node, ArrayUnorderedList<T> tempList) {
        if (node != null) {
            tempList.addToRear(node.getElement());
            inOrder(node.getLeft(), tempList);

            inOrder(node.getRight(), tempList);
        }

    }

    /**
     * Performs an postorder traversal on this binary tree by calling an
     * overloaded, recursive postorder method that starts with the root.
     * 
     * @return a post order iterator over this tree
     */
    @Override
    public Iterator<T> iteratorPostOrder() {
        ArrayUnorderedList<T> tempList = new ArrayUnorderedList<T>();
        postOrder(root, tempList);
        return new TreeIterator(tempList.iterator());
    }

    /**
     * Performs a recursive postorder traversal.
     * 
     * @param node
     *            the node to be used as the root for this traversal
     * @param tempList
     *            the temporary list for use in this traversal
     */
    private void postOrder(BinaryTreeNode<T> node,
            ArrayUnorderedList<T> tempList) {

        if (node != null) {
            tempList.addToRear(node.getElement());
            inOrder(node.getLeft(), tempList);

            inOrder(node.getRight(), tempList);
        }

    }

    @Override
    public Iterator<T> iteratorLevelOrder() {
        ArrayUnorderedList<BinaryTreeNode<T>> nodes = new ArrayUnorderedList<BinaryTreeNode<T>>();
        ArrayUnorderedList<T> tempList = new ArrayUnorderedList<T>();
        BinaryTreeNode<T> current;

        nodes.addToRear(root);

        while (!nodes.isEmpty()) {
            current = nodes.removeFirst();

            if (current != null) {
                tempList.addToRear(current.getElement());
                if (current.getLeft() != null)
                    nodes.addToRear(current.getLeft());
                if (current.getRight() != null)
                    nodes.addToRear(current.getRight());
            } else
                tempList.addToRear(null);
        }

        return new TreeIterator(tempList.iterator());
    }

    @Override
    public Iterator<T> iterator() {
        return iteratorInOrder();
    }

    @Override
    public String toString() {
        // TODO Auto-generated method stub
        return super.toString();
    }

    /**
     * Inner class to represent an iterator over the elements of this tree
     */
    private class TreeIterator implements Iterator<T> {
        private int expectedModCount;
        private Iterator<T> iter;

        /**
         * Sets up this iterator using the specified iterator.
         * 
         * @param iter
         *            the list iterator created by a tree traversal
         */
        public TreeIterator(Iterator<T> iter) {
            this.iter = iter;
            expectedModCount = modCount;
        }

        /**
         * Returns true if this iterator has at least one more element to
         * deliver in the iteration.
         * 
         * @return true if this iterator has at least one more element to
         *         deliver in the iteration
         * @throws ConcurrentModificationException
         *             if the collection has changed while the iterator is in
         *             use
         */
        public boolean hasNext() throws ConcurrentModificationException {
            if (!(modCount == expectedModCount))
                throw new ConcurrentModificationException();

            return (iter.hasNext());
        }

        /**
         * Returns the next element in the iteration. If there are no more
         * elements in this iteration, a NoSuchElementException is thrown.
         * 
         * @return the next element in the iteration
         * @throws NoSuchElementException
         *             if the iterator is empty
         */
        public T next() throws NoSuchElementException {
            if (hasNext())
                return (iter.next());
            else
                throw new NoSuchElementException();
        }

        /**
         * The remove operation is not supported.
         * 
         * @throws UnsupportedOperationException
         *             if the remove operation is called
         */
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
}
```

### **6. 二叉树类图结构**

![二叉树类图结构](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174131.png)

踏实 踏踏实实~

https://www.cnblogs.com/mrzhang123/p/5365827.html