---
title: leetcode-3372-Maximize the Number of Target Nodes After Connecting Trees I
date: 2025-05-31:47:05+08:00
categories: 
- 算法
tags: 
- dfs
---

> There exist two undirected trees with n and m nodes, with distinct labels in ranges [0, n - 1] and [0, m - 1], respectively.
> You are given two 2D integer arrays edges1 and edges2 of lengths n - 1 and m - 1, respectively, where edges1[i] = [ai, bi] indicates that there is an edge between nodes ai and bi in the first tree and edges2[i] = [ui, vi] indicates that there is an edge between nodes ui and vi in the second tree. You are also given an integer k.
> Node u is target to node v if the number of edges on the path from u to v is less than or equal to k. Note that a node is always target to itself.
> Return an array of n integers answer, where answer[i] is the maximum possible number of nodes target to node i of the first tree if you have to connect one node from the first tree to another node in the second tree.
> Note that queries are independent from each other. That is, for every query you will remove the added edge before proceeding to the next query.

<!-- more -->
[题目](https://leetcode.cn/problems/maximize-the-number-of-target-nodes-after-connecting-trees-i/description/)


![2025-05-28T205059](2025-05-28T205059.png)

## 思路
根据题意，``answer``数组中每一项都是edges1中对应下标的元素所邻接最近的k个元素个数 与 edges2 中邻接元素个数最多（k - 1）的和

至于这个邻接元素的个数，可以通过dfs遍历每个点所邻接的边来实现。
由于题目限定了是有效的树，那么我们可以认为edges1和edges2不存在环，所以在dfs的时候，只需要保证children != parent即可保证不会重复递归。
由此有有以下代码


## 代码
```java
    public int[] maxTargetNodes(int[][] edges1, int[][] edges2, int k) {
        int n = edges1.length + 1;
        List<List<Integer>> record = new ArrayList<>();
        for (int x = 0; x < n; x++) {
            record.add(new ArrayList<>());
        }
        int[] sums = new int[n];
        for (int[] edge : edges1) {    
            record.get(edge[0]).add(edge[1]);
            record.get(edge[1]).add(edge[0]);
        }

        for (int x = 0; x < n; x++) {
            int d = dfs(x, -1, record, k);
            sums[x] = d;
        }

        // target
        int m = edges2.length + 1;
        List<List<Integer>> record2 = new ArrayList<>();
        for (int x = 0; x < m; x++) {
            record2.add(new ArrayList<>());
        }
        int[] sums2 = new int[m];
        for (int[] edge : edges2) {    
            record2.get(edge[0]).add(edge[1]);
            record2.get(edge[1]).add(edge[0]);
        }
        int max = 0;
        for (int x = 0; x < m; x++) {
            int d = dfs(x, -1, record2, k - 1);
            sums2[x] = d;
            max = Math.max(max, sums2[x]);
        }
        System.out.println(max);
        for (int x = 0; x < n; x++) {
            sums[x] += max;
        }
        return sums;

    }

    int dfs (int n, int p, List<List<Integer>> record, int k) {
        if (k < 0) {
            return 0;
        }
        int ans = 1;
        List<Integer> children = record.get(n);
        for (int c : children) {
            if (c == p) {
                continue;
            }
            ans += dfs(c, n, record, k - 1);
        }
        return ans;
    }


```