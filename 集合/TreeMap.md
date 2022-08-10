### TreeMap

- JDK中对TreeMap的实现使用的是**[红黑树](https://ws.wiki.fallingwaterdesignbuild.com/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)**

#### 红黑树[维基百科]

- 红黑树和[AVL树](https://ws.wiki.fallingwaterdesignbuild.com/baike-AVL树)一样都对插入时间、删除时间和查找时间提供了最好可能的最坏情况担保。这不只是使它们在时间敏感的应用，如[实时应用](https://ws.wiki.fallingwaterdesignbuild.com/baike-即時運算)（real time application）中有价值，而且使它们有在提供最坏情况担保的其他数据结构中作为基础模板的价值；例如，在[计算几何](https://ws.wiki.fallingwaterdesignbuild.com/baike-计算几何)中使用的很多数据结构都可以基于红黑树实现。

- 红黑树是[2-3-4树](https://ws.wiki.fallingwaterdesignbuild.com/baike-2-3-4树)的一种等同。换句话说，对于每个2-3-4树，都存在至少一个数据元素是同样次序的红黑树。在2-3-4树上的插入和删除操作也等同于在红黑树中颜色翻转和旋转。这使得2-3-4树成为理解红黑树背后的逻辑的重要工具，这也是很多介绍算法的教科书在红黑树之前介绍2-3-4树的原因，尽管2-3-4树在实践中不经常使用。

- 红黑树相对于AVL树来说，牺牲了部分平衡性以换取插入/删除操作时少量的旋转操作，整体来说性能要优于AVL树。

##### 红黑树的性质

1. 节点是红色或黑色。
2. 根是黑色。
3. 所有叶子都是黑色（叶子是NIL节点）。
4. 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
5. 从任一节点到其每个叶子的所有[简单路径](https://ws.wiki.fallingwaterdesignbuild.com/baike-道路_(图论))都包含相同数目的黑色节点。

#### Question

##### question1 新插入的节点为什么是红色的

- 由于红黑树性质：“某个节点到它的叶子节点上的简单路径的黑色节点数相同”，这意味着如果新插入的节点是**黑色**的，那么为了不破坏这条性质，一定要进行旋转与变色（除了第一个节点）；而新插入的节点为红色，如果它的父节点为黑色，则无需进行调整，简化了插入的复杂性



#### 插入操作

- 新插入的节点一定是作为叶子节点插入的
- 新插入的节点是红色的

##### 情况一：新增节点的父节点是黑色的

- 无需调整

##### 情况二：新增节点的父节点是红色的

- 这个时候违反了红黑树的性质：“每个红色节点必须有两个黑色的子节点。”，需要进行调整

- 调整的思路：

  



#### 删除操作



```java
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

	//p有两个孩子节点，找到p节点的后继节点，将其的值赋给p节点，然后将这个后继节点赋值给p，在这个后继节点上执行真正的删除操作，这个操作与普通二叉树的操作一样
    //由于p的后继节点是p的右子树中最小的节点，所以后继节点的左子树一定为null
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    }

    //如果 p.left != null 为真，那么
    // 1. p.right == null (否则会走上面的代码块，使得p.left == null)
    // 2. 此时p.left.color == red且p.left的两个子节点为null (“任何节点到其所有分枝叶子的简单路径上的黑节点个数相同”)
    //如果 p.left != null 为假，那么
    // 1. p.left == null
    // 2. p.right == null || p.right.color == red
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    //p有且仅有一个孩子节点，且这个节点的颜色为红色，直接用这个孩子节点替代掉p
    if (replacement != null) {
        // Link replacement to parent
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // 如果p节点的颜色为黑色，则需要进行调整
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
       	//只有一个节点的情况，直接删除这个节点
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
        //p节点没有孩子
        
        //p节点的颜色为黑色，进行调整
        if (p.color == BLACK)
            fixAfterDeletion(p);

        //调整完之后将p节点清理掉
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```



- 删除黑色节点后调整颜色与旋转

```java
private void fixAfterDeletion(Entry<K,V> x) {
    //有些情况会进行递归的调整
    while (x != root && colorOf(x) == BLACK) {
        //替换节点是删除节点的左孩子
        if (x == leftOf(parentOf(x))) {
            //取得替换节点的兄弟节点
            Entry<K,V> sib = rightOf(parentOf(x));
            //兄弟节点的颜色是红色，进行左旋
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateLeft(parentOf(x));
                sib = rightOf(parentOf(x));
            }

            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateRight(sib);
                    sib = rightOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        } else { // symmetric
            Entry<K,V> sib = leftOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }

    setColor(x, BLACK);
}
```

