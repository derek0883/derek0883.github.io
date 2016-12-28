---
layout: post 
serial: "From brute force to ultimate optimization - Recursion"
title: "Recursion to two pointer I"
categories: Algorithm
comments: true
---

> This example will show you how to use recursion method solve problem, then lead to two pointer solution.

## [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/){:target="_blank"}
Leetcode [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/){:target="_blank"}

Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).

For example,
S = "ADOBECODEBANC"
T = "ABC"
Minimum window is "BANC".

Note:
If there is no such window in S that covers all characters in T, return the empty string "".

If there are multiple such windows, you are guaranteed that there will always be only one unique minimum window in S.

### Video explanation
<iframe width="560" height="315" src="https://www.youtube.com/embed/wTdClP36Bx4" frameborder="0" allowfullscreen></iframe>

## Is string S contian string T
It is very easy to comeup with a brute force solution, Just examine each possible sub-string of S. 
call isSubStr(subS, T) to check if subs contain all the characters in T.

```java
public class MinWindowSubStr {
    private HashMap<Character, Integer> needed;
    private HashMap<Character, Integer> found;
    private int numOfChar;

    private HashMap<Character, Integer> buildNeedMap(String s, int L, int R) {
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();
        for (int i=0; i<s.length(); i++) {
            char c = s.charAt(i);    
            if (map.containsKey(c)){
                map.put(c, map.get(c)+1);
            } else {
                map.put(c,1);  
            }
        }
        return map;
    }

    private int addChar(char c) {
        if (needed.containsKey(c)) {
            if (!found.containsKey(c)) {
                found.put(c, 1);
                this.numOfChar++;
            } else {
                if (found.get(c) < needed.get(c))
                    this.numOfChar++;
                found.put(c, found.get(c)+1);
            }
        }
        return numOfChar;
    }


    private Boolean isSubStr(String s, int L, int R, String t) {
        found.clear();
        int len = 0;
        for (int i = L; i <= R; i++) {
            if (addChar(s.charAt(i)) == t.length())
                return true;
        }
        return false;
    }

    public String minWindow(String s, String t) {
        if(t.length()>s.length() || t.length()==0) 
            return "";
        needed = buildMap(t, 0, t.length()-1);
        found = new HashMap<Character, Integer>();

        /* 
         * for each possible sub-string of S
         * call isSubStr to check
         * return minimal sub-string of S
         */
    }
}
```

## Reduce unnecessary computation
Every optimal solution comes from brute force, find out unnecessary computation, then eliminate it.
Here in isSubStr we rebuild HashMap for sub string each time. If you take a look at the recursion tree,
when we traval from one node to another, it just remove 1 character from S, and add another into S.
Now we can re-use previous result, each time we add/remove 1 character from HashMap, then return the length,
If the length >= length of t, it means current sub string contain all the characters in T.

You should watch above video explanation if you don't understand.

```java
private int addChar(char c) {
    if (needed.containsKey(c)) {
        if (!found.containsKey(c)) {
            found.put(c, 1);
            this.numOfChar++;
        } else {
            if (found.get(c) < needed.get(c))
                this.numOfChar++;
            found.put(c, found.get(c)+1);
        }
    }
    return numOfChar;
}
private int delChar(char c) {
    if (!needed.containsKey(c))
        return this.numOfChar;
    if (!found.containsKey(c)) // should not happen
        return this.numOfChar;
    found.put(c, found.get(c)-1);
    if (found.get(c) < needed.get(c))
        this.numOfChar--;
    return this.numOfChar;
}
```

## DFS + memorize O(n^2) time complexity solution
Now we are ready for a DFS Top Down solution, 
here we are looking for minimal length. when we found subS contian T,
it may not the minimal length, we need continue examine each sub tree of S, 
only if all sub tree return empty, then we can return subS.

You should watch above video explanation if you don't understand.

This introduced a extra complexity to this problem. that's why BFS is better solution.
But as a practice, it is still worth to pratice DFS, it will improve your skill to use DFS solve problem.

You should watch my previous post if you DFS + memorizing if new to you. e.g.
1. [Recursion to 1 dimensional dynamic programming I]({% post_url 2016-11-20-1d-dynamic1 %})
2. [Recursion to 1 dimensional dynamic programing II]({% post_url 2016-11-20-1d-dynamic2 %})
3. [Recursion to 1 dimensional dynamic programing III]({% post_url 2016-11-20-1d-dynamic3 %})


