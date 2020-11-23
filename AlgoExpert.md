### Depth First Search

```python
# Feel free to add new properties
# and methods to the class.
class Node:
    def __init__(self, name):
        self.children = []
        self.name = name

    def addChild(self, name):
        self.children.append(Node(name))
        return self

    def depthFirstSearch(self, array):
        # Write your code here.
        array.append(self.name)
		for child in self.children:
			child.depthFirstSearch(array)
		return array
```

```
graph = A
	/   |  \
   B    C    D
   
```



```python
{
  "graph": {
    "nodes": [
      {"children": ["B", "C", "D"], "id": "A", "value": "A"},
      {"children": ["E", "F"], "id": "B", "value": "B"},
      {"children": [], "id": "C", "value": "C"},
      {"children": ["G", "H"], "id": "D", "value": "D"},
      {"children": [], "id": "E", "value": "E"},
      {"children": ["I", "J"], "id": "F", "value": "F"},
      {"children": ["K"], "id": "G", "value": "G"},
      {"children": [], "id": "H", "value": "H"},
      {"children": [], "id": "I", "value": "I"},
      {"children": [], "id": "J", "value": "J"},
      {"children": [], "id": "K", "value": "K"}
    ],
    "startNode": "A"
  }
}
```



```python

import program
import unittest
class TestProgram(unittest.TestCase):
    def test_case_1(self):
        graph = program.Node("A")
        graph.addChild("B").addChild("C").addChild("D")
        graph.children[0].addChild("E").addChild("F")
        graph.children[2].addChild("G").addChild("H")
        graph.children[0].children[1].addChild("I").addChild("J")
        graph.children[2].children[0].addChild("K")
        self.assertEqual(graph.depthFirstSearch([]), ["A", "B", "E", "F", "I", "J", "C", "D", "G", "K", "H"])

```



# 复杂度 Complexity

There are two dimensions, time and space

空间度





