---
title: LRU缓存
date: 2022-04-21 22:51:48
tags:
  - LRU缓存
categories:
  - 数据结构与算法
---

LRU，Least Recently Used，即最近最少使用，是一种常用的缓存置换算法，它在缓存中选择最近最久未使用的数据予以淘汰。

## 使用双链表和哈希表构建

### 双链表

```go
type Node struct {
    key int
    val int
    prev *Node
    next *Node
}
type dllist struct {
    head *Node
    tail *Node
}
func NewDllist() dllist {
    a, b := new(Node), new(Node)
    a.next, b.prev = b, a
    return dllist{
        head: a,
        tail: b,
    }
}
func(d *dllist) remove(node *Node) {
    a, b := node.prev, node.next
    a.next, b.prev = b, a
}
func(d *dllist) insert(node *Node) {
    a, b, c := d.head, node, d.head.next
    a.next, b.prev, b.next, c.prev = b, a, c, b
}
```

### LRU Cache 基本操作 get 和 put

```go
type LRUCache struct {
    list dllist
    dict map[int]*Node
    len_ int
    cap_ int
}
func NewLRUCache(capacity int) LRUCache {
    return LRUCache{
        list: NewDllist(),
        dict: make(map[int]*Node),
        len_: 0,
        cap_: capacity,
    }
}
func(l *LRUCache)get(key int) {
    val := -1
    if node, ok := l.dict[key]; ok {
        val = node.val
        l.list.remove(node)
        l.list.insert(node)
    }
    return val
}
func(l *LRUCache)put(key int, val int) {
    if node, ok := l.dict[key]; ok {
        node.val = val
        l.list.remove(node)
        l.list.insert(node)
        return
    }
    if l.len_ < l.cap_ {
        l.len_++
    } else {
        delete(l.dict, l.list.tail.prev.key)
        l.list.remove(l.list.tail.prev)
    }
    node := &Node{
        key: key,
        val: val,
    }
    l.dict[key] = node
    l.list.insert(node)
}
```
