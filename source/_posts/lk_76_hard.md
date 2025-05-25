---
title: 手撸算法 - 力扣 76. Minimum Window Substring
date: 2025-05-21T23:35:56+08:00
categories: 
- 算法
tags: 
- 滑动窗口
---

![2025-05-21T233755](2025-05-21T233755.png)
没有看题解手撸出来的hard，开心
<!-- more -->

## 思路
思路就是考虑到字符串中的字符集是有限个数的，那么就用一个数组(bits)来存储t所有字符集的个数，同时也可以根据这个来得到t中初始包括哪些字符集。
同时，需要再生成一个相同的数字(modi)，用来在对s遍历时，对已经出现过存在于t中的字符集进行统计。
遍历的思路，就是使用双端队列，用来记录当前出现t中字符集的字符在s中的下标，从头到尾表示从小到大。first代表最开始，last代表最后。
每次遍历到一个字符，判断其是否存在与t中(使用bits数组对应下标数据是否为正数来判断), 若其存在于t中，则将其推入队列尾部，并且在(modi)中对于该字符的计数进行减减。此时我们可以明显看到，若modi中的数值减少到0以下，那么证明该字符已经**溢出**了，我们可以尝试着减掉之前出现过的这些字符.
此时需要进行一个while循环判断，若双端队列不为空，并且头部元素所对应的字符在modi中的数值为负数，那么证明该字符是**溢出**的，是可以被移除的，此时的操作就是从队列中移除该元素，然后使其在modi中对应的数量加加.

最后，我们peek出来队列首位的元素，与当前记录的首位元素下标进行判断，若当前peek的下标插值小于记录的下标插值，那么就将记录的下标替换为我们当前peek的数据。

最后，判断是否取出过符合记录的下标，若没有，按照题目规定返回""， 否则从s中取相对应的字串即可

## 代码： 
```java
class Solution {
    public String minWindow(String s, String t) {
        int[] bits = new int[52];
        int[] modi = new int[52];
        for (int x = 0; x < t.length(); x++) {
            addArr(bits, t.charAt(x));
            addArr(modi, t.charAt(x));
        }
        Deque<Integer> deque = new LinkedList<>();
        int si = -1;
        int ei = -1;
        for (int x=0; x<s.length(); x++) {
            char cc = s.charAt(x);
            if (getBit(bits, cc) > 0) {
                deque.offerLast(x);
                subArr(modi, cc);
                while (!deque.isEmpty()  && getBit(modi, s.charAt(deque.peekFirst())) < 0) {
                    addArr(modi, s.charAt(deque.peekFirst()));
                    deque.pollFirst();
                }
                // if (getBit(modi, cc) < 0 && !deque.isEmpty() && s.charAt(deque.peekFirst()) == cc) {
                //     addArr(modi, cc);
                //     deque.pollFirst();
                // }
                if (allZero(modi)) {
                    if (si == -1) {
                        si = deque.peekFirst();
                    }
                    if (ei == -1) {
                        ei = deque.peekLast();
                    }
                    if ((ei - si) > (deque.peekLast() - deque.peekFirst())) {
                        si = deque.peekFirst();
                        ei = deque.peekLast();
                    }
                }

            }

        }
        if (si == -1) {
            return "";
        } else {
            return s.substring(si, ei+1);
        }

    }

    boolean allZero (int[] arr) {
        for (int i : arr) {
            if (i > 0) {
                return false;
            }
        }
        return true;
    }

    int getBit (int[] arr, char c) {
        if (c < 'a' || c > 'z') {
            return arr[ (c - 'A' + 26)];
        } else {
            return arr[c - 'a'];
        }
    }

    void subArr (int[] arr, char c) {
        if (c < 'a' || c > 'z') {
            arr[ (c - 'A' + 26)]--;
        } else {
            arr[c - 'a']--;
        }
    }

    void addArr (int[] arr, char c) {
        if (c < 'a' || c > 'z') {
            arr[ (c - 'A' + 26)]++;
        } else {
            arr[c - 'a']++;
        }
    }
}

```

