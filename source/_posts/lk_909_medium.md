---
title: leetcode_909 Snakes and Ladders
date: 2025-05-31T13:36:38+08:00
categories: 
- 算法
tags: 
- bfs
---
You are given an n x n integer matrix board where the cells are labeled from 1 to n2 in a Boustrophedon style starting from the bottom left of the board (i.e. board[n - 1][0]) and alternating direction each row.

You start on square 1 of the board. In each move, starting from square curr, do the following:

Choose a destination square next with a label in the range [curr + 1, min(curr + 6, n2)].
This choice simulates the result of a standard 6-sided die roll: i.e., there are always at most 6 destinations, regardless of the size of the board.
If next has a snake or ladder, you must move to the destination of that snake or ladder. Otherwise, you move to next.
The game ends when you reach the square n2.
A board square on row r and column c has a snake or ladder if board[r][c] != -1. The destination of that snake or ladder is board[r][c]. Squares 1 and n2 are not the starting points of any snake or ladder.

Note that you only take a snake or ladder at most once per dice roll. If the destination to a snake or ladder is the start of another snake or ladder, you do not follow the subsequent snake or ladder.

For example, suppose the board is [[-1,4],[-1,3]], and on the first move, your destination square is 2. You follow the ladder to square 3, but do not follow the subsequent ladder to 4.
Return the least number of dice rolls required to reach the square n2. If it is not possible to reach the square, return -1.

<!--more-->

[题目](https://leetcode.cn/problems/snakes-and-ladders/description)

![2025-05-31T134744](2025-05-31T134744.png)

## 思路
题目中给定的棋盘(board)，是按照左右交互(``Boustrophedon``)的方式来排列的.并且需要注意，棋盘位置中的值是从1开始的，但是我们代码中的数组下标是从0开始的，那么我们就需要注意，遇到梯子（ladder）或者蛇（snake）的时候，跳跃到指定格子时，根据格子的值来寻找坐标时，需要将当前的值进行减一的操作后，再去寻找坐标。
然后因为是需要找到到达(n * n)的最短方式，又由于在此题中，每次移动均会产生相同的一个步骤(step)的消耗，所以可以使用bfs(Breadth First Search)来遍历。此方法的优点是只要找到了(n * n)的点，那么即可停止遍历。如果使用dfs，那么需要收集所有点的信息，会比较麻烦一些

## 代码
```java
class Solution {
    public int snakesAndLadders(int[][] board) {
        int n = board.length;
        // System.out.println(computeCol(6,6));
        int[][] records = new int[n][n];

        for (int[] record : records) {
            Arrays.fill(record, -1);
        }
        Deque<Integer> lastPath = new LinkedList<>();
        lastPath.offerLast(0);
        
        int destination = n * n - 1;
        int ans = 0;
        while (!lastPath.isEmpty()) {
            int size = lastPath.size();
            for (int i = 0; i < size; i++) {
                int target = lastPath.pollFirst();
                if (target == destination) {
                    return ans;
                }
                int row = computeRow(n,target);

                int c = computeCol(n,target);
                if (records[row][c] == -1) { // 尚未遍历过当前节点
                    records[row][c] = ans;
                    // 把当前节点接下来可能到达的节点遍历放入lastPath
                    for (int j = 1; j <= 6; j++) {
                        if (target + j > destination) {
                            break;
                        }
                        int drow = computeRow(n,target + j);
                        int dc = computeCol(n,target + j);
                        // System.out.println(drow +" _  " + dc);
                        // System.out.println(target + j);
                        if (board[drow][dc] == -1) {// 该节点是normal节点
                            lastPath.offerLast(target + j);
                        } else {
                            int newTarget = board[drow][dc]; // 该节点是梯子或者蛇，我们需要转移到对应的节点
                            lastPath.offerLast(newTarget-1);
                        }
                    }
                }
                
            }

            ans++;
        }
        return -1;
    }

    int computeRow(int n, int target) {
        return ((n - 1 - ((target) / n)));
    }

    int computeCol(int n, int target) {
        int num = target%n;
        int tmpRow = target/n;
        return (tmpRow%2 == 0)? (num):(n-1-num);
    }
}
```