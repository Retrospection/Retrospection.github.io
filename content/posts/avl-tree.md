---
title: "AVL树"
date: 2022-02-20T21:01:08+08:00
draft: false
---

题记：AVL树是笔者在上学时就一直没弄懂的数据结构，时隔多年重学数据结构，终于跨越了新手误区，成功理解了它的旋转，特作此文以记录一下。

> 注：本文默认读者已对二叉查找树、AVL树的基本功能有了一个感性认识，因此不赘述其查询过程，也不分析其查询复杂度，仅就其理解上的难点给出说明。

## 一、正确理解递归

树是一个典型的递归定义的数据结构，在树上执行的操作很多都是递归进行的，因此要理解树上的操作，我们首先需要正确理解递归。

笔者在第一次学习递归时，一度试图跟着递归调用一直走到递归终止，然后跟着递归弹栈，以理解全过程。这种策略在理解迭代型程序是可行的，但是理解递归程序切不可这么做。试图跟着递归程序理解其递归全过程，是新手学习递归过程中最容易进入的误区。

那么应该如何正确理解递归呢？我们可以考虑高中数学里一个常见的递归（递推）结构：数列。当我们描述一个数列时，有时我们可以给定一个数列的通项公式，但更多时候，通项公式并没有数列的递推公式容易得到。所以，也时常有数列是以`递推公式+首项`的方式进行描述的，这其实就是理解递归的正确方式。

> 在理解递归时，只需要考虑相邻结构，以及递归终止条件（即相当于数列首项）即可，数学结构保证了其有效性。

## 二、二叉查找树

> 二叉查找树是指一棵空树或者具有下列性质的二叉树：
> 1. 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
> 2. 若任意节点的右子树不空，则右子树上所有节点的值均大于它的根节点的值；
> 3. 任意节点的左、右子树也分别为二叉查找树；

上面是二叉查找树的正式的定义，其实一般情况下，只需要记住二叉查找树的左子树的值都小于根，右子树的值都大于根，就已足够。

二叉查找树的构建是通过一个值一个值插入来构建的，哪怕有一组用来构建二叉查找树的值，二叉查找树也只能通过逐个插入来构建。

举例：1、4、3、5、6、2、7 这个序列：
![二叉查找树插入示例](/avl-tree-image/1.jpg)

但同样的7个数字，如果不是按照上述插入方式插入的话，得到的二叉查找树的结构就会大相径庭。比如一个很容易发现的规律是：序列中第一个插入的元素一定会成为朴素二叉查找树的根节点。

## 三、AVL树

由于朴素二叉查找树在极端情况下（即插入序列为有序序列）构建的查找树退化成一个单链表，使得在二叉查找树上进行查询的时间复杂度退化成O(n)。因此，为了保证查询高效，我们必须在插入元素后，对树的结构进行调整，使树尽量「宽」，而非尽量「深」，使得树能够保证平均查询复杂度始终保持在O(logN)数量级。

为了衡量一棵树是否需要调整，我们需要有一个方式对树的状态进行度量。在AVL树中，我们选择的度量方式是比较子树的高度。

> - 路径长度：从根节点到叶子节点经过的节点数量（包含根节点和叶子结点）
> - 树的高度：一棵树的所有路径里最长路径的长度

在AVL树结构中，当一棵树的左子树与右子树的高度差的绝对值超过2时，就需要对树的结构进行调整了，这种调整在一般的文章讨论中成为「旋转」，其实本质就是对树中一些指针的指向进行调整的过程，它和链表插入也没什么本质的区别，都是指针调整而已。

那么 AVL 树是怎么做的呢？

首先 AVL 树将需要调整的情况分成了四类：

1. 在左子树的左子树上做插入
2. 在左子树的右子树上做插入
3. 在右子树的左子树上做插入
4. 在右子树的右子树上做插入

其中类型 1、4 镜像对称，类型 2、3 镜像对称。因此我们只需要研究类型 1 和类型 2 即可。

> 这里额外插一句，从上文对情况分类讨论时的用词「左子树的左子树」「右子树的左子树」等，我们可以很清晰地发现关于 AVL 结构调整的操作是在两层（即子树，以及子树的子树）上去操作的，这一定程度上有别于一些简单的递归操作时仅仅只在一层上去操作，此处需要格外注意去理解。

> 另外，关于如何找到要调整的根节点，这块看看具体的代码实现就清晰了，文字描述反而容易增加理解难度，因此此处不用文字赘述。

### 1、4两种情况的处理

关于这种调整是如何进行的，也话不多说，看图即可。

![左旋转](/avl-tree-image/2.jpg)

上图描述了如何处理 `在左子树的左子树上做插入`， 即第一类情况需要做出的调整。
用文字描述的话就是：让根节点成为其左子树的右子树，然后让左子树原来的右子树成为根节点新的左子树。经过调整后，原左子树变成了根节点。

相信上面这段描述结合图示应该足够清晰。

将上面的这个调整过程写成代码的话：
```c++
void rotateWithLeftChild (AVLNode** node)
{
    // 先标记一下左子树为t
    auto t = (*node)->left;
    // 让左子树的右子树成为根节点的左子树
    (*node)->left = t->right;
    // 让根节点成为左子树的右子树
    t->right = *node;

    // 在调整完后，重新计算一下各节点的高度
    (*node)->height = std::max(height((*node)->left), height((*node)->right)) + 1;
    t->height = std::max(height(t->left), (*node)->height) + 1;

    // 重新标记一下根节点
    *node = t;
}
```

