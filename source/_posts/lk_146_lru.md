---
title: leetcode-146-[LRU Cache]
date: 2025-06-09T17:37:40+08:00
categories: 
- 算法
tags:
- LRU
---

Design a data structure that follows the constraints of a Least Recently Used (LRU) cache.

Implement the LRUCache class:

LRUCache(int capacity) Initialize the LRU cache with positive size capacity.
int get(int key) Return the value of the key if the key exists, otherwise return -1.
void put(int key, int value) Update the value of the key if the key exists. Otherwise, add the key-value pair to the cache. If the number of keys exceeds the capacity from this operation, evict the least recently used key.
The functions get and put must each run in O(1) average time complexity.

<!--more-->
[题目](https://leetcode.cn/problems/lru-cache/description/)

## 思路
LRU(``Least Recently Used``)是一种数据置换算法，核心思路就是将数据集合按照最近所使用的时间进行排序，越是近期使用过的，就会越在前面。在达到容器的上限(capacity)后，再次放入新的元素后会将最末尾（最久未被使用的元素）抛弃。
在这种规定下，我们每当put 或是get一个元素后，该元素都会被放置到头(head)处，若是put后元素数量超过上限，则需要将尾部(``tail``)抛弃
根据这种规则，我们可以发现，使用``链表``可以更好的实现将数据移动到头部，以及断开tail链接的操作。

需要注意的一点是，get到元素后，移动元素时，需要首先考虑该元素是否已经为head节点。若已经是head，则无需操作。否则需要进行移动操作。
在移动时，也需要注意，如果该节点是tail节点，那么需要将tail节点改变指向。

{% note info %}
需要注意，直接通过链表进行get，当调用次数过多，并且均未命中时，会导致每次都遍历一遍整个链表，导致超时。
此时我们需要使用缓存。我这里因为考虑到输入key的范围为[0, 10000], 所以直接使用了数组来存储node节点。在get时，最直接查询对应key索引处的node是否存在即可。
但是引入缓存后，需额外注意一点，就是在移除tail时需要将tail对应的缓存也一并移除，否则会查询到不存在的数据
{% endnote %}


## code
```java
class LRUCache {
    class Node {
        Node next;
        Node pre;
        int key;
        int value;

        Node() {
        };

        Node(int key, int val) {
            this.value = val;
            this.key = key;
        }
    }

    final Node root = new Node();
    Node tail;
    int capacity;
    int size;
    Node[] record = new Node[1_0001];

    public LRUCache(int capacity) {
        this.capacity = capacity;
    }

    public int get(int key) {
        Node c = findNode(key);
        if (c != null) {
            makeNodeHead(c);
            return c.value;
        }
        return -1;
    }

    public void put(int key, int value) {
        Node c = findNode(key);
        if (c == null) {
            c = new Node(key, value);
            record[key] = c;
            Node head = root.next;
            root.next = c;
            c.next = head;
            if (head != null) {
                head.pre = c;
            }
            if ((size) == capacity) {
                record[tail.key] = null;
                tail = tail.pre;
                tail.next = null;
            } else {
                if (tail == null) {
                    tail = c;
                }
                size++;
            }
        } else {
            c.value = value;
            makeNodeHead(c);
        }
    }

    Node findNode(int key) {
        if (record[key] != null) {
            return record[key];
        }
        
        return null;
    }

    void makeNodeHead(Node c) {
        if (root.next == c) {
            return;
        }
        Node cp = c.pre;
        Node ca = c.next;
        cp.next = ca;
        if (ca != null) {
            ca.pre = cp;
        }
        Node head = root.next;
        root.next = c;
        c.next = head;
        if (head != null) {
            head.pre = c;
        }
        if (tail == c) {
            tail = tail.pre;
        }
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```
