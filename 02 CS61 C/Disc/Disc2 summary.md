---
title: Disc2 summary
date: 2022-09-05 周一
tags:
  - c

mindmap-plugin: basic
---
# Disc2 summary

### 0.

When should you use the heap over the stack?

If you need to keep access to data over several function calls, use the heap. If you’re dealing with a large piece of data, passing around a pointer to something on the heap is more efficient and a better practice than passing around the data itself.

### 1.

```c
while(*str != '\0'){ 
    ...
    str ++ ;
}
// 可以改为
while(*str ++ ){ 
    ...
}
```

### 2.

```c
int bar(int *arr, size_t n) {
	int sum = 0, i;
	for (i = n; i > 0; i--)
	sum += !arr[i - 1];
	return ˜sum + 1;
}
```

Returns -1 times the number of zeroes in the first *N* elements of arr.

### 3.

```c
void baz(int x, int y) {
	x = x ˆ y;
	y = x ˆ y;
	x = x ˆ y;
}
```

Ultimately does not change the value of either x or y.

### 4. 


(Bonus: How do you write the *bitwise exclusive-nor* (XNOR) operator in C?)

`x == y`

### 5.

![image-20220905085914148](C:\Users\wei\AppData\Roaming\Typora\typora-user-images\image-20220905085914148.png)

- 缺null
- 有可能导致某些字符溢出