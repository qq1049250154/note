英文原文：https://www.codeproject.com/Articles/1158559/B-Tree-Another-Implementation-By-Java

本文对原文做了少许修改，并借鉴了其部分代码完成了B树的java实现

# 1.介绍

B-Tree应该是大多数像我一样以计算机科学为学习专业的大学生所熟悉的。其最初目的是通过尽可能减少存储I / O操作来减少在计算机硬盘驱动器上花费的时间。该技术在数据库和文件系统等计算机领域中发挥了很好的作用。大数据和NoSQL分布式数据库系统（由于廉价的硬件和互联网增长）B-Tree及其变体在数据存储方面发挥着前所未有的重要作用。

在本文中，我不讨论B-Tree操作的性能和时间测量。相反，我会尽最大努力花大部分时间在本文中讨论和解释如何使用Java来完成B-Tree数据结构和实现。

# 2.什么是B树？

B-Tree是一种自平衡树（即所有叶节点具有相同的高度级别）数据结构。然而，与其他树，例如二进制树，红黑和AVL树不同，其节点只有2个子节点：左子节点和右子节点，而B-Tree节点具有2个以上的子节点。因此，有时它被称为M-way分支树，因为B-Tree中的节点可以具有M个子节点（M> = 2）。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174218.png)图1： B树的一个例子

- ### **根节点，内部节点和叶节点**

内部节点是具有子节点的节点。内部节点位于树的底部上方。在图1中，节点[3 | 6]，节点[12 | 15 | 19 | 26]和节点[33 | 36]是内部节点。

叶节点是没有子节点的节点。它们是树底部的节点。在图1中，节点[1 | 2]，节点[4 | 5]，节点[20 | 22 | 25]和节点[31 | 32]是一些叶节点。

B-Tree的根节点是一个特殊节点。B-Tree只有一个根节点，它位于树的顶部。根据B-Tree中的项目数，根节点可以是内部节点或叶节点。节点[9 | 30]是图1中B树的根节点。

- ### **B树节点的属性**

每个节点可以有一堆键和一堆子节点（子节点），其中子节点数可以是0或其键的总数加1。让我们考虑节点[x]。如果node [x]是叶子节点，那么它将没有任何子节点。如果它是一个内部节点，那么它的子节点总数是n [x] + 1，其中n [x]是其键的总数。

- ### **B-Tree的约束**

令***t\***为B树的最小度，其中***t\*** >= 3**（原文为\*t\* >= 2，本文为编程实现简单，故取此）**

***约束＃1\*：**除根节点以外的每个节点必须至少有（**t** -1）个键。它是B-Tree节点中键总数的下限。

***约束＃2\*：**包含根节点的每个节点必须至多具有（2 ***t\*** - 1）个键。所以我们说如果节点有（2 ***t\*** - 1）个键，那么节点已满。它是B-Tree节点中键总数的上限。

***约束＃3\*：**每个内部节点（根节点除外）必须至少有***t\***个子节点。每个内部节点（包括根节点）必须具有至多2***t\***的子结点。

***约束＃4\*：**节点中的键必须按升序存储。例如，在图1中，节点[12 | 15 | 19 | 26]按键12 <键15 <键19 <键26

***约束＃5\*：**键左侧的子节点的所有键必须小于该键。在图1中，位于密钥30左侧的子节点是节点[12 | 15 | 19 | 26]，节点[10 | 11]，节点[13 | 14]，节点[16 | 18]，节点[20 | 22 | 25]和节点[28 | 29]，他们的键均小于30。

***约束＃6\*：**键右侧的子节点的所有键必须大于该键。例如，在图1中，位于键9右侧的子节点是节点[12 | 15 | 19 | 26]，节点[10 | 11]，节点[13 | 14]，节点[16 | 18]，节点[20 | 22 | 25]和节点[28 | 29]，他们的键大于9。

对于*约束＃4*和*约束＃5*，通常在一个键的左侧的所有节点必须具有小于它的键。

对于*约束＃4*和*约束＃6*，通常在一个键的右侧的所有节点都必须具有大于它的键。

在图1中，B树的最小度为3。因此其键总数的下限为2，上限为5。

如果你密切关注图1中B-Tree的例子，你会注意到任何节点中的一个键实际上是该键左侧和右侧较低级别节点中所有键的范围分隔符。 

让我们看一下节点[9 | 30]所示，键9左侧的下部节点的键（由蓝色箭头链接的节点中的键）小于9。键9右侧的下部节点的键（由绿色箭头链接的节点中的键）大于9。键30左侧的较低节点的键（由绿色箭头链接的节点中的键）小于30。键30右侧的较低节点的键（由红色箭头链接的节点中的键）大于30。这种B-Tree模式使得键搜索类似于二叉树的键搜索。

