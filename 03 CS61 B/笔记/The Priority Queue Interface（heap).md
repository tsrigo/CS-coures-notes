---
title: The Priority Queue Interface（heap)
date: 2022-09-04 周日
tags:
  - 数据结构
  - heap
  - 笔记

mindmap-plugin: basic
---
# The Priority Queue Interface（heap)

记录几个亮点

## 1. 雏形

**Exercise 13.1.1.** Complete the method listed above, `unharmoniousTexts` with the same functionality as described using only $\Theta (M)$ space.

维护一个长度为$M$的`arraylist`，而不是包含所有信息的`arraylist`。

```java
for (Timer timer = new Timer(); timer.hours() < 24; ) {
    allMessages.add(sniffer.getNextMessage());
    if (allMessages.size() > M){		// 关键
        allMessages.removeSmallest();
    }
}
```

## 2. Heap Operations

最巧妙的Swim up和Sink down操作

- `add` : Add to the end of heap temporarily. **Swim up** the hierarchy to the proper place.
  - Swimming involves swapping nodes if child < parent
  
- `getSmallest`: Return the root of the heap (This is guaranteed to be the minimum by our *min-heap* property

- `removeSmallest`: Swap the last item in the heap into the root. **Sink down** the hierarchy to the proper place.
- Sinking involves swapping nodes if parent > child. Swap with the smallest child to preserve *min-heap* property.

## 3. Tree Representation

![image-20220904104752637](C:\Users\wei\AppData\Roaming\Typora\typora-user-images\image-20220904104752637.png)

![image-20220904104902818](C:\Users\wei\AppData\Roaming\Typora\typora-user-images\image-20220904104902818.png)

三类表示方法，数组一类最为巧妙。

- 利用了满二叉树的性质，建立了下标与父子节点的关系，节省空间，代码量
- 增加哨兵字符，使得`parent()`函数具有普遍性（不用特判根节点）

## 4. Comparing to alternative implementations

![image-20220904105114702](C:\Users\wei\AppData\Roaming\Typora\typora-user-images\image-20220904105114702.png)