---
layout: post 
serial: "From brute force to ultimate optimization - Recursion"
title: "Recursion tree and recursion tree traversal II"
categories: algorithm
comments: true
---

> This article will show you how to using recursion tree to analysis problem, and how to traverse through the tree to get all result we needed. 
Before you using recursion tree to solve practical problem, you have to learn basic graph concept. Here is very good material for you. 
[http://algs4.cs.princeton.edu/40graphs/]()

## [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/){:target="_blank"}
Leetcode [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/){:target="_blank"}

Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.
For example, given n = 3, a solution set is:

```
[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

### Video explanation

<iframe width="560" height="315" src="https://www.youtube.com/embed/JKXs7a4RMFU" frameborder="0" allowfullscreen></iframe>

As you can see, each leaf node is the result we need. So we just travel through the tree using Depth First Search(DFS) or Breadth First Search(BFS). 
When you draw the tree step by step, pay attention to the conditions.
1. What the termination condition at the leaf of the tree.
2. What is the expansion condition which allows you to draw one or more node from a parent node.

### DFS solution
```java
public class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> all;
        all = recursiveDfs(n);
        return all;
    }
    public List<String> recursiveDfs(int n) {
      List<String> all = new ArrayList<String>();
      dfs(all, "", 0, 0, n);
      return all;
    }
    public void dfs(List<String> all, String one, int L, int R, int n) {
        // Termination condition, when both L and R equals n
        // meaning reached leaf node
        if (L == n && R == n) { 
            all.add(one);
            return;
        }
        // Condition 1: draw a child node from parent 
        // by adding "(" to parent node
        if (L < n) 
            dfs(all, one+"(", L+1, R, n); 
        // Condition 2: draw a child node from same parent 
        // by adding ")" to parent node
        if (R < L) 
            dfs(all, one+")", L, R+1, n); 
    }
}
```

Wow, Don't you feel how easy it is! Let take a look at BFS version. BFS requires an auxiliary data structure to keep tracking each node.

### BFS solution

```java
class Node {
    public String str;
    public int l, r;
    public Node(String s, int l, int r) {
        this.l = l;
        this.r = r;
        this.str = s;
    }
}
public List<String> bfs(int n) {
    List<String> all = new ArrayList<String>();
    Queue<Node> q = new LinkedList<>();
    q.offer(new Node("", 0, 0));
    while (!q.isEmpty()) {
        Node cur = q.poll();
        // Termination condition, when both L and R equals n
        // meaning reached leaf node
        if (cur.l == n && cur.r == n) {
            all.add(cur.str);
            continue;
        }
        // Condition 1: draw a child node from parent 
        // by adding "(" to parent node
        if (cur.l < n)
            q.offer(new Node(cur.str+"(", cur.l+1, cur.r));
        // Condition 2: draw a child node from same parent 
        // by adding ")" to parent node
        if (cur.r < cur.l)
            q.offer(new Node(cur.str+")", cur.l, cur.r+1));
    }
    return all;
}
```
Have you noticed the symmetric between DFS and BFS. Both termination condition and expansion condition are exactly same. Here is another non recursive DFS version. 

### Non-recursive DFS solution

```java
public List<String> nonRecursiveDfs(int n) {
    List<String> all = new ArrayList<String>();
    Stack<Node> st = new Stack<Node>();
    st.push(new Node("", 0, 0));
    while (!st.isEmpty()) {
        Node cur = st.pop();
        if (cur.l == n && cur.r == n) {
            all.add(cur.str);
            continue;
        }
        if (cur.l < n)
            st.push(new Node(cur.str+"(", cur.l+1, cur.r));
        if (cur.r < cur.l)
            st.push(new Node(cur.str+")", cur.l, cur.r+1));
    }
    return all;
}
```

Again the beauty of symmetric. 

You should be able to implement 3 version easily, if you don't, then keep practicing. 

Quiz: 
For same input, 3 implementation will give right answer, are the output of each version exactly same? 
If your answer is no, explain why, and can you change the code to make all version give exactly same result? 

To answer those question, don't compile and run the code. You should be able to answer By using visualized recursion tree and recursion analysis concept.