只要B树操作不违反上述约束，它就会自动进行自我平衡。换句话说，约束是这样设计的，以保持其平衡属性。

最小度数（**t**）与B树高度（**h**）成反比。增加**t**将减少**h**。但是，较大的**t**意味着在节点中花费更多时间来搜索键和遍历子节点。

# 3.B-Tree操作的概念和代码

- ### 结点的实现

在第二节，我们了解了B树的概念和它的约束条件，现在我们给出B树中结点的代码如下（其中的静态方法暂时可先忽略）:

```java
public class BTNode<K extends Comparable<K>, V> {
 
	// 构成B树的最小度数
	public final static int MIN_DEGREE = 3;
	// 除根节点外，每个结点中总键数的下限
	public final static int LOWER_BOUND_KEYNUM = MIN_DEGREE - 1;
	// 包含根节点外，每个结点中总键数的上限
	public final static int UPPER_BOUND_KEYNUM = (MIN_DEGREE * 2) - 1;
 
	protected boolean mIsLeaf;// 标记此节点是否为叶子结点
	protected int mCurrentKeyNum;// 此节点的键数量计数器
	protected BTKeyValue<K, V>[] mKeys;// 用于存键值对的数组
	protected BTNode<K, V>[] mChildren;// 用于存子结点的数组
 
	/**
	 * 构造函数
	 */
	@SuppressWarnings("unchecked")
	public BTNode() {
		mIsLeaf = true;
		mCurrentKeyNum = 0;
		mKeys = new BTKeyValue[UPPER_BOUND_KEYNUM];
		mChildren = new BTNode[UPPER_BOUND_KEYNUM + 1];
	}
 
	protected static BTNode<?, ?> getChildNodeAtIndex(BTNode<?, ?> btNode, int keyIdx, int nDirection) {
		if (btNode.mIsLeaf) {
			return null;
		}
		keyIdx += nDirection;
		if ((keyIdx < 0) || (keyIdx > btNode.mCurrentKeyNum)) {
			throw new IllegalArgumentException();
		}
 
		return btNode.mChildren[keyIdx];
	}
 
	/**
	 * 返回btNode节点中位于keyIdx位上的键左边的子结点
	 * @param btNode
	 * @param keyIdx
	 * @return
	 */
	protected static BTNode<?, ?> getLeftChildAtIndex(BTNode<?, ?> btNode, int keyIdx) {
		return getChildNodeAtIndex(btNode, keyIdx, 0);
	}
 
	/**
	 * 返回btNode节点中位于keyIdx位上的键右边的子结点
	 * @param btNode
	 * @param keyIdx
	 * @return
	 */
	protected static BTNode<?, ?> getRightChildAtIndex(BTNode<?, ?> btNode, int keyIdx) {
		return getChildNodeAtIndex(btNode, keyIdx, 1);
	}
 
	/**
	 * @param parentNode
	 * @param keyIdx
	 * @return 返回父结点的keyIdx位上的子结点的左兄弟结点
	 */
	protected static BTNode<?, ?> getLeftSiblingAtIndex(BTNode<?, ?> parentNode, int keyIdx) {
		return getChildNodeAtIndex(parentNode, keyIdx, -1);
	}
 
	/**
	 * 
	 * @param parentNode
	 * @param keyIdx
	 * @return	返回父结点的keyIdx位上的子结点的右兄弟结点
	 */
	protected static BTNode<?, ?> getRightSiblingAtIndex(BTNode<?, ?> parentNode, int keyIdx) {
		return getChildNodeAtIndex(parentNode, keyIdx, 1);
	}
	
	
	/**
	 * 判断父结点的keyIdx位上的子结点是否存在左兄弟结点
	 * @param parentNode
	 * @param keyIdx
	 * @return
	 */
	protected static boolean hasLeftSiblingAtIndex(BTNode<?, ?> parentNode, int keyIdx) {
		if (keyIdx - 1 < 0) {
			return false;
		} else {
			return true;
		}
	}
	
	/**
	 * 判断父结点的keyIdx位上的子结点是否存在右兄弟结点
	 * @param parentNode
	 * @param keyIdx
	 * @return
	 */
	protected static boolean hasRightSiblingAtIndex(BTNode<?, ?> parentNode, int keyIdx) {
		if (keyIdx + 1 > parentNode.mCurrentKeyNum) {
			return false;
		} else {
			return true;
		}
	}
}
```

