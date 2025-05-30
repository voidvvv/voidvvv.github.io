---
title: leetcode_2359_Find Closest Node to Given Two Nodes
date: 2025-05-30T21:30:39+08:00
tags:
---

You are given a directed graph of n nodes numbered from 0 to n - 1, where each node has at most one outgoing edge.

The graph is represented with a given 0-indexed array edges of size n, indicating that there is a directed edge from node i to node edges[i]. If there is no outgoing edge from i, then edges[i] == -1.

You are also given two integers node1 and node2.

Return the index of the node that can be reached from both node1 and node2, such that the maximum between the distance from node1 to that node, and from node2 to that node is minimized. If there are multiple answers, return the node with the smallest index, and if no possible answer exists, return -1.

Note that edges may contain cycles.

<!--more-->

[原题](https://leetcode.cn/problems/find-closest-node-to-given-two-nodes/description/)

![2025-05-30T215336](2025-05-30T215336.png)

## 思路
根据题意，我们需要在有向图中，找出两个指定node所共同能到达的点，并且该点到达两个node的距离尽可能短。
那么我们就可以分别列出两个node所能到达的所有点。有点类似与寻路算法(AStar)，但是会更简单，因为此题中每个点只会指向一个另外的点，即每个点只会延伸出一条connection，所以不需要进行dfs或者bfs，直接链式的向下遍历即可。
需要注意，题目明确提出graph会出现环(cycle)，所以我们需要注意处理，处理的方法也简单，只需要记录到达当前点所花费的步数即可。。遍历到某一node时，判断该步数是否为初始值(-1)，若是初始值，则表明之前没有处理过，需要处理该点。但如果该点的step不为初始值，那么就需要判断是否继续处理了。

## 代码
```java
class Solution {
    public int closestMeetingNode(int[] edges, int node1, int node2) {
        int n = edges.length;

        int[] node1Record = new int[n]; // 标记node1所能访问的点
        Arrays.fill(node1Record, -1);

        int[] node2Record = new int[n]; // node2 所能访问的点
        Arrays.fill(node2Record, -1);

        addRecord(node1Record, edges, node1, 0);
        addRecord(node2Record, edges, node2, 0);
        // System.out.println(Arrays.toString(edges));

        // System.out.println(Arrays.toString(node1Record));
        // System.out.println(Arrays.toString(node2Record));
        int min = -1;
        int node = -1;
        for (int i = 0; i < n; i++) {
            if (node1Record[i] >=0 && node2Record[i] >= 0) {
                if (min < 0) {
                    min = Math.max(node1Record[i], node2Record[i]);
                    node = i;
                } else {
                    if (min > Math.max(node1Record[i], node2Record[i])) {
                        min = Math.max(node1Record[i], node2Record[i]);
                        node = i;
                    }
                }
            }
        }
        // System.out.println(node + " _   " + min);
        return node;

    }

    void addRecord(int[] record, int[] edges, int node, int step) {

        if (record[node] > step || record[node] < 0) {
            record[node] = step;

            if (edges[node] == -1) {
                return;
            }
            addRecord(record, edges, edges[node], step + 1);

        }

    }
}

```