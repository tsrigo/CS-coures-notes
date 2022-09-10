---
title: lab 1 note
date: 2022-09-01 周四
tags:
  - GDB
  - C

mindmap-plugin: basic
---
# lab 1 note

## Exercise 1

### Compiling and Running a C Program

basic

```shell
gcc ex1.c test_ex1.c
./a.out
```

自定义输出名

```shell
gcc -o ex1 ex1.c test_ex1.c
./ex1
```

### Assert Statements

> assert - abort the program if assertion is false

SYNOPSIS

```c
#include <assert.h>
void assert(scalar expression);
```

既可以用于测试，也可以作为一种使程序正确运行下去的保证（出错可以很快进行定位）

## Exercise 2

### Intro to GDB: start, step, next, finish, print, quit

> 若要使用cgdb，在用gcc编译时应加上参数'-g'

| Command       | Abbreviation | Description                                                  |
| ------------- | ------------ | ------------------------------------------------------------ |
| start         | N/A          | begin running the program and stop at line 1 in main         |
| step          | s            | execute the current line of code (this command will step into functions) |
| next          | n            | execute the current line of code (this command will not step into functions) |
| finish        | fin          | executes the remainder of the current function and returns to the calling function |
| print \[arg\] | p            | prints the value of the argument                             |
| quit          | q            | exits gdb                                                    |

### Intro to GDB: break, conditional break, run, continue

| Command                                                      | Abbreviation      | Description                                                  |
| ------------------------------------------------------------ | ----------------- | ------------------------------------------------------------ |
| break \[line num or function name\](ex: break pwd_checker.c:check_number) | b                 | set a breakpoint at the specified location                   |
| conditional break (ex: break 3 if n==4)                      | (ex: b 3 if n==4) | set a breakpoint at the specified location only if a given condition is met |
| run                                                          | r                 | execute the program until termination or reaching a breakpoint |
| continue                                                     | c                 | continues the execution of a program that was paused         |

这里continue和run的区别？

## Exercise 3

主要遇到了段错误和代码逻辑的问题，详见注释。

```c
#include <stddef.h>
#include <assert.h>
#include "ll_cycle.h"

int node_cmp(node *a, node *b){
    // 判断node类型的 a 和 b 是否相等
    // 很重要的一点是：a 和 b 都不能为空，否则会出现段错误。
    // 添加下面两个assert，有助于debug（因为可能出现段错误的地方还有赋值部分）
    assert(a != NULL);    
    assert(b != NULL);
    return (*a).value == (*b).value && (*a).next == (*b).next;
}

int ll_has_cycle(node *head) {
    /* TODO: Implement ll_has_cycle */
    node *fast, *slow;
    slow = fast = head;

    while (fast != NULL){  
        // 下面这个判断是最关键的，一方面，next不能为空，否则不能给fast赋值（next的next）
        // 另一方面，next的next不能为空，否则待会进行比较时，会出现段错误。
        if ((*fast).next != NULL && (*((*fast).next)).next != NULL) fast = (*((*fast).next)).next;
        else return 0;
        slow = (*slow).next;
        // 上面已经判断过空指针了（形式完全一样），因此对于slow则不用判断。
        if (node_cmp(fast, slow)) return 1;
    }
    return 0;
}   

```

## Other Useful GDB Commands (Recommended)

### Command: `info locals`

Prints the value of all of the local variables in the current stack frame

### Command: `command`

Executes a list of commands every time a break point is reached. For example:

Set a breakpoint:

```
b 73 
```

Type `commands` followed by the **breakpoint number**:

```
commands 1 
```

Type the list of commands that you want to execute separated by a new line. After your list of commands, type `end` and hit Enter.

```
p var1
p var2
end 
```

### Command: `delete`

Deletes the specified breakpoint. See the [reference card](https://inst.eecs.berkeley.edu/~cs61c/resources/gdb5-refcard.pdf) for more info.