其中用于存键值对的代码如下：

```java
/**
 * @author Herry
 *
 * @param <K>
 * @param <V>
 */
public class BTKeyValue<K extends Comparable<K>, V> {
 
	protected K mKey;
	protected V mValue;
 
	public BTKeyValue(K mKey, V mValue) {
		super();
		this.mKey = mKey;
		this.mValue = mValue;
	}
}
```

为了理解如何实现基本的B-Tree操作，如键搜索，插入和删除，我想在介绍完整的实现细节之前介绍一些关键概念。

- ### ***左右兄弟结点\***

节点的左右兄弟是其右侧和左侧的节点处于同一级别。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174258.png)图2：兄弟结点

在图2中，节点[18 | 22]的左兄弟结点是节点[3 | 12]和它的右兄弟结点是节点[33 | 36]。节点[3 | 12]没有左兄弟结点，它的右兄弟结点是节点[18 | 22]。节点[33 | 36]的左兄弟结点是节点[18 | 22]，并且它没有右兄弟结点。

- ### ***（内部）节点的键的前任和后继\***

在本文中，此处提到的指定节点中的键的前任和后继仅适用于内部节点。

前任是键的左侧子树内的叶节点，它包含其值是该子树中最大值的键。

对称地，后继是键右侧子树内的叶节点，它包含其值是该子树中最小值的键。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174301.png)图3：键的前任和后继

在图3中，17的前身是节点[13 | 14 | 因为键16是其左侧最大的键。密钥17的后继者是节点[19 | 20 | 因为键19是其右侧的最小键。

- ### ***拆分节点\***

对于插入，我们每次会将键插入到叶子节点中（除了在内部结点找到了插入的键，这种情况下直接将原键对应的值替换即可），而结点中键的数量需要满足它的上、下限要求（见第二节BTree树的约束），故当叶子结点中键的数量达到上限时，需要将其拆分再将键插入。其过程示意图如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174303.png)图4-a： 在B树中分裂

分裂后的结点会导致树的高度不一致，故而影响平衡性，因此将分裂后的顶点结点合并到父结点中去，具体的过程如下图所示：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174305.png)图4-b： B-Tree中的另一个分裂

 结点拆分的代码实现如下：

```java
		/**
	 * 将满结点x分裂为含有一个键值对的父结点和两个子结点，并返回父结点的链接
	 * 
	 * @param x
	 * @return
	 */
	private BTNode<K, V> split(BTNode<K, V> x) {
		int mid = x.mCurrentKeyNum / 2;
 
		BTNode<K, V> left = new BTNode<>();
		for (int i = 0; i < mid; i++) {
			left.mKeys[i] = x.mKeys[i];
			left.mChildren[i] = x.mChildren[i];
		}
		left.mChildren[mid] = x.mChildren[mid];
		left.mIsLeaf = x.mIsLeaf;
		left.mCurrentKeyNum = mid;
 
		BTNode<K, V> right = new BTNode<>();
		for (int i = mid + 1; i < x.mCurrentKeyNum; i++) {
			right.mKeys[i - mid - 1] = x.mKeys[i];
			right.mChildren[i - mid - 1] = x.mChildren[i];
		}
		right.mChildren[x.mCurrentKeyNum - mid - 1] = x.mChildren[x.mCurrentKeyNum];
		right.mIsLeaf = x.mIsLeaf;
		right.mCurrentKeyNum = x.mCurrentKeyNum - mid - 1;
 
		BTNode<K, V> top = new BTNode<>();
		top.mCurrentKeyNum = 1;
		top.mIsLeaf = false;
		top.mKeys[0] = x.mKeys[mid];
		top.mChildren[0] = left;
		top.mChildren[1] = right;
		return top;
	}
```

- ### *左右旋转*

***注：与原文不同，本文将向左结点借键称为左旋转，将向右结点借键称为右旋转***

对于删除，从节点中删除键可能违反约束＃1（有时也称为下溢）。如果当删除了键的结点的键数量下溢时，同时它的其中一个兄弟结点的键数大于下限，我们可以将其兄弟中的一个键移动（借用）到该节点。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174324.png)图5-a： 右旋转

如图5-a，结点[4，5]借用结点[10，12，15，20]中的键，起始就是结点[4，5]借用其右节点的键，我们将其称为右旋转。图5-b同理。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174329.png)图5-b： 左旋转

左右旋转的代码实现如下：

