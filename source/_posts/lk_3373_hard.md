---
title: leetcode_3373_Maximize the Number of Target Nodes After Connecting Trees II
date: 2025-05-29T21:54:27+08:00
categories: 
- 算法
tags: 
- dfs
---

![2025-05-29T220509](2025-05-29T220509.png)

<!-- more -->

[题目](https://leetcode.cn/problems/maximize-the-number-of-target-nodes-after-connecting-trees-ii/description/)

> 并没有想出完美的解答方案，看了官方题解之后才会做的。之前陷入了昨天那道题之中，采用了dfs + 计数的方法，导致超时。看了官方题解使用的分组（分色）法，大悟

## 思路

{%note info%}
主要思路就是题目规定了给出的edges符合合法的树。那么就必然不会有环。并且其实无向的，所以任意节点开始向外遍历，均会遍历整个树，并且无论从哪个节点开始遍历，最后的结果都是等价的。
并且题目中要求，每个节点的target要求，是要与当前节点相邻偶数位个节点。那么可以想象一下，有一系列几点与当前节点相邻偶数位算作当前节点的target，那么这些节点彼此之间也会互相作为target。反之，有一系列节点与当前节点相邻奇数位，那那么这些节点不会成为当前节点的target，并且**这些节点之间均会形成target**
有了这几个前置条件，我们就已经捋顺了整个逻辑了。我们可以把所有节点遍历，分为两组，每组之间互相为target。
至于edges2，几乎是同理，不过edges1与其相关联时，可以选取其两组中较大的那个作为target数量。

{%endnote%}


## 代码
```java
class Solution {
    public int[] maxTargetNodes(int[][] edges1, int[][] edges2) {
        // 合法的树形结构决定了其不会有环，并且从任意节点出发，遍历其相邻节点，均可达到每一个节点
        // 换句话说，任意节点开始遍历树，结果都是等价的
        // 本题要求的是偶数个数节点记作target（包括本身0），那么对于任何一个节点来说，是否以其作为target就两种情况：
        // 1. 与其相邻偶数位，可以算作该节点的target。并且这些节点互相均以彼此为target
        // 2. 与其相邻奇数位，不算做该节点的target。。并且这些节点互相均已彼此为target
        // 那么我们就可以把树中的所有节点划分为两类，用颜色（color）来标记
        int n = edges1.length + 1;
        int m = edges2.length + 1;
        boolean[] color1 = new boolean[n];
        boolean[] color2 = new boolean[m];
        // 我们需要将所有节点分别划分到不同color中
        int[] nums = divide(edges1, color1);
        int[] nums2 = divide(edges2, color2);

        int[] result = new int[n];
        for (int i = 0; i < n; i++) {
            result[i] = nums[((color1[i])? 0 :1)] + Math.max(nums2[0], nums2[1]);
        }
        return result;
    }

    /**
        该方法返回值是一个数组，长度为2。分别代表两种颜色的节点的数量
     */
    int[] divide(int[][] edges, boolean[] color) {
        List<List<Integer>> connections = new ArrayList<>();
        int n = edges.length + 1;
        for (int x = 0; x < n; x++) {
            connections.add(new ArrayList<>());
        }
        // 初始化并且填充connections集合
        for (int[] edge : edges) {
            connections.get(edge[0]).add(edge[1]);
            connections.get(edge[1]).add(edge[0]);
        }
        // 构造完毕
        // 遍历所有connection，并记录
        int ans = dfs(0, -1 , 0, connections, color);
        return new int[] {ans, n - ans};
    }

    int dfs (int n, int p, int depth, List<List<Integer>> connections, boolean[] colors) {
        List<Integer> list = connections.get(n);
        int ans = (depth %2==0) ? 1 : 0;
        colors[n] = (depth %2 == 0);
        for (int c : list) {
            if (c == p) {
                continue;
            }
            ans += dfs(c, n, depth + 1, connections, colors);
        }
        return ans;
    }
}

```