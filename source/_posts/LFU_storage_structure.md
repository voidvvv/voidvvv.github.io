---
title: LFU_storage_structure
date: 2025-06-13T21:04:41+08:00
tags:
---

there are two main replacement policies used in redis, LRU(https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU)  and [LFU](https://en.wikipedia.org/wiki/Least_frequently_used)

the LRU we can solve by use a simple LinkList [code](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU), but the LFU is a bit of difficult. It's necessary for me to write a note to record the way of thinking I have about it.
<!--more-->
## Thinking
according to the [LFU definition] (https://en.wikipedia.org/wiki/Least_frequently_used), it's a storage structure that would replace the least used element in this container when the size is reached out the capacity, to achieve this , we need to record the elements in a list order by used count, this is easy when we use the LinkList. as for the Node in LinkList, we need add a num to indicate how many times this node is called, since it's default value is , because every node is added to the LinkList is operated one time.

In the very beginning, I made a mistake to operate the tail element. when we get or update the tail, we need to do an extra check to see if this element is the only one in the record, and this is important.
at mean time, we need to iterate to get the proper position for the node currently be edited, and this is also a pitfall I encountered. 

as same as LRU, I used an array to store every node(element), so that we can get each one by O(1) since we use LinkList which is not very easy to query a specific node (normal O(1)).

## Code 
```java
class LFUCache {
    class Node {
        int key;
        int val;
        Node next;
        Node pre;
        int cnt = 1;

        public Node() {
        };

        public Node(int key, int value) {
            this.key = key;
            this.val = value;
        }
    }

    int capacity = 0;
    final Node header = new Node();
    Node tail = null;
    int size = 0;
    Node[] record;
    public LFUCache(int capacity) {
        this.capacity = capacity;
        record = new Node[100_000];
        tail = header;
    }
    
    public int get(int key) {
        Node node = findByKey(key);
        if (node != null) {
            return node.val;
        }
        return -1;
    }
    
    public void put(int key, int value) {
        Node node = findByKey(key);
        if (node != null) {
            node.val = value;
        } else {
            node = new Node(key, value);
            while (size >= capacity) {
                removeTail();
            }
            addNewNode(node);
        }
    }

    Node findByKey(int key) {
        Node node = record[key];
        if (node != null) {
            
            node.cnt++;
            moveAheade(node);
        }
        return node;
    }

    void moveAheade (Node node) {
        Node pre = node.pre;
        Node initialPre = pre;
        Node next = node.next;
        pre.next = next;
        if (next != null) {
            next.pre = pre;
        }
        while (pre != header && pre.cnt <= node.cnt) {
            pre = pre.pre;
        }
        Node pn = pre.next;
        pre.next = node;
        node.pre = pre;
        node.next = pn;
        if (pn != null) {
            pn.pre = node;
        }

        if (tail == node) {
            tail = initialPre == pre ? node : initialPre;
        }
    }

    void removeTail () {
        if (tail == header || tail == null) {
            return;
        }
        record[tail.key] = null;
        tail = tail.pre;
        tail.next = null;
        size--;
    } 

    void addNewNode (Node node) {
        tail.next = node;
        node.pre = tail;
        tail = node;
        moveAheade(node);
                record[node.key] = node;

        size++;
    }  
}

/**
 * Your LFUCache object will be instantiated and called as such:
 * LFUCache obj = new LFUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```