```java
	/**
	 * 从右结点借一个键值对过来
	 * 
	 * @param x
	 * @param possibleIdx
	 * @return
	 */
	private BTNode<K, V> rightRotate(BTNode<K, V> x, int possibleIdx) {
 
		// 获取右节点和右节点中最小的键值对
		BTNode<K, V> rightNode = x.mChildren[possibleIdx + 1];
		BTKeyValue<K, V> rightKey = rightNode.mKeys[0];
		// 获取右节点中最小的结点
		BTNode<K, V> rightFirstNode = rightNode.mChildren[0];
		// 获取父结点交换位置的键值对
		BTKeyValue<K, V> topKey = x.mKeys[possibleIdx];
		// 获取需补齐键值对的节点，并将父结点交换位置的键值对加到此节点的最高位
		BTNode<K, V> possibleNode = x.mChildren[possibleIdx];
		possibleNode.mKeys[possibleNode.mCurrentKeyNum] = topKey;
		// 将右节点中最小的结点添加到此节点
		possibleNode.mChildren[possibleNode.mCurrentKeyNum + 1] = rightFirstNode;
		possibleNode.mCurrentKeyNum++;
 
		// 将父结点拿走键值对的位置填上右节点提出的键值对
		x.mKeys[possibleIdx] = rightKey;
		// 将右节点提出的键值对和最小结点在右节点中删除
		for (int i = 1; i < rightNode.mCurrentKeyNum; i++) {
			rightNode.mKeys[i - 1] = rightNode.mKeys[i];
		}
		rightNode.mKeys[rightNode.mCurrentKeyNum - 1] = null;
		for (int i = 1; i < rightNode.mCurrentKeyNum + 1; i++) {
			rightNode.mChildren[i - 1] = rightNode.mChildren[i];
		}
		rightNode.mChildren[rightNode.mCurrentKeyNum] = null;
		rightNode.mCurrentKeyNum--;
//		System.out.println("rightRotate executed");
		return x;
	}
 
	/**
	 * ‘
	 * 
	 * @param x           父结点
	 * @param possibleIdx 需要补充键值对的子结点的索引
	 * @return
	 */
	private BTNode<K, V> leftRotate(BTNode<K, V> x, int possibleIdx) {
 
		// 获取左节点和左节点中最大的键值对
		BTNode<K, V> leftNode = x.mChildren[possibleIdx - 1];
		BTKeyValue<K, V> leftKey = leftNode.mKeys[leftNode.mCurrentKeyNum - 1];
		// 获取左节点中最大的结点
		BTNode<K, V> leftLastNode = leftNode.mChildren[leftNode.mCurrentKeyNum];
		// 获取父结点交换位置的键值对
		BTKeyValue<K, V> topKey = x.mKeys[possibleIdx - 1];
		// 获取需补齐键值对的节点，并移动其中的键值对将最低位空出来：以用来填充从父结点交换过来的键值对
		BTNode<K, V> possibleNode = x.mChildren[possibleIdx];
		for (int i = possibleNode.mCurrentKeyNum; i > 0; i--) {
			possibleNode.mKeys[i] = possibleNode.mKeys[i - 1];
		}
		// 同理对此节点的子结点
		for (int i = possibleNode.mCurrentKeyNum + 1; i > 0; i--) {
			possibleNode.mChildren[i] = possibleNode.mChildren[i - 1];
		}
		// 填充键值对和其带过来的链接，并将键数量计数器加1
		possibleNode.mKeys[0] = topKey;
		possibleNode.mChildren[0] = leftLastNode;
		possibleNode.mCurrentKeyNum++;
		// 将父结点拿走键值对的位置填上左节点提出的键值对
		x.mKeys[possibleIdx - 1] = leftKey;
		// 将左节点提出的键值对和子结点在左节点中删除
		leftNode.mKeys[leftNode.mCurrentKeyNum - 1] = null;
		leftNode.mChildren[leftNode.mCurrentKeyNum] = null;
		leftNode.mCurrentKeyNum--;
//		System.out.println("leftRotate executed");
		return x;
	}
```

- ### 兄弟结点的合并

当删除了键的结点的键数量下溢时，同时它的其中一个兄弟结点的键数小于等于下限（此时借用兄弟结点的键会导致，兄弟结点的键下溢），因此我们需要将删除了键的结点和其键数不大于下限的兄弟结点合并。如下图：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174346.png)图6：与左兄弟结点合并

**图6中结点[12]与其左兄弟结点[3，6，9]合并等价于结点[3，6，9]与其右兄弟结点[12]合并。这点很重要，他意味着，我们在写代码的时候可以将左右合并互相转换。**

左右合并的代码的实现如下：

