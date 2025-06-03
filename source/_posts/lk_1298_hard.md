---
title: leetcode_1298_Maximum Candies You Can Get from Boxes
date: 2025-06-03T22:11:02+08:00
tags:
- bfs
categories:
- 算法
---

![2025-06-03T221137](2025-06-03T221137.png)

<!-- more -->

## 描述
{%note info%}
You have n boxes labeled from 0 to n - 1. You are given four arrays: status, candies, keys, and containedBoxes where:

status[i] is 1 if the ith box is open and 0 if the ith box is closed,
candies[i] is the number of candies in the ith box,
keys[i] is a list of the labels of the boxes you can open after opening the ith box.
containedBoxes[i] is a list of the boxes you found inside the ith box.
You are given an integer array initialBoxes that contains the labels of the boxes you initially have. You can take all the candies in any open box and you can use the keys in it to open new boxes and you also can use the boxes you find in it.

Return the maximum number of candies you can get following the rules above.
{% endnote %}

## 思路
这道题我感觉比较简单，直接bfs遍历即可。
首先，我们需要三个集合（使用数组也可以），来记录我们当前所持有的boxes，我们当前可用的boxes，我们当前持有的钥匙
然后我们不断循环遍历可用的boxes集合，每当遍历一个元素就将其弹出。直至其为空
每遍历一个可用的boxes集合，我们就需要执行一下操作：
1. 将该box中的所有boxes取出放入持有的boxes集合中
2. 将该box中的所有钥匙取出放入持有的钥匙集合中
在一个阶段遍历完后，从持有boxes集合中不断取出box尝试与持有的keys集合来比对，若该box对应的status为1或者我们当前有该box的钥匙即可打开该盒子，即将其放入可用boxes集合中

## 代码
```java
class Solution {
    public int maxCandies(int[] status, int[] candies, int[][] keys, int[][] containedBoxes, int[] initialBoxes) {
        Set<Integer> availableBoxes = new HashSet<>();
        Set<Integer> holdBoxes = new HashSet<>();
        Set<Integer> holdKeys = new HashSet<>();
        // init
        for (int i : initialBoxes) {
            if (status[i] == 1) {
                availableBoxes.add(i);
                int[] subKeys = keys[i];
                for (int subKey : subKeys) {
                    holdKeys.add(subKey);
                }
                int[] subBoxes = containedBoxes[i];
                for (int subBox : subBoxes) {
                    holdBoxes.add(subBox);
                }
            } else {
                holdBoxes.add(i);
            }
            
        }
        int ans = 0;
        while (!availableBoxes.isEmpty()) {
            var it = availableBoxes.iterator();
            
            while (it.hasNext()) {
                int box = it.next();
                it.remove();
                ans += candies[box];
                
                // sub boxes
                int[] subBoxes = containedBoxes[box];
                for (int subBox : subBoxes) {
                    holdBoxes.add(subBox);
                }
                int[] subKeys = keys[box];
                for (int subKey : subKeys) {
                    holdKeys.add(subKey);
                }

            }
            var hbi = holdBoxes.iterator();
            // var hki = holdKeys.iterator();
            while (hbi.hasNext()) {
                int holdNextBox =  hbi.next();
                if (holdKeys.contains(holdNextBox) || status[holdNextBox] == 1) {
                    hbi.remove();
                    holdKeys.remove(holdNextBox);
                    availableBoxes.add(holdNextBox);
                }
            }
        }
        return ans;
    }
}
```