第四种情况的处理完全是镜面对称的：
```c++
void rotateWithRightChild (AVLNode** node)
{
    // 先标记一下右子树为t
    auto t = (*node)->right;
    // 让右子树的左子树成为根节点的右子树
    (*node)->right = t->left;
    // 让根节点成为右子树的左子树
    t->left = *node;

    // 在调整完后，重新计算一下各节点的高度
    (*node)->height = std::max(height((*node)->left), height((*node)->right)) + 1;
    t->height = std::max(height(t->right), (*node)->height) + 1;
    
    // 重新标记一下根节点
    *node = t;
}
```

至此，我们就有了处理2、3两种情况的武器

### 2、3两种情况的处理

话不多说，还是先看图吧，我们依旧以在`左子树的右子树插入`为例来看

![先右后左旋转](/avl-tree-image/3.jpg)

从图上我们可以发现，情况2其实可以分解成两步：
1. 处理在左子树上的情况4
2. 处理在根节点上的情况1

由此，代码也可以这么写了
```c++
void doubleRotateWithLeftChild (AVLNode** node)
{
    // 处理左子树上的情况4
    rotateWithRightChild(&((*node)->left));
    // 处理根节点上的情况1
    rotateWithLeftChild(node);
}
```

同理，情况3也是情况2的镜像对称：
```c++
void doubleRotateWithRightChild (AVLNode** node)
{
    // 处理右子树上的情况1
    rotateWithLeftChild(&((*node)->right));
    // 处理根节点上的情况4
    rotateWithRightChild(node);
}
```

由此我们便搞定了AVL树中最艰难的树高再平衡过程，或者叫做「旋转」过程。


## 四、完整代码

```c++
struct AVLNode
{
  int data;
  int height;
  AVLNode* left;
  AVLNode* right;
};

class AVLTree
{
public:
  AVLTree()
    : m_pRoot(nullptr)
  {
    
  }
  
  ~AVLTree()
  {
    dispose(m_pRoot);
  }
  
  bool contains(int value)
  {
    return contains(value, m_pRoot);
  }
  
  void insert(int value)
  {
    insert(value, &m_pRoot);
  }
  
  int height(AVLNode* t) const
  {
    return t == nullptr ? -1 : t->height;
  }
  
  void print()
  {
    print(m_pRoot);
  }
  
  
private:
  
  void dispose(AVLNode* node)
  {
    
    if (node->left != nullptr)
    {
      dispose(node->left);
    }
    if (node->right != nullptr)
    {
      dispose(node->right);
    }
    delete node;
  }
  
  bool contains(int value, AVLNode* node)
  {
    if (node == nullptr)
    {
      return false;
    }
    
    if (value == node->data)
    {
      return true;
    }
    else if (value < node->data)
    {
      return contains(value, node->left);
    }
    else
    {
      return contains(value, node->right);
    }
  }
  
  void insert(int value, AVLNode** node) {
    if (*node == nullptr) {
      *node = new AVLNode{ value, 0, nullptr, nullptr };
    }
    else if (value < (*node)->data) {
      insert(value, &((*node)->left));
      if (height((*node)->left) - height((*node)->right) == 2) {
        if (value < (*node)->left->data) {
          rotateWithLeftChild(node);
        }
        else {
          doubleRotateWithLeftChild(node);
        }
      }
    }
    else if ((*node)->data < value) {
      insert(value, &((*node)->right));
      if (height((*node)->right) - height((*node)->left) == 2) {
        if (value > (*node)->right->data) {
          rotateWithRightChild(node);
        }
        else {
          doubleRotateWithRightChild(node);
        }
      }
    }
    
    (*node)->height = std::max(height((*node)->left), height((*node)->right)) + 1;
  }
  
  void rotateWithLeftChild (AVLNode** node)
  {
    auto t = (*node)->left;
    (*node)->left = t->right;
    t->right = *node;
    
    (*node)->height = std::max(height((*node)->left), height((*node)->right)) + 1;
    t->height = std::max(height(t->left), (*node)->height) + 1;
    
    *node = t;
  }
  
  void doubleRotateWithLeftChild (AVLNode** node)
  {
    rotateWithRightChild(&((*node)->left));
    rotateWithLeftChild(node);
  }
  
  void rotateWithRightChild (AVLNode** node)
  {
    auto t = (*node)->right;
    (*node)->right = t->left;
    t->left = *node;
    
    (*node)->height = std::max(height((*node)->left), height((*node)->right)) + 1;
    t->height = std::max(height(t->right), (*node)->height) + 1;
    *node = t;
  }
  
  void doubleRotateWithRightChild (AVLNode** node)
  {
    rotateWithLeftChild(&((*node)->right));
    rotateWithRightChild(node);
  }
  
  
  
  void print(AVLNode* node)
  {
    if (node->left != nullptr)
    {
      print(node->left);
    }
  
    std::cout << node->data << " ";
  
    if (node->right != nullptr)
    {
      print(node->right);
    }
  }
  
  
  
  AVLNode* m_pRoot;
};

```