```java
		/**
	 * 合并父结点中位于possibleIdx和possibleIdx+1处的俩结点（由此可见可用执行做合并来完成右合并同样的任务）
	 * 
	 * @param x
	 * @param possibleIdx
	 * @return
	 */
	private BTNode<K, V> rightMerge(BTNode<K, V> x, int possibleIdx) {
		return leftMerge(x, possibleIdx + 1);
	}
 
	/**
	 * 合并父结点中位于possibleIdx和possibleIdx-1处的俩结点
	 * 
	 * @param x
	 * @param possibleIdx
	 * @return
	 */
	private BTNode<K, V> leftMerge(BTNode<K, V> x, int possibleIdx) {
		// 获取左节点
		BTNode<K, V> leftNode = x.mChildren[possibleIdx - 1];
		// 获取父结点要合并到左节点的键值对
		BTKeyValue<K, V> topKey = x.mKeys[possibleIdx - 1];
		// 获取需要合并的结点
		BTNode<K, V> possibleNode = x.mChildren[possibleIdx];
		// 将父结点获取的键值对放入左节点
		leftNode.mKeys[leftNode.mCurrentKeyNum] = topKey;
		// 将需要合并的结点的键值对全部放入左节点
		for (int i = 0; i < possibleNode.mCurrentKeyNum; i++) {
			leftNode.mKeys[i + leftNode.mCurrentKeyNum + 1] = possibleNode.mKeys[i];
		}
		// 同理，将结点链接也移过来
		for (int i = 0; i < possibleNode.mCurrentKeyNum + 1; i++) {
			leftNode.mChildren[i + leftNode.mCurrentKeyNum + 1] = possibleNode.mChildren[i];
		}
		// 更新左节点的键值对计数器
		leftNode.mCurrentKeyNum += 1 + possibleNode.mCurrentKeyNum;
		// 更新父结点
		for (int i = possibleIdx; i < x.mCurrentKeyNum; i++) {
			x.mKeys[i - 1] = x.mKeys[i];
		}
		x.mKeys[x.mCurrentKeyNum - 1] = null;
		for (int i = possibleIdx; i < x.mCurrentKeyNum; i++) {
			x.mChildren[i] = x.mChildren[i + 1];
		}
		x.mChildren[x.mCurrentKeyNum] = null;
		x.mCurrentKeyNum--;
//		System.out.println("leftMerge executed");
		return x;
	}
```

- ###  最大键和最小键

求取指定结点的最大键和最小键比较简单，直接贴代码：

```java
		/**
	 * @return
	 */
	public BTKeyValue<K, V> minKey() {
		return minKey(mRoot);
	}
 
	/**
	 * @param x
	 * @return
	 */
	private BTKeyValue<K, V> minKey(BTNode<K, V> x) {
		if (x == null) {
			return null;
		}
		if (x.mChildren[0] != null) {
			return minKey(x.mChildren[0]);
		} else {
			return x.mKeys[0];
		}
	}
 
	/**
	 * @return
	 */
	public BTKeyValue<K, V> maxKey() {
		return maxKey(mRoot);
	}
 
	/**
	 * @param x
	 * @return
	 */
	private BTKeyValue<K, V> maxKey(BTNode<K, V> x) {
		if (x == null) {
			return null;
		}
		if (x.mChildren[x.mCurrentKeyNum] != null) {
			return maxKey(x.mChildren[x.mCurrentKeyNum]);
		} else {
			return x.mKeys[x.mCurrentKeyNum - 1];
		}
	}
```

## 3.1 实现B树操作的一般概念

键搜索B-Tree简单明了。如果您熟悉二叉树搜索，那么您可以毫无问题地实现它。然而，与二叉树不同，在B-Tree中，您需要查看节点中的键数组，以验证搜索到的键是否在该节点中。如果不是，我们需要决定选择哪个子节点进一步查看。更复杂的逻辑用于键插入，最复杂的逻辑用于键删除。然而，在高级别，B-Tree的插入和删除非常容易捕获：键插入涉及拆分节点，并且键删除涉及旋转和合并。这很简单。为了使键插入和删除成为可能，我们需要注意的一件重要事情是：在对指定键执行这些操作之前，它必须位于叶节点中。如果密钥位于内部节点中，我们需要将其交换为叶节点中的适当密钥。使用该策略，我们不必在添加或删除密钥后担心节点的子节点。

## 3.2 B树的Java实现

- **键搜索**

BTree搜索与二叉树不同，在B-Tree中，您需要查看节点中的键数组，以验证搜索到的键是否在该节点中。在这里，我们定义一个方法，用二分查找来实现指定结点中对特定键的搜索，它返回一个索引，若在该结点中查找到此键，则该索引就是此键的位置，若没查到，则该索引代表的时此键应该插入到该节点的位置（即该键应该位于索引处的子结点中）。其代码实现如下：

