---
title: leetcode-135-Candy
date: 2025-06-02T20:27:36+08:00
categories:
- 算法
---

There are n children standing in a line. Each child is assigned a rating value given in the integer array ratings.

You are giving candies to these children subjected to the following requirements:

Each child must have at least one candy.
Children with a higher rating get more candies than their neighbors.
Return the minimum number of candies you need to have to distribute the candies to the children.

<!-- more -->

![2025-06-02T203114](2025-06-02T203114.png)

[题目](https://leetcode.cn/problems/candy/description/)

## 思路
对于这个题，乍一看以为很简单，以为只需要遍历ratings 数组找到高点更新其值即可。但是看了测试用例仔细想过后，发现并不是这么简单。
首先，一个元素的rating高于左侧的元素的rating，那么该索引处获得的糖果需要比左侧高，右侧同理。
* 一开始我想到了从前向后遍历，若当前index的rating比前一个元素的rating高，那么就将其更新为前一个元素的candy + 1，否则保持1.这样做看起来很好，但是如果遇见连续降序的rating排列就不奏效了：因为连续降序，所以遍历到的每一个元素都比前一个低，按照前面说的，我们需要将其全部保持为1.但是对于连续降序的排列，我们应该将前一个索引的糖果进行增加。
* 于是乎，我就想到了在出现降序排列时，就对candy数组中之前的索引进行回溯，对每个索引对应的rating与其之索引的rating进行判断，若当前元素大于后一个索引的rating，那么就将该元素对应的candy变为其之后的索引的candy数+1，然后不断向前(--)遍历。此时，大部分的测试用例均可以通过。但是仍有一些会因为超时无法通过
* 根据观察，超时无法通过的测试用例是一个长达20000，整个数据均为降序的排列。按照我们之前写的代码，我们每当遇到降序就进行回溯更新，会导致我们超时。所以，我发现回溯可以暂时放置，只有在出现增序或者数组完毕后，再进行回溯，这样可以极大的减少回溯次数，即使回溯次数很多，也能保证回溯的长度很短，保证时间复杂度。
* 最后，由于处理不慎，我的算法对于最后一位的元素判断出现了些问题。最后终于解决，解决方法就是在回溯后对当前元素以及之前元素进行判断（即类似第一步的想法）


## 代码
```java
class Solution {
    public int candy(int[] ratings) {
        int n = ratings.length;
        int[] record = new int[n];
        Deque<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (i > 0 && ratings[i] >= ratings[i - 1]) {

                int size = queue.size();
                for (int x = 0; x < size; x++) {
                    int v = queue.pollLast();
                    if (ratings[v] > ratings[v + 1] && record[v] <= record[v + 1]) {
                        record[v] = record[v + 1] + 1;
                    }
                    // queue.offerFirst(v);
                }
                if (ratings[i] > ratings[i - 1] && record[i] <= record[i - 1])
                    record[i] = record[i - 1] + 1;
                queue.offerLast(i);
            } else if (i == 0 || ratings[i] < ratings[i - 1]) {
                // int size = queue.size();
                // for (int x = 0; x < size; x++) {
                //     int v = queue.pollLast();
                //     if (record[v] <= record[v + 1]) {
                //         record[v] = record[v+1] + 1;
                //     }
                //     queue.offerFirst(v);
                // }
                queue.offerLast(i);
            }
            // System.out.println(Arrays.toString(record));
        }
        int size = queue.size();
        for (int x = 0; x < size; x++) {
            int v = queue.pollLast();
            if (v < n - 1 && ratings[v] > ratings[v + 1] && record[v] <= record[v + 1]) {
                record[v] = record[v + 1] + 1;
            }
            // queue.offerFirst(v);
        }
        System.out.println(Arrays.toString(record));
        int sum = 0;
        for (int num : record) {
            sum += num;
        }
        sum += n;
        return sum;
    }
}
```