[BFS](https://www.youtube.com/watch?v=E_V71Ejz3f4&t=315s)



# 动态规划

## 动态规划思维

A way to solve the problem by breaking it down into a collection of sub problems. 

### 动态规划和递归的区别

动态规划和递归都需要转移方程, 即上一个状态和现在状态的关系. 

动态规划充分利用了cache, 将计算过的子问题的结果保存起来, 用来计算最终的结果

## 动态规划题目类型

如何判断那种体型使用动态规划

1. 技术
   * 有多少种方法走到右下角
   * 有多少种方式, 选出K个数使得和是sum
2. 求最值
   * 从左上角到右下角路径的最大数字
   * 最长上升子序列长度

3. 求存在性
   * 取石子游戏, 先手是否必胜
   * 能不能选出k个数使得和是sum

   

习题:

有三种硬币, 分别面试2元, 5元和7元, 每种硬币有足够多. 

买一本书需要27元,正好没有零钱, 你需要最少的银币组合付清

动态规划有四个组成部分: 

1. 确定状态
   * 解动态规划的时候需要开一个数组, 数组的每一个元素`f[i]`或者 `f[i][j]`代表什么
   * 确定状态需要两个意识
     * 最后一步: 最优策略的最后一步  (对于最优策略的子问题, 也必须是最优的, 才能向下分解)
     * 子问题: 通过最后一步, 把问题转为子问题, 规模减小
2. 转移方程
3. 初始条件和边界情况
   * 边界调节: 不要数组越界
   * 初始条件: 用转移方程算不出来, 需要手动定义的值

4. 计算顺序: 一般是从小到大, 如果后面的结果, 基于先前计算的结果



# Trees

A Tree consists of a set of nodes (vertex) and a set of edges that connect pairs of nodes. A tree has following properties:

- One node of the tree is designed as the root
- Every node `n`, except the root node, is connected by an edge from exactly one other node p, where p is parent of n
- A unique path traverse from the root to each node

**Branch**

- Any path in the tree that starts from the root node and ends at one of the bottom nodes in the tree

### Complete Tree

- Every single level filled up except the final level which may or may not be filled up, but the final level has nodes, they should be filled up left to right

  ```
  complete binary tree                       not complete binary tree
   		20											 20
   	   /   \                                            /
   	 30     40                                        30
      /   \                                            /
    23     26                                        40
                                                    /
                                                  50  
  ```

  

### Full Tree

- Every node in the tree, either has no children nodes or `k` children nodes

### Perfect Tree

- All of the leaf node has the same depth

### Binary Tree

If each node in the tree has a maximum of two children, we say that the tree is a binary tree

# BST



## 概念和定义

* 每一个节点只有左右两个子节点
* Its value is strictly greater than the values of every node to its left, its value is less than or equal to the values of every node to its right;
* 有定义推断: 最左侧的子节点是整个树最小的节点, 最右侧的子节点是整个树最大的节点

```
              10
             /   \
           5      15
         /   \   /   \
       2      5 13    22
     /            \
    1              14  
```

类定义

```python
class BST:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None
```

插入操作

* 每一次迭代判断是插入左侧子树还是右侧子树, 都能略去一半的子节点. 故平均时间复杂度$O(logN
  )$ 
* 只有遇到当前节点是叶子节点时才做插入操作
* 插入完成后就break循环
* 如果遇到元素相同, 则往右侧子节点继续遍历

```python
def insert(self, value):
   	currentNode = self
    while True:
        if value < currentNode.value:
            if currentNode.left is None:
                currentNode.left = BST(value)
                break
            else:
                currentNode = currentNode.left
        else:
            if currentNode.right is None:
                currentNode.right = BST(value)
                break
            else:
            	currentNode = currentNode.right
            current = current.right
        
            
    return 
```

查找

* 循环终止的条件 1: 当找到目标节点时 break
* 循环终止条件2: 当遍历到叶子节点还是找不到

```python
def contain(self, value):
    currentNode = self
    while currentNode is Not None:
        current 
    
    return False
```

## BST Traverse

```    
tree =          10
             /       \
           5          15
         /    \         \
       2        5        22
     /
    1 
```



### In Order Traverse

* from the most left leaf
* left -> Top -> right

```
[1, 2, 5, 5, 10, 15, 22]
```



### Pre Order Traverse

* from root node
* top -> left -> right

```
[10, 5, 2, 1, 5, 15, 22]
```



### Post Order Traverse

* from the most left leaf
* left -> right -> top

```
[1, 2, 5, 5, 22, 15, 10]
```



# Heap

* Heap is a complete, balanced binary tree

* There are two types of heap trees: Min heap, Max heap

  

## Min Heap 

* The value of parent node is less than or equal to either of its children

  ```
             8
           /   \
         12      23
        /   \   /   \
      17   31 30     44
     /  \
   102   18
  ```

  

* the root node is the smallest value of the heap

* 节点index公式

  ```
  currentNode = i
  childone = 2i + 1
  childtwo = 2i + 2
  
  parentNode = floor((i-1)/2)
  ```

  

```python
# siftup

```



## Max Heap

* Value of parent node is greater than or equal to its children

  ```
  Example of max heap
              55
            /    \
          11     33
         /   \      
        9     8
  ```

* Construction of Max Heap

  1. Insert from the left leaf
  2. during insertion, make the tree complete 

```
[8,5,2,9,5,6,3] --> [9,8,6,5,5,2,3]
```



# Recursion

## Permutation

> Write a function that takes in an array of unique integers and returns an array of all permutations of those integers in no particular order.
>
> If the input array is empty, the function should return an empty array.
>
> Sample Input
>
> ```
> array = [1,2,3]
> ```
>
> Sample Output
>
> ```
> [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
> ```
>
> 

### Python Implementation

```python
def 
```

# Linked List

> Write a `DoublyLinkedList` class that has a `head` and `tail`, both of which point to either a linked list `Node` or `None`/`null`. The class should support:
>
> - setting the head and tail of the linked list.
> - Inserting nodes before and after other nodes as well as at given positions (the position of the head node is `1`)
> - Removing given nodes and removing nodes with given values.
> - Searching for nodes with given values
>
> Note that the `setHead`, `setTail`, `insertBefore`, `insertAfter`, `insertAtPosition` and `remove` methods all take in actual `Node` as input parameters -- not integers (except for `insertAtPosistion`, which also takes in an integer representing the position): this means that you don't need to create any new `Node` in these methods.



### Python Implementation

Doubly Linked List

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.prev = None
        self.after = None
      
class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None
	
    def setHead(self, node):
        self.head = node
       
   	def setTail(self, node):
        self.tail = node
        
        
```



# Sort

## Bubble Sort

### Java Implementation

### Python Implementation

```python
def
```

## Heap Sort

1. Transform the array to the max heap
2. 