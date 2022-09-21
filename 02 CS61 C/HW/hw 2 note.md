---
title: hw 2 note
date: 2022-09-07 周三
tags:
  - 位运算
  - c
  - Memory_Management
  - Pointer
  - Struct
  - union
  - endianness
mindmap-plugin: basic
---
# hw2 note

## HW2.1、2.2

位运算，略

## HW2.3. Memory Alpha Model

```c
#define SPOCK 1701
int KIRK = 1701;
int sulu(int scotty) {
	return scotty * scotty;
}
int main(int argc, char *argv[]) {
	int *chekov = malloc(sizeof(int) * 1701);
	if (chekov) free(chekov);
	sulu(SPOCK); // ← snapshot just before it returns
	return 0;
}
```

| 名称       | 位置        | 类型                         |
| ---------- | ----------- | ---------------------------- |
| `sulu`     | code        | 函数                         |
| `checkov`  | stack       | 局部变量（malloc分配的指针） |
| `*checkov` | heap        | malloc分配的指针指向的变量   |
| `KIRK`     | static/data | 静态变量（全局变量）         |
| `scotty`   | stack       | 局部变量                     |
| `SPOCK`    | code        | 宏定义                       |

Q1.1: `sulu` is a **function**, and therefore it is stored in the code.

Q1.2: The variable `chekov` itself is stored in the stack, since it's a **local variable**. Note that the value of `chekov` is a pointer elsewhere.

Q1.3: Since we set `chekov` to a pointer returned by `malloc`, `*chekov` is located on the heap.

Q1.4: `KIRK` is declared **outside any function** and is a variable, so it gets stored in the static/data segment.

Q1.5: `scotty` is a **local variable** (as a function argument) to the function `sulu`, so it gets stored on the stack. Note that the value of SPOCK gets copied into a new local variable,since C is pass-by-value.

Q1.6: `SPOCK` is a defined by a `#define` statement. Effectively, all instances of `SPOCK` get replaced with 1701 by the compiler before it continues to compile the code. As such, the value of `SPOCK` gets copied directly into the code

## HW2.4. Memory Multiple Choice

### 1. Which parts of memory can fall under that category?  

Data, Heap, and Stack generally contain segments of memory allocated by the user, but also contain buffers/metadata which help the system keep track of everything, and can contain data/heap/stack data of other processes.  

所以在这些地方都有可能出现段错误。

### 2. What are general reasons such protections exist within a specific process? 

From a debugging perspective, it is generally not correct behavior to access a memory location you didn't originally initialize. 

As such, segfaults allow you to catch bugs more easily.



## HW2.5. Memory Endianness  

Let x be an 8-bit unsigned number. We consider the "first bit" to be **the least significant bit**, and the "eighth bit" to be **the most significant bit**. 

### 1.

```c
Suppose that we have the following bytes stored in memory:
0x00001000: 0x00
0x00001001: 0x01
0x00001002: 0xFF
0x00001003: 0xFF
```

big-endian = 0x0001FFFF = 131071

little-endian = 0xFFFF0100(补码) = 0x8000FF00(原码) = -65280

### 2.

```c
uint32_t *x = malloc(sizeof(uint32_t)*4); //Assume that x == 0x20000000
x[0] = 0xDEADBEEF;
x[1] = 0xC561C156;
x[2] = 0x00DC1A55;
x[3] = 0xABADCAFE;
uint64_t y = *((uint64_t*)x);
```

Q2.1: The byte at address `0x2000000E `is `0xAD `(the 15th byte)
Q2.2: strlen goes to the **first 0x00 byte**, which is the 12th byte. Our string length is thus 11.
Q2.3: We interpret the first 8 bytes as a little-endian integer, so we get 0xC561C156DEADBEEF. Note that this is the same as x[1] and x[0] in order.

主要是理解下表以及little-endian

