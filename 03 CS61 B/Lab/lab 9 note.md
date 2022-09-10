---
title: lab 9 note
date: 2022-09-01 周四
tags:
  - BST
  - Recurrsion

mindmap-plugin: basic
---
# lab 9 note

## 1: BSTMap

### get

比较简单，递归实现。

```java
private V getHelper(K key, Node p) {
        if (p == null) return null;
        int t = key.compareTo(p.key);
        if (t == 0) return p.value;
        else if (t > 0) return getHelper(key, p.right);
        else return getHelper(key, p.left);
    }

    /** Returns the value to which the specified key is mapped, or null if this
     *  map contains no mapping for the key.
     */
@Override
public V get(K key) {
	return getHelper(key, root);
}
```

### put

注意是修改了 p 的左/右节点后，返回 p，而不是返回 p 的左/右节点（我一开始就是这样的）。

否则会使根节点变为一棵子树。

```java
private Node putHelper(K key, V value, Node p) {
    if (p == null){
        size ++;
        return new Node(key, value);
    }
    int t = key.compareTo(p.key);
    if (t == 0) p.value = value;
    else if (t > 0) p.right = putHelper(key, value, p.right);
    else  p.left = putHelper(key, value, p.left);
    return p;
}

/** Inserts the key KEY
 *  If it is already present, updates value to be VALUE.
 */
@Override
public void put(K key, V value) {
    root = putHelper(key, value, root);
}
```

### size

维护代码中给的`size`即可


## 2: MyHashMap
### 任务
target: 

- Your implementation is required to implement `get`, `put`, and `size`. Other methods such as `remove`, `keySet`, and `iterator` are optional for this lab

provided:

- We’ve provided instance variables for you.
- We also provide the `clear` method 
- and a private `hash` method that computes the hash function of a key.

special:

- Unlike lecture (where each bucket was represented as a naked recursive linked list), each bucket in this lab is implemented as an `ArrayMap`

step:

- Start by implementing `get`, `put`, and `size` with no resizing. 
- After you’ve made figured these out, modify `put` so that it resizes. 
- (You should resize the array of buckets anytime the load factor exceeds `MAX_LF`, and you should resize multiplicatively)
- You can test your implementation using the `TestMyHashMap` class in the `lab9tester` package.


### get, put, size
都比较简单，比较特殊的是利用`buckets[k]`这个`ArrayMap`的大小变化来更新`MyHashMap`的大小
```java
public V get(K key) {
    int k = hash(key);
    return buckets[k].get(key);
}

public void put(K key, V value) {
    if (loadFactor() > MAX_LF){
        resize();
    }

    helperSet.add(key);
    int k = hash(key), t = buckets[k].size();
    buckets[k].put(key, value);
    this.size += buckets[k].size() - t;
}
public int size() {  
    return size;  
}
```

### resize
这个是大头，搞了一个多小时，因为一些细节没理解到位=-=
```java
private void resize() {  
    ArrayMap<K, V>[] new_buckets = new ArrayMap[buckets.length * 2];  
    for (int i = 0; i < new_buckets.length; i += 1) {  
        new_buckets[i] = new ArrayMap<>();  
    }  
  
    for(K key : helperSet){  
        int old_k = hash(key);  
        int new_k = Math.floorMod(key.hashCode(), buckets.length * 2);  
        V v = buckets[old_k].get(key);  
        new_buckets[new_k].put(key, v);  
    }  
    
    buckets = new_buckets;  
}
```
大体上：
- 首先，应该创建一个新的buckets，`new_buckets`，并对其进行初始化（类似`this.clear()`)
- 然后，将所有现有的`key`映射到`new_buckets`中。
- 最后将`this.buckets`赋值为`new_buckets`即可。

具体看第二步：

关键是：`hash()`函数是要用到`buckets.length`的，因此不同的bucket，对应的hashcode是不一样的，因此需要用不同方法，得到`key`的新旧两种hashcode，然后分别应用。

旧的hashcode用于从旧的`buckets`中得到对应的值。

新的hashcode用于在`new_buckets`建立映射。



虽然能解决问题，但感觉不太优雅，期待改进==