```java
		/**
	 * 用二分查找法查找结点中键的位置，若找到返回键的位置，若没找到，则返回键应该插入的位置
	 * 
	 * @param btNode
	 * @param key
	 * @return
	 */
	private int binarySearch(BTNode<K, V> btNode, K key) {
		BTKeyValue<K, V>[] keys = btNode.mKeys;
		int lo = 0;
		int hi = btNode.mCurrentKeyNum - 1;
		while (lo <= hi) {
			int mid = (hi - lo) / 2 + lo;
			int cmp = key.compareTo(keys[mid].mKey);
			if (cmp == 0) {
				return mid;
			} else if (cmp > 0) {
				lo = mid + 1;
			} else if (cmp < 0) {
				hi = mid - 1;
			}
		}
		return lo;
	}
```

解决了在指定节点中搜索给定键，下面我们就可以很轻松的用迭代法从根节点开始搜索给定点，具体代码如下：

```java
		/**
	 * 查找指定键所对应的值
	 * 
	 * @param key
	 * @return 若键存在，则返回键所对应的值。若键不存在，则抛出异常。
	 */
	public V search(K key) {
 
		checkKey(key);
 
		BTNode<K, V> currentNode = mRoot;
 
		// 迭代查找每一个可能存储key的结点
		while (currentNode != null) {
			int possibleIdx = binarySearch(currentNode, key);
			BTKeyValue<K, V> possibleKeyValue = null;
			// 判断二分查找返回位置索引处的元素是否为查找的元素，若是则返回其值，如不是，则迭代到下一个可能的结点中查找
			if (possibleIdx < currentNode.mCurrentKeyNum && key.compareTo(
                                                (possibleKeyValue = currentNode.mKeys[possibleIdx]).mKey) == 0) {
				return possibleKeyValue.mValue;
			} else {
				currentNode = currentNode.mChildren[possibleIdx];
			}
		}
		return null;
	}
```

- **键插入**

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174444.png)图7：键插入的示例

前边提到，键插入时需要直接插入到叶子结点（排除在内结点中找到了要插入的键的情况），并且，在插入之前先判断叶子结点中的键数量是否达到了上限，若是，则分裂结点，并将分裂后的顶结点合并到父结点中，这时，父结点中键的数量有可能也会达到上限，不过不用但心，我们用递归的方法去向结点插入键，每次递归向结点插入键前先判断此节点是否为满，若满则，先分裂，再插入，若未满则递归插入。代码实现如下：

```java
		/**
	 * 将键-值对插入到BTree结构中
	 * 
	 * @param key   键不允许为null
	 * @param value
	 */
	public void insert(K key, V value) {
 
		checkKey(key);
 
		if (mRoot == null) {
			mRoot = createNode();
		}
		// 使用递归的方法将键-值对插入到BTree结构中
		mRoot = insert(mRoot, key, value);
 
	}
 
	/**
	 * 递归插入方法
	 * 
	 * @param x     要插入到的结点
	 * @param key
	 * @param value
	 * @return
	 */
	private BTNode<K, V> insert(BTNode<K, V> x, K key, V value) {
 
		// 1.首先判断此节点是否已经为满，若满，则将此节点分裂
		if (x.mCurrentKeyNum == BTNode.UPPER_BOUND_KEYNUM) {
			x = split(x);
		}
		// 2.对没有满的结点进行键值对的查找，找出可能的键值对索引，和可能的键值对
		int possibleIdx = binarySearch(x, key);
		/*
		 * 由于第一步操作会确定当前节点为非满结点，故不用担心数组越界问题(不然试想，当此结点已满，且要插入的键大于此节点中所有键，
		 * 故possibleIdx的值会等于UPPER_BOUND_KEYNUM，故造成越界)
		 */
		BTKeyValue<K, V> possibleKeyValue = x.mKeys[possibleIdx];
		/*
		 * 3.判断可能的键值对中的键是否与要插入的键相同（当要插入的键大于当前结点中所有的键时，possibleKeyValue取值为x.mKeys[x.
		 * mCurrentKeyNum]为null，故要判断possibleKeyValue的值是否为空，以防止空指针异常）
		 * 如果相同则直接替换当前值为插入值，并返回当前结点(用于更新)
		 */
		if (possibleKeyValue != null && key.compareTo(possibleKeyValue.mKey) == 0) {
			possibleKeyValue.mValue = value;
			return x;
		}
		/*
		 * 4.当前节点为叶子节点时，直接插入（由于在最前边进行了当前结点是否为满的判断，并做了相应的处理，故到此步插入键值对后，此节点最多为满，且不会溢出）
		 * 当前结点不为叶子结点时，递归到下一个可能的结点继续寻找、插入
		 */
		if (x.mIsLeaf) {
			// 4.1
			for (int i = x.mCurrentKeyNum; i > possibleIdx; i--) {
				x.mKeys[i] = x.mKeys[i - 1];
			}
			x.mKeys[possibleIdx] = new BTKeyValue<>(key, value);
			x.mCurrentKeyNum++;
			mSize++;
		} else {
			// 4.2
			BTNode<K, V> t = insert(x.mChildren[possibleIdx], key, value);
			/*
			 * 4.3判断当返回的结点中的键值对数量为1时，说明返回的结点经过了分裂，故需要将其合并到当前节点中（同上理，合并后，当前结点最多为满）
			 */
			if (t.mCurrentKeyNum == 1) {
				// 4.3.1移动当前节点中的键值对为要合并的键值对腾出地方，并存入
				for (int i = x.mCurrentKeyNum; i > possibleIdx; i--) {
					x.mKeys[i] = x.mKeys[i - 1];
				}
				x.mKeys[possibleIdx] = t.mKeys[0];
				// 4.3.2移动当前节点中的子结点为要合并的子结点腾出地方，并存入
				for (int i = x.mCurrentKeyNum + 1; i > possibleIdx + 1; i--) {
					x.mChildren[i] = x.mChildren[i - 1];
				}
				x.mChildren[possibleIdx] = t.mChildren[0];
				x.mChildren[possibleIdx + 1] = t.mChildren[1];
				// 4.3.3更新当前结点的键值对计数器
				x.mCurrentKeyNum++;
			}
		}
		return x;
	}
```