| 地址 | (1)字节 |
| ---------- | ---- |
| 0x20000000 | 0xEF |
| 0x20000001 | 0xBE |
| 0x20000002 | 0xAD |
| 0x20000003 | 0xDE |
| 0x20000004 | 0x56 |
| 0x20000005 | 0xC1 |
| 0x20000006 | 0x61 |
| 0x20000007 | 0xC5 |
| 0x20000008 | 0x55 |
| 0x20000009 | 0x1A |
| 0x2000000A | 0xDC |
| 0x2000000B | **0x00** |
| 0x2000000C | 0xFE |
| 0x2000000D | 0xCA |
| **0x2000000E** | **0xAD** |
| 0x2000000F | 0xAB |

## HW2.6. Pointers

注意一点，数组名相当于一个指针，这个指针指向它自己，也就是说`&x = x = 0x188c`

> An array variable is a “pointer” to the first (0-th) element.

> K&R: “An array name is not a variable”

## HW2.7  Pointers and Structs

### 1.

```c
void changeX2 (Point *pt) {
	pt.x = 22;
}

Point my_pt;
changeX2(&my_pt);
printf("%d\n", my_pt.x);
```

Q1.3: The `.` operator to retrieve a field from a struct only works on **a struct** and **not a struct pointer**, so this code would not compile.

### 2.

```c
void changeX5 (Point *pt) {
	pt = 55;
}
Point *my_pt = calloc(1, sizeof(Point));
changeX5(my_pt);
printf("%d\n", my_pt->x);
```

Q1.5: C is pass by value. This code passes the value of the pointer to changeX5 which mutates its local copy, and does nothing to change my_pt. Since the my_pt was calloc-ed, all of its fields were initialized to 0 and remain equal to 0

## HW2.8. Strings

要充分理解`'\0'`在字符串中的意义，`strlen()`就是根据它来计算字符串长度的。

几种形式

`char* a = "foobar";`(不可修改)

`char a[] = "foobar"`

`char a[] = {'f', 'o', 'o', 'b', 'a', 'r', '\0'};`(可修改，与上面的相同，但要手动添加`'\0'`)

`char* a = "foo\0bar";`（此时`strlen(a) == 3`)



## HW2.9. Common C Pitfalls

### 1.

![image-20220907091817214](https://s2.loli.net/2022/09/07/reDUHohqQnsaxTX.png)

申请空间过大，会失败，此时malloc不会报错，只会返回`NULL`。因此应该添加一段判断语句，`if(x != NULL)`，才继续进行操作。

### 2.

```c
int foo () {
	int *x = malloc(20);
	x[0] = x[1] = 1;
	x[2] = x[0] + x[1];
	x[3] = 99;
	return x[3];
}
```

未`free`

### 3.

```c
int main () {
	char *x = "patrick";
	printf("%s", x);
	free(x); // tidy up
}
```

不能`free`非malloc分配的区域

### 4.

```c
void strncpy(char *destination, char *source, uint32_t maxlength)
{
    int i = 0;
    while (i < maxlength && source[i])
    {
        destination[i] = source[i];
        i++;
    }
    destination[strlen(destination)] = '\0';
}
```

最后一行会把最后一个字符（而不是'\0')赋值为'\0'

### 5.

```c
#DEFINE MAXSTRINGLENGTH 64 // Assume that the string input is at most 64
characters long char *copystringtoheap(char *string)
{
    char *c = malloc(sizeof(char) * (MAXSTRINGLENGTH + 2));
    char *cclone = c;
    do
    {
        *(c++) = *string;
    } while (*(string++));
    free(c);
    return cclone;
}
```

只能`free`最开始`malloc`的那个指针

## HW2.10. Reading Memory

str比较简单就略过了，主要是树

首先清楚这段代码的几个点

- 树的右节点要么指向子树，要么是值
- 树的左节点要么指向子树，要么为空
- 叶子节点的左节点为空，右节点是值
- `printtree()`的打印顺序从左到右叶子节点的值

### q1

如图所示

