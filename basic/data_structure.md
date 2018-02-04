---
title: 数据结构和算法
date: 2016-10-23 11:05:49
tags: [basic]
---

# 冒泡排序（bubble sort）

```
do
  swapped = false
  for i = 1 to indexOfLastUnsortedElement
    if leftElement > rightElement
      swap(leftElement, rightElement)
      swapped = true; swapCounter++
while swapped
```

# 二叉堆（binary heap）

[演示](https://visualgo.net/heap)

create，N

给定树，构建成二叉堆，

```
for (i = A.length/2; i >= 1; i--)
  shiftDown(i)
```

insert，logN

插入一个元素，循环将这个元素和父元素比较，然后上虑（percolate up）

```
A[A.length] = v
i = A.length-1
while (i > 1 && A[parent(i)] < A[i])
  swap(A[i], A[parent(i)])
```

extract max，logN

弹出根元素（假设是最大值），将最后一个元素放到根位置，然后这个元素和最大的子元素比较

```
take out A[1]
A[1] = A[A.length-1]
i = 1; A.length--
while (i < A.length)
  if A[i] < (L = the larger of i's children)
    swap(A[i], L)
```

# 查找树

存放树元素的结构体

```
struct node
{
    int element;
    node *right;
    node *left;
};
```

插入元素

```
node *insert(node *tree, int val)
{
    if(tree == nullptr)
    {
        tree = new node;
        tree -> element = val;
        tree -> right = nullptr;
        tree -> left = nullptr;
    }
    else if(val > tree -> element)
        tree -> right insert(tree -> right, val);
    else if(val < tree -> element)
        tree -> left insert(tree -> left, val);
}
```

搜索元素，返回节点指针

```
node *searchVal(node *tree, int val)
{
    if(tree == nullptr)
        return nullptr;
    if(val > tree -> element)
        return searchVal(tree -> right, val);
    else if(val < tree -> element)
        return searchVal(tree -> left, val);
    else
        return tree;
}
```

查找最小值，返回节点指针

```
node *findMin(node *tree)
{
    if(tree == nullptr)
        return nullptr;
    else if(tree -> left == nullptr)
        return tree;
    else
        return findMin(tree -> left);
}
```

删除元素

```
node *deleteVal(node *tree, int val)
{
    if(tree == nullptr)
        return nullptr;
    else if(val > tree -> element)
        tree -> right deleteVal(tree -> right, val);
    else if(val < tree -> element)
        tree -> left deleteVal(tree -> left, val);
    else if(tree -> left != nullptr && tree -> right != nullptr)
    {
        node *tmpnode = findMin(tree -> right);
        tree -> element = tmpnode -> element;
        tree -> right deleteVal(tree -> right, tmpnode -> element);
    }
    else 
    {
        node *tmpnode = tree;
        if(tree -> left == nullptr)
            tree = tree -> right;
        else if(tree -> right == nullptr)
            tree = tree -> left;
        delete tmpnode;
    }
}
```

# AVL树

[演示](https://visualgo.net/bst)

AVL（Adelson-Velskill Landis）树是带有平衡条件的二叉查找树

findMax，logN

递归查找，同理，最小值就是一直往左

```
if this is null return empty
if right != null
  go right
else return this key
```

search，logN

递归查找

```
if this == null
  return null
else if this key == search value
  return this
else if this key < search value
  search right
else search left
```

insert，插入元素

插入元素之后需要检查是否平衡，如果左右子树高度相差2则不平衡

```
insert v
check balance factor of this and its children
  case1: this.rotateRight
  case2: this.left.rotateLeft, this.rotateRight
  case3: this.rotateLeft
  case4: this.right.rotateRight, this.rotateLeft
  this is balanced
```
