---
title: "从2-3树到红黑树"
date: 2022-03-13T16:50:00+08:00
draft: false
---

在上一篇文章中，我们讨论了一种严格平衡的平衡查找树 AVL 树，本文来看一个在实际场景中运用更多的平衡查找树——红黑树。

本文所解析的红黑树并非一般意义上的更一般化的红黑树，解析的是源自《算法》第四版中由 2-3 树变化而来的左偏红黑树。感兴趣的读者可以在理解了本文所阐释的左偏红黑树后，再去理解更一般的红黑树，相信能够更容易理解。

# 2-3树
2-3 树的思想不仅仅有利于帮助我们理解左偏红黑树，也有利于我们去理解 B 树——一种其变种被广泛用于数据库索引领域的更宽的平衡树。
简单来说，2-3 树就是这样一种树：它的节点可能有两个孩子，也可能有三个孩子。对于两个孩子的节点，其左子节点小于根节点的值，右子节点大于根节点的值。这个与普通二叉搜索树一致，很容易理解。对于三个孩子的节点，其实也很容易理解。如果我们将三个孩子节点中包含的两个值分别称为 A 和 B，那么第一孩子的值全部 小于 A，第三孩子的值全部 大于 B，第二孩子的值介于 A 和 B 之间。大概就是如下图所示：
![Screenshot_2022-03-13-15-23-28-405_com.newskyer.draw.png](/from-two-three-tree-to-rbtree/Screenshot_2022-03-13-15-23-28-405_com.newskyer.draw.png)
但 2-3 树的构建过程与前面的普通二叉搜索树，以及 AVL 树不同，它是由下至上构建的。具体来说，2-3 树的插入规则可以这么理解：

- 如果插入的节点是 2 节点，则生成一个新的键，使其变为 3 节点
- 如果插入的节点是 3 节点，则同样生成一个新的键，使其变为 4 节点，然后再进行调整：
   - 如果这个 4 节点是 2 节点的孩子，那么提取中间的键到父节点，使父节点变成 3 节点
   - 如果这个 4 节点 是 3 节点的孩子，依旧提取中间的键到父节点，使父节点变成 4 节点，递归此过程
   - 如果 4 节点是根节点，则将其分解成两个 2 节点，以及一个 2 节点的根节点

![Screenshot_2022-03-13-16-07-47-053_com.newskyer.draw.png](/from-two-three-tree-to-rbtree/Screenshot_2022-03-13-16-07-47-053_com.newskyer.draw.png)
如果想要了解 2-3 树是如何保证全局的有序性与平衡性的，可以参阅 《算法》第四版 P272-273，本文在此就不赘述。

> 这里再多说一句，B 树的生长过程与 2-3 树非常类似，只是约束条件不同，在下一篇文章讲解 B 树的时候就可以看到。


# 红黑树
我们有了一颗十分平衡的 2-3 树，现在需要将这棵 2-3 树变成红黑树，那么我们只需要将所有的 3 节点拆成两个节点即可。
![Screenshot_2022-03-13-16-10-33-593_com.newskyer.draw.png](/from-two-three-tree-to-rbtree/Screenshot_2022-03-13-16-10-33-593_com.newskyer.draw.png)
可以看出，上图相当于我们对原 2-3 树做这样的一个变换：

- 红色链接：将 2 个 2 节点相连以代替 3 节点
- 黑色链接：其他

由此我们便得到了左偏红黑树的定义：

- 红链接均为左链接
- 没有任何一个节点同时和两条红链接相连
- 该树是完美黑色平衡的。即任意空连接到根的路径上的黑链接数量相同

由于我们只能对节点着色，而除根以外的每个节点必然都有且仅有一个父亲节点，因此我们可以将颜色着色在链接对应的子节点上。

## 旋转（平衡调整）

那么红黑树的再平衡是怎么操作的呢？我们还是需要从 2-3 树的再平衡上去找答案。首先我们需要记住一个规则是：所有新加的节点都是红色节点。原因是，假设我们思考一个 2-3 树，所有添加行为都会让 2 节点变成 3 节点，或者让 3 节点变成临时的 4 节点。因此，新增节点的颜色必然是红色。

然后我们看图：

![微信图片_20220313163654.jpg](/from-two-three-tree-to-rbtree/微信图片_20220313163654.jpg)
![微信图片_20220313163651.jpg](/from-two-three-tree-to-rbtree/微信图片_20220313163651.jpg)
![微信图片_20220313163640.jpg](/from-two-three-tree-to-rbtree/微信图片_20220313163640.jpg)

其中有几个图需要注意一下，一个是等效 3 节点右侧插入一个元素时，这里相当于直接新增一个元素并且做了一个反色。反色操作是相对 AVL 树多出来的一个操作。
# 红黑树实现源码

```java
public class RedBlackBST<Key extends Comparable<Key>, Value> {
    private static final boolean RED = true;
    private static final boolean BLACK = false;
    private Node root;
    
    private class Node {
        Key key;                 
        Value val;               
        Node left, right;     
        int N;                 
        boolean color;      

        public Node(Key key, Value val, int N, boolean color) {
            this.key = key;
            this.val = val;
            this.N = N;
            this.color = color;
        } 
    }

    private boolean isRed(Node x) {
        if (x == null) return false;
        return x.color == RED;
    }
    
    private Node rotateLeft(Node h) {
        Node x = h.right;
        h.right = x.left;
        x.left = h;
        x.color = h.color;
        h.color = RED;
        x.N = h.N;
        h.N = 1 + size(h.left) + size(h.right);
        return x;
    }
    
    private Node rotateRight(Node h) {
        Node x = h.left;
        h.left = x.right;
        x.right = h;
        x.color = h.color;
        h.color = RED;
        x.N = h.N;
        h.N = 1 + size(h.left) + size(h.right);
        return x;
    }
    
    private void flipColors(Node h) {
        h.color = RED;
        h.left.color = BLACK;
        h.right.color = BLACK;
    }

    private int size() {
        return size(root);
    }
    
    private int size(Node h) {
        if (h == null) return 0;
        return h.N;
    }

    public void put(Key key, Value val) {
        root = put(root, key, val);
        root.color = BLACK;
    }

    private Node put(Node h, Key key, Value val) {
        if (h == null) {
            return new Node(key, val, 1, RED);
        }

        int cmp = key.compareTo(h.key);
        if (cmp < 0) {
            h.left = put(h.left, key, val);
        } else if (cmp > 0) {
            h.right = put(h.right, key, val);
        } else {
            h.val = val;
        }
        if (isRed(h.right) && !isRed(h.left)) {
            h = rotateLeft(h);
        }
        if (isRed(h.left) && isRed(h.left.left)) {
            h = rotateRight(h);
        }
        if (isRed(h.left) && isRed(h.right)) {
            flipColors(h);
        }
        h.N = size(h.left) + size(h.right) + 1;
        return h;
    }
}
```