- **键删除**

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174501.png)图8： 键删除的示例

键删除稍微的复杂一点，我们用递归删除的方法来删除键，并实时更新树的结构使其保持平衡。在当前结点删除指定键大体可分为两中情况

1. 当前结点为叶子结点，并且在其中找到的要删除的键（若没找到，直接返回，说明键不存在在树中），将找到的键删除。之后判断删除指定键后当前节点的键数是否满足约束条件，若满足，直接返回，若不满足，则执行适当的旋转和合并操作，使其保持平衡。
2. 当前结点为非叶子结点，且未在其中找到要删除的键，则递归到可能存在要删除键的结点去执行删除操作，若在当前结点找到了要删除的键，则将要删除的键的左链接中最小的键提出来替换当前键即可（提出键的操作可能会影响约束条件，故而需要重新平衡，处理方法为，先记住此最小键，然后执行递归删除此最小键--递归删除操作会自动平衡--，最后找出要删除的键，并替换）。

具体的代码实现如下：

```java
		/**
	 * 
	 * @param key
	 * @return
	 */
	public V deleteKey(K key) {
 
		checkKey(key);
 
		V v = search(key);
		// 递归的删除键key
		mRoot = deleteKey(mRoot, key);
		return v;
	}
 
	/**
	 * @param x
	 * @param key
	 * @return
	 */
	private BTNode<K, V> deleteKey(BTNode<K, V> x, K key) {
 
		// 1.获取要删除的键可能处在当前结点上的索引位置
		int possibleIdx = binarySearch(x, key);
 
		// 2.根据当前结点是否为叶子结点分情况讨论
		if (x.mIsLeaf == true) {
			// 2.1当前结点为叶子节点
 
			if (possibleIdx < x.mCurrentKeyNum && key.compareTo(x.mKeys[possibleIdx].mKey) == 0) {
				// 2.1.1判断在当前结点上possible索引位置上的键是否与要删除的键相等（前提是possible索引小于当前节点键的数量，负责会出现空指针异常）
				// 如果相等，则直接删除此键，否则，此键不存在树中，不做任何操作
 
				for (int i = possibleIdx; i < x.mCurrentKeyNum - 1; i++) {
					x.mKeys[i] = x.mKeys[i + 1];
				}
				x.mKeys[x.mCurrentKeyNum - 1] = null;
				x.mCurrentKeyNum--;
				mSize--;
			}
		} else {
			// 2.2当前结点不为子结点
			if (possibleIdx < x.mCurrentKeyNum && key.compareTo(x.mKeys[possibleIdx].mKey) == 0) {
				// 2.2.1判断在当前结点上possible索引位置上的键是否与要删除的键相等（前提是possible索引小于当前节点键的数量，负责会出现空指针异常）
				// 如果成立，用possible索引处的子结点的最大键替换要删除的键
 
				// 1）记住possilbe索引处子结点的最大键
				BTKeyValue<K, V> keyNeedToSwim = maxKey(x.mChildren[possibleIdx]);
 
				// 2）将1）中记住的键删除
				x = deleteKey(x, keyNeedToSwim.mKey);
 
				// 3）将key替换为记住的键
				possibleIdx = binarySearch(x, key);
				x.mKeys[possibleIdx] = keyNeedToSwim;
 
			} else {
				// 2.2.2
				// 如果不成立，递归的在possible索引处子结点上删除键key
 
				// 递归删除
				BTNode<K, V> t = deleteKey(x.mChildren[possibleIdx], key);
 
				// 检测删除后返回结点的状态，如果不满足键数量>=最低度数-1，则酌情旋转或合并
				if (t.mCurrentKeyNum < BTNode.LOWER_BOUND_KEYNUM) {
					// 不满足键数量>=最低度数-1
 
					// 判断返回结点的兄弟结点的状况，若兄弟结点的键数量>最低度数-1，则旋转（向兄弟结点借键），若兄弟结点的键数量<=最低度数-1，则与兄弟结点合并
					if (BTNode.hasLeftSiblingAtIndex(x, possibleIdx)) {
						if (BTNode.getLeftSiblingAtIndex(x, possibleIdx).mCurrentKeyNum > BTNode.LOWER_BOUND_KEYNUM) {
							leftRotate(x, possibleIdx);
						} else {
							leftMerge(x, possibleIdx);
						}
					} else if (BTNode.hasRightSiblingAtIndex(x, possibleIdx)) {
						if (BTNode.getRightSiblingAtIndex(x, possibleIdx).mCurrentKeyNum > BTNode.LOWER_BOUND_KEYNUM) {
							rightRotate(x, possibleIdx);
						} else {
							rightMerge(x, possibleIdx);
						}
					}
 
					// 判断删完后根节点是否没有键存在，若没有，则将根节点重新赋值
					if (x == mRoot && x.mCurrentKeyNum == 0) {
						x = x.mChildren[0];
					}
				}
			}
		}
		return x;
	}
```

