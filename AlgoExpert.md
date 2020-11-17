# AlgoExpert

| #    | Question                   | Category            | Level  | Comments          | Date       |
| ---- | -------------------------- | ------------------- | ------ | ----------------- | ---------- |
| 1    | Two Number Sum             | Arrays              | Easy   |                   | 10/10/2020 |
| 2    | Validate Subsequence       | Arrays              | Easy   |                   | 10/10/2020 |
| 3    | Three Number Sum           | Arrays              | Easy   |                   | 10/10/2020 |
| 4    | Smallest Difference        | Arrays              | Easy   |                   | 10/10/2020 |
| 5    | Move Element To End        | Arrays              | Easy   |                   | 10/10/2020 |
| 6    | Monotonic Array            | Arrays              | Easy   |                   | 10/10/2020 |
| 7    | Spiral Traverse            | Arrays              | Easy   |                   | 10/10/2020 |
| 8    | Max Subset Sum No Adjacent | Dynamic Programming | Medium |                   | 10/10/2020 |
| 9    | Depth-first search         | Graphs              | Easy   |                   | 10/10/2020 |
| 10   | Binary Search              | Searching           | Easy   | 需要看视频        | 10/10/2020 |
| 11   | Bubble Sort                | Sorting             | Easy   | 需要看视频        | 10/10/2020 |
| 12   | Insertion Sort             | Sorting             | Easy   |                   | 10/10/2020 |
| 13   | Selection Sort             | Sorting             | Easy   |                   | 10/10/2020 |
| 14   | Palindrome Check           | Strings             | Easy   | 需要看视频        | 10/10/2020 |
| 15   | Caesar Cipher Encryptor    | Strings             | Easy   | need to see video | 11/16/2020 |
| 16   | Find Three Largest Numbers | Searching           | Easy   | 未解出, 看视频    | 11/16/2020 |
| 17   | Run-Length Encoding        | String              | Easy   | 需要看视频        | 11/16/2020 |
| 18   | Nth Fibonacci              | Recursion           | Easy   | 需要看视频        | 11/16/2020 |
| 19   | Product Sum                | Recursion           | Easy   | 没解出, 看视频    | 11/16/2020 |
| 20   | Branch Sums                | Binary Trees        | Easy   | 没解出, 看视频    | 11/16/2020 |
| 21   | Quick Sort                 | Sorting             | Hard   |                   |            |
| 22   | Three Number Sort          | Sorting             | Medium | 需要看视频        | 11/17/2020 |
| 23   | Node Depths                | Binary Trees        | Easy   |                   |            |
| 24   | Find Closest Value in BST  | Binary Search Trees | Easy   |                   |            |
| 25   | Longest Peak               | Arrays              | Medium |                   |            |
| 26   | Array of Products          | Arrays              | Medium |                   |            |
| 27   | Heap Sort                  | Sorting             | Medium |                   |            |
| 28   | Search In Sorted Matrix    | Searching           | Medium |                   |            |
|      |                            |                     |        |                   |            |
|      |                            |                     |        |                   |            |
|      |                            |                     |        |                   |            |



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