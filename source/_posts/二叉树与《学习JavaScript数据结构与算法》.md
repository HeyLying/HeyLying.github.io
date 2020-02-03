---
title: 二叉树与《学习JavaScript数据结构与算法》
date: 2020-02-03 15:44:25
tags: 算法
---



因为新型冠状病毒，这个春节假期格外漫长。

最近看完了知乎推荐的《学习JavaScript数据结构与算法》。这本书写得比想象中好，我对书背后的其中一条读者评论非常赞同：

>书中的例子写得很好，易于学习和实践。其教学方法也比一般的C/C++图书好得多。

大学那会学算法，教科书用的就是C写的，内容实在绕。相比之下，这本书就简单易懂多了。当然，有一点也不得不说，由于篇幅有限，后面几章内容多但不详尽，确实需要在看完这本书后的进一步巩固。

这本书的第十章是介绍树，让我想起之前有次面试要求手写树相关的算法——不论什么时候，经典算法知识总是那么重要。

结合本书内容，今天来复习树的知识点。另一方面也是提醒自己，总是要学习。

<!-- more -->

![](/images/11.png)

### 定义

树：是一种非顺序数据结构。类似家谱与公司的组织架构。

节点的深度：节点的深度取决于它的祖先节点的数量。

树的高度：树的高度取决于所有节点深度的最大值。



### 二叉树与遍历

```javascript
const Compare = {
  LESS_THAN: -1,
  MORE_THAN: 1
}

const defaultCompare = function(val1, val2) {
  return val1 > val2 ? Compare.MORE_THAN : Compare.LESS_THAN
}

class Node {
  constructor(key) {
    this.key = key // 节点值
    this.left = null // 左节点的引用
    this.right = null // 右节点的引用
  }
}
class BinarySearchTree {
  constructor(compareFn = defaultCompare) {
    this.compareFn = compareFn
    this.root = null
  }

  // 插入
  insert(key) {
    if (this.root === null) {
      this.root = new Node(key)
    } else {
      this.insertNode(this.root, key)
    }
  }

  insertNode(node, key) {
    if (this.compareFn(key, node.key) === Compare.LESS_THAN) {
      if (node.left === null) {
        node.left = new Node(key)
      } else {
        this.insertNode(node.left, key)
      }
    } else {
      if (node.right === null) {
        node.right = new Node(key)
      } else {
        this.insertNode(node.right, key)
      }
    }
  }

  // 中序遍历
  inOrderTraverse(callback) {
    this.inOrderTraverseNode(this.root, callback)
  }

  inOrderTraverseNode(node, callback) {
    if (node != null) {
      this.inOrderTraverseNode(node.left, callback)
      callback(node.key)
      this.inOrderTraverseNode(node.right, callback)
    }
  }

  // 先序遍历
  preOrderTraverse(callback) {
    this.preOrderTraverseNode(this.root, callback)
  }

  preOrderTraverseNode(node, callback) {
    if (node != null) {
      callback(node.key)
      this.inOrderTraverseNode(node.left, callback)
      this.inOrderTraverseNode(node.right, callback)
    }
  }

  // 后序遍历
  postOrderTraverse(callback) {
    this.postOrderTraverseNode(this.root, callback)
  }

  postOrderTraverseNode(node, callback) {
    if (node != null) {
      this.postOrderTraverseNode(node.left, callback)
      this.postOrderTraverseNode(node.right, callback)
      callback(node.key)
    }
  }
}

const tree = new BinarySearchTree()
tree.insert(11)
tree.insert(7)
tree.insert(15)
tree.insert(5)
console.log(tree)

const printNode = (value) => console.log(value)

tree.inOrderTraverse(printNode)
tree.preOrderTraverse(printNode)
tree.postOrderTraverse(printNode)
```