# 本文源代码的Github链接

[BTree-Github-By-Herry](https://github.com/AmazingRi/B-Tree-Another-Implementation-By-Java)

# 原作者的的代码链接

[原作者源代码 - 561.8 KB](https://www.codeproject.com/KB/recipes/1158559/btree-source.zip)

**其中附有实现B-Tree的源代码。**原作者使用NetBeans IDE 8来创建B-Tree项目。但是，如果您使用Eclipse或其他Java IDE，则可以直接将java文件导入您的文件中。主要的java文件是BTIteratorIF.java，BTKeyValue.java，BTNode.java和BTree.java。泛型类型用于B-Tree的键值对，因此任何作为Comparable的子类的键都可以使用。

**其中还包含BTree的测试用例:**

测试利用Java TreeMap进行比较和数据验证。

- *测试用例0：*  手动并随机设置，搜索和删除键。验证键顺序，大小和值的所有项目。
- *测试用例1：*  从1到19添加键并删除其中的一些键。验证键顺序，大小和值的所有项目。
- *测试用例2：*  从1到32添加键并删除其中的一些键。验证键顺序，大小和值的所有项目。
- *测试用例3：*  从1到50添加键并删除其中一些键。验证键顺序，大小和值的所有项目。
- *测试用例4：*  从1到80添加键，从10到30删除。验证键顺序，大小和值的所有项目。
- *测试用例5：*  从0到40000添加键，从-10到100搜索键，从40000到0删除。验证键顺序，大小和值的所有项目。
- *测试用例6：*  将密钥从40000添加到0，从-10到200搜索键，从1到40000删除。验证键顺序，大小和值的所有项目。

如果由于某些原因导致测试失败，它将在该失败点停止，打印出相关信息并退出测试程序。这样我们就可以确切地指出它失败的确切位置。

**其中还包含呈现B-Tree的工具**

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517174519.png)图9： B树渲染工具

原作者语：“您可以通过添加或删除指定的密钥来尝试并使用B-Tree。如果您遇到过这个工具的错误，只需单击**保存**按钮保存您的步骤，然后将保存的文件发送给我。我将用它来重现bug，并希望下次修复它。我认为该工具本身应该是一篇单独的文章。但是，暂时我只是将它堆叠在本文中。”

# 最后的想法

展望：实现迭代全部键值对的方法。