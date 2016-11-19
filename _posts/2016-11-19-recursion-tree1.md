---
layout: post 
serial: "From brute force to ultimate optimization - Recursion"
title: "Recursion tree and recursion tree traversal I"
categories: algorithm
toc: true
comments: true
---

> This article will show you how to using recursion tree to analysis problem, and how to traverse through the tree to get all result we needed. 
Before you using recursion tree to solve practical problem, you have to learn basic graph concept. Here is very good material for you. 
[http://algs4.cs.princeton.edu/40graphs/]()

## [Subsets](https://leetcode.com/problems/subsets/){:target="_blank"}
Leetcode [Subsets](https://leetcode.com/problems/subsets/){:target="_blank"}

Given a set of distinct integers, nums, return all possible subsets.
Note: The solution set must not contain duplicate subsets.
For example,
If nums = [1,2,3], a solution is:

```
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

The following video show you how to draw the recursion tree step by step.

<iframe width="560" height="315" src="https://www.youtube.com/embed/rxitBSy8pZ0" frameborder="0" allowfullscreen></iframe>


### DFS solution

```java
private List<List<Integer>> dfs(int[] nums) {
    List<List<Integer>> all = new ArrayList<>();
    List<Integer> one = new ArrayList<Integer>();
    dfs(nums, 0, one, all);
    return all;
}
private void dfs(int[] nums, int pos, List<Integer> one, List<List<Integer>> all) {
    if (pos == nums.length) {
        all.add(new ArrayList<Integer>(one));
        return;
    }
    dfs(nums, pos+1, one, all); 
    one.add(nums[pos]);
    dfs(nums, pos+1, one, all); 
    one.remove(one.size()-1);
}
```

### BFS solution
Again BFS requires an auxiliary data structure to keep tracking each node.

```java
class Node {
    ArrayList<Integer> one;
    int idx;
    Node() {one = new ArrayList<Integer>();}
    Node(int i) { idx = i; }
}
private List<List<Integer>> bfs(int[] nums) {
    Queue<Node> q = new LinkedList<>();
    List<List<Integer>> all = new ArrayList<>();
    q.offer(new Node());
    while (!q.isEmpty()) {
        Node n = q.poll();
        if (n.idx == nums.length) {
            all.add(n.one);
        } else {
            Node n1 = new Node(n.idx+1);
            n1.one = new ArrayList<Integer>(n.one);
            n1.one.add(nums[n.idx]);
            // reuse this Node
            n.idx++;
            q.offer(n);
            q.offer(n1);
        }
    }
    return all;
}
```

### Non-recursive DFS solution

```java
private List<List<Integer>> nonRecursiveDfs(int[] nums) {
    Stack<Node> st = new Stack<Node>();
    List<List<Integer>> all = new ArrayList<>();
    st.push(new Node());
    while (!st.isEmpty()) {
        Node n = st.pop();
        if (n.idx == nums.length) {
            all.add(n.one);
        } else {
            Node n1 = new Node(n.idx+1);
            n1.one = new ArrayList<Integer>(n.one);
            n1.one.add(nums[n.idx]);
            n.idx++;
            st.push(n);
            st.push(n1);
        }
    }
    return all;
}
```