![image-20220907094458700](https://s2.loli.net/2022/09/07/HUsuJg29DVlyTOp.png)

从左到右打印`3 1 4 1 5 9 3`

### q2

这题推荐用vscode做，颜色比较丰富

#### 首先解释以下下面这张图

![image-20220907095005091](https://s2.loli.net/2022/09/07/KAkvwUQuqyC6seW.png)

- 每一行有四个单元，每个单元的大小是四个字节（4 bytes），每对十六进制数的大小是一个字节（1 byte = 8 bits）
- 按行看，从左到右单元的地址是增大的；按单元看，从左到右每一位的地址是减小的（little-endian）
- 两行的地址相差"10"，这里的"10"是16进制的，相当于十进制的"16"，与第一点是吻合的

### 接下来是地址

![image-20220907100001960](https://s2.loli.net/2022/09/07/HLu7l42weEFr8CA.png)

由于是64位的机器，因此地址（指针）的大小是64位，也就是8字节，也就是两个单元，也可以说是8对（16个）十六进制数。

但是这里为什么长度只有12呢，因为前面几位都是0，所以省略了（可能有更丰富的原因）

`0x7fffffffe350`是a1的地址，该地址的值是`0xef`

类似的，下面用表格列出`str`的表示

| 地址           | 值   | ascii（16） | str[]  |
| -------------- | ---- | ----------- | ------ |
| 0x7fffffffe360 | 20   | [space]     | str[0] |
| 0x7fffffffe361 | 69   | i           | str[1] |
| 0x7fffffffe362 | 73   | s           | str[2] |
| 0x7fffffffe363 | 20   | [space]     | str[3] |
| 0x7fffffffe364 | 66   | f           | str[4] |
| 0x7fffffffe365 | 75   | u           | str[5] |
| 0x7fffffffe366 | 6e   | n           | str[6] |
| 0x7fffffffe367 | 00   | [null]      | str[7] |

所以`str = " is fun"`，注意最前面有一个空格

### 接下来是复原树

由上图知树的地址的地址`0x7fffffffe358`，也就是说树的地址存储在这里。从这里开始取8位就可以得到树的(根节点)地址`0x00005555 557577f0`(两个单元），这里就不再赘述。

找到这一行

![image-20220907102035439](https://s2.loli.net/2022/09/07/8YnhogSTleBUavz.png)

可以看到这一行有四个单元，回顾代码中树的定义，可以知道这里对应了两个地址`left, right`。

顺便说一句，union的作用就是划定一段足够的空间，放置自定义的若干种数据的其中一种，在这里，指针需要两个单元，int需要1个单元格，所以这俩单元就是union规划的空间，足以容纳指针或者int。由于只有两种值，稍后我们可以很容易判断出来什么时候是指针，什么时候是值。

```c
typedef struct tree {
  struct tree* left;
  union 
  {
    struct tree* right;
    int data;
  } rd;
} Inttree;
```

知道了这点，我们就知道，根节点(`0x5555557577f0`)的左右各有一棵子树，地址分别是`0x00005555 55757340, 0x00005555 557572e0`

以此类推，可以不断找下去，强烈建议准备一个草稿纸，一步一步画出树的结构，下图是我的草稿（以地址的最后3位为代表，前面均为`0x00005555 55757...`）。

![image-20220907103430132](https://s2.loli.net/2022/09/07/ROowAeupijNdKs9.png)

中间过程我就不再赘述，这里再给出叶子节点(`0x00005555 55757320`)的例子。

![image-20220907103700871](https://s2.loli.net/2022/09/07/vV8TJLl9gCD5dSz.png)

可以看到，320这一行，左边两个单元为空，说明它是叶子节点，那么就可以看第三个单元，就是它的值（int类型）。这时右边两个单元不表示指针，据此也可以更好地理解`union`。



tips：这里有一个坑点，如果用了暗色主题，`2a0`对应的值可能一时难以发现，如下图

![image-20220907103935418](https://s2.loli.net/2022/09/07/ClUMSwkBAI681hX.png)



最后画出整棵树，还要注意值都是**16进制**的，还需要进行进一步的转换，然后从左到右列举，就能得到答案: `6 28 496 8128 33550336 28 28`.