```java
public class MinWindowSubStr {
    private HashMap<Character, Integer> needed;
    private HashMap<Character, Integer> found;
    private int numOfChar;

    class Node {
        public int l, r;
        public Node(int l, int r) { this.l = l; this.r = r; }
        public Node(int l, int r, int f) { this.l = l; this.r = r;}
        public Boolean isEmpty() { return this.l == -1 && this.r == -1; }
        public int size() { return this.r - this.l + 1; }
    }
    public String minWindow(String s, String t) {
        if(t.length()>s.length() || t.length()==0) 
            return "";
        needed = buildNeedMap(t, 0, t.length()-1);
        found = new HashMap<Character, Integer>();
        return dfsCache(s, t);
    }
    private String dfsCache(String s, String t) {
        int n = s.length()+1;
        Node[][] cache = new Node[n][n];
        for (int i = 0; i < s.length(); i++) 
            addChar(s.charAt(i));
        Node node = dfsCache(s, 0, s.length()-1, t, cache);
        if (node.isEmpty())
            return "";
        return s.substring(node.l, node.r+1);
    }

    Node dfsCache(String s, int L, int R, String t, Node[][] cache) {
        if (R-L+1 < t.length() || R < L) 
            return new Node(-1, -1); 
        if (this.numOfChar < t.length())
            return new Node(-1, -1); 
        if (cache[L][R] != null)
            return cache[L][R];

        delChar(s.charAt(L));
        Node n1 = dfsCache2(s, L+1, R, t, cache);
        addChar(s.charAt(L));

        delChar(s.charAt(R));
        Node n2 = dfsCache2(s, L, R-1, t, cache);
        addChar(s.charAt(R));

        Node node;
        if (n1.isEmpty() && n2.isEmpty()) {
            node = new Node(L, R);
        } else if (n1.isEmpty()) {
            node = n2;
        } else if (n2.isEmpty()) {
            node = n1;
        } else {
            node = n1.size() < n2.size() ? n1 : n2;
        }
        cache[L][R] = node;
        return node;
    }
}
```

## BFS O(n^2) time complexity solution
Keep in mind whenever DFS Top Down is inconvenience, you should examine BFS Bottom UP.
The following code is a bottom up BFS zigzag traverse solution.
You should watch above video explanation if you don't understand.

```java
private String bottomUpBfs(String s, String t) {
    int len = t.length();
    needed = buildNeedMap(t, 0, t.length()-1);
    int L = 0;
    int R = 0;
    int shift = 1;
    for (int i = 0; i < len-1; i++)
        addChar(s.charAt(i));
    while (len <= s.length()) {
        if (shift == 1) {
            L = 0;
            R = L + len -1;
            addChar(s.charAt(R));
        } else {
            R = s.length()-1;
            L = R-len+1;
            addChar(s.charAt(L));
        }
        do {
            if (this.numOfChar == t.length())
                return s.substring(L, R+1);
            if (shift == 1 && R+1 < s.length()) {
                delChar(s.charAt(L));
                addChar(s.charAt(R+1));
            } else if (shift == -1 && L-1 >= 0) {
                delChar(s.charAt(R));
                addChar(s.charAt(L-1));
            }
            R += shift;
            L += shift;
        } while ((shift == 1 && R < s.length()) || (shift == -1 && L >= 0));
        shift = (shift == 1 ? -1 : 1);
        len++;
    }

    if (this.numOfChar == t.length())
        return s.substring(L, R+1);
    return "";
}
```

## Two pointer O(n) time complexity solution
Two pointer solution is a eager version of bottom up BFS.
In two pointer solution, we never go back, we start with L=0, R=0. 
in the worst case, R reached the length of s, and L reached the length of s.
that 2*n, ignore leading constant, time complexity is O(n). Passed leetcode OJ, Runtime: 49 ms

You should watch above video explanation if you don't understand the following code, it is pretty easy.


```java
private String twoPointer(String s, String t) {
    int len = t.length();
    needed = buildNeedMap(t, 0, t.length()-1);
    int L = 0;
    int R = 0;
    String r = "";
    int minLen = s.length();
    while (R < s.length()) {
        while (R < s.length() && addChar(s.charAt(R)) < t.length())
            R++;
        while (delChar(s.charAt(L)) >= t.length())
            L++;
        if (R-L+1 <= minLen) {
            minLen = R-L+1;
            r = s.substring(L, R+1);
        }
        L++;
        R++;
    }
    return r;
}
```
