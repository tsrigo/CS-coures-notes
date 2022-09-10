---
title: Project 1 snek
date: 2022-09-09 周五
tags:
  - c
  - Memory_Management
  - pointer
  - struct
mindmap-plugin: basic
---
# Project 1: snek

TODO: 总结一下指针使用的几种错误

## Task 1: `create_default_state`

比较复杂的是`game_state_t`里的`**board`，特此记录。

首先，数据肯定是要保存在堆里的，我开始的思路就是先用malloc生成一个指针，再定义一个局部数组，然后把数组的字符串往里复制。大体是对的，但是实现起来有很多细节问题。

### 局部数组的声明

正确做法：

```c
char *t_board[19] = {
// 法二：char t_board[19][21] = {...}
    "####################",
    "#                  #",
    "# d>D    *         #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "#                  #",
    "####################"};
```

表示t_board是有19个字符指针的数组，法二类似，不过额外规定规定了长度。

错误做法：

```c
char t_board[19] = {...}
char **t_board = {...}
```

### 字符串的复制

```c
char **board = malloc(sizeof(char) * 8 * 20);
for (int i = 0; i < 18; i++)
{
  board[i] = malloc(sizeof(char) * 21);
  if (board[i] == NULL)
    return NULL;
  strcpy(board[i], t_board[i]);
}
```

这里格外注意，`board`表示的也是有若干个字符指针的数组，每个字符指针指向游戏的一行。一开始我给它malloc了18*20的空间（行×列），这实际上是错误的。正确做法应该是给它分配足够容纳[行数]个字符指针的空间，这里，我们需要分配空间来存储指针。

理解了这个，下面才能意识到复制字符串时，也需要malloc，即，我们需要分配空间来存储字符串。

然后就可以把这段空间的地址存储到board中，接着再使用strcpy。



如果不进行二次malloc，board内的内容可能是下面这样子的，很显然，有很多垃圾信息

```c
00000150447765e0
0000015044770150
65775c7372657355
7461447070415c69
5c6c61636f4c5c61
726573555c3a433d
70415c6965775c73
6f4c5c6174614470
706d65545c6c6163
4d4f445245535500
4f4149583d4e4941
52455355004e4958
525f4e49414d4f44
5250474e494d414f
49583d454c49464f
5355004e49584f41
773d454d414e5245
```

反之，则是这样子的

```c
00000148cafa24c0
00000148cafa24e0
00000148cafa2500
00000148cafa2520
00000148cafa2540
00000148cafa2560
00000148cafa2580
00000148cafa25a0
00000148cafa25c0
00000148cafa25e0
00000148cafa2600
00000148cafa2620
00000148cafa2640
00000148cafa2660
00000148cafa6770
00000148cafa6810
00000148cafa66b0
00000148cafa66d0
```



### 综上，以下是一个完整的过程（仅含board）

```c
  char **board = malloc(sizeof(char) * 8 * 20);  
  if (res == NULL || snake == NULL || board == NULL)
    return NULL;
  char *t_board[19] = {
      "####################",
      "#                  #",
      "# d>D    *         #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "#                  #",
      "####################"};
  for (int i = 0; i < 18; i++)
  {
    board[i] = malloc(sizeof(char) * 22); // 这里注意要加上零字符
    if (board[i] == NULL)
      return NULL;
    strcpy(board[i], t_board[i]);
  }

  res->board = board;
```

### 还没完呢

当我风平浪静做到task5的时候，发现上述做法会产生矛盾，即关于换行符的矛盾，详见task3。

## Task 2: `free_state`

理解了第一题，才能准确地做第二题，主要就是逐项地free。

```c
/* Task 2 */
void free_state(game_state_t *state)
{
  // TODO: Implement this function.
  free(state->snakes);
  for (int i = 0; i < state->num_rows; i ++ ){
    free(state->board[i]);
  }
  free(state->board);
  free(state);
  return;
}
```

## Task 3: `print_board`

使用`fprintf`即可，但是要注意可能有两种方法。即输不输出换行符。

这里测试时，打印的是第一问初始化的board，初始化时一开始没有换行符，所以用了`fprintf(fp,"%s\n" ,state->board[i]);`

但是后面会从文件中读取board，这是自带换行符的，所以我们在task1中初始化时应该添加换行符，这一问中不添加。

```c
/* Task 3 */
void print_board(game_state_t *state, FILE *fp)
{
  // TODO: Implement this function.
  for (int i = 0; i < state->num_rows; i ++ ){
    fprintf(fp,"%s" ,state->board[i]);
  }
  return;
}
```

```c
  char *t_board[19] = {
      "####################\n",
      "#                  #\n",
      "# d>D    *         #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "#                  #\n",
      "####################\n"};
```



## Task 4: `update_state`

4.1 的 helper_function 比较简单，用数组+循环的判断。

### Task 4.3: `update_head`

这里注意，要用指针的方式修改结构体，否则修改的只是它的复制。

![image-20220908103409983](https://s2.loli.net/2022/09/09/eDqHbNhi64XMJAg.png)

错误：

```c
static void update_head(game_state_t *state, unsigned int snum)
{
  snake_t snake = state->snakes[snum];
  unsigned int r = snake.head_row, c = snake.head_col;
  char head = get_board_at(state, r, c);
  unsigned int nr = get_next_row(r, head), nc = get_next_col(c, head);
    
  snake.head_row = nr, snake.head_col = nc;
    
  set_board_at(state, r, c, head_to_body(head));
  set_board_at(state, nr, nc, head);
  return;
}
```

正确：

```c
static void update_head(game_state_t *state, unsigned int snum)
{
  snake_t *snake = &state->snakes[snum];
  unsigned int r = snake->head_row, c = snake->head_col;
  char head = get_board_at(state, r, c);
  unsigned int nr = get_next_row(r, head), nc = get_next_col(c, head);

  snake->head_row = nr, snake->head_col = nc;

  set_board_at(state, r, c, head_to_body(head));
  set_board_at(state, nr, nc, head);
  return;
}
```

### Task 4.5: `update_state`

根据提示，在吃到食物时，只`update_head`，而不`update_tail`，感觉很巧妙。

```C
void update_state(game_state_t *state, int (*add_food)(game_state_t *state))
{
  for (unsigned int snum = 0; snum < state->num_snakes; snum ++ ){
    snake_t *snake = &(state->snakes[snum]);
    char ne = next_square(state, snum);

    if (ne == ' '){
      update_head(state, snum);
      update_tail(state, snum);
    }
    else if (ne == '*'){
      update_head(state, snum);
      add_food(state);
    }
    else{
      snake->live = false;
      set_board_at(state, snake->head_row, snake->head_col, 'x');
    }
  }
  return;
}
```

## Task 5: `load_board`

这一问是比较复杂的，涉及的操作比较多，一点一点来。

```c
/* Task 5 */
game_state_t *load_board(char *filename)
{
  // 读取文件
  FILE *ptr;
  ptr = fopen(filename, "r");   // BUG: 把"r"写成了'r'
  if (ptr == NULL) return NULL;

  // 声明并定义待返回结果
  game_state_t *res = malloc(sizeof(game_state_t));
  if (res == NULL) return NULL;

  // 声明并定义数组，方便输入。这些内容都存储在栈中。
  char str[100010];     // 暂存输入的字符串；地图的长宽最大都可以达到10^5，所以之前只开 100 是不够的
  char *tep[100010];    // 暂存分配好的字符串，或者说暂存输入的board，因为不确定要输入多少行
  unsigned int cnt = 0; // 记录输入的字符串个数，即行数

  // 输入数据，分配堆，转移数据，保存位置
  while (fgets(str, 100010, ptr) != NULL){
    char *line = malloc(strlen(str) + 1); // 留一个位给空字符
    if (line == NULL) return NULL;

    strcpy(line, str);
    tep[cnt ++ ] = line;
  }
  
  // 声明定义我们要保存（返回）的board，存储cnt个字符指针，然后把tep的内容转移过来。
  char **board = malloc(sizeof(char) * 8 * cnt);
  for (int i = 0; i < cnt; i ++ ){
    board[i] = tep[i];
  }

  fclose(ptr);  // 记得关掉噢

  res->num_rows = cnt;
  res->board = board;

  return res;
}
```



另外记录一个难以发现的bug：

错误：

```c
    FILE *ptr;
    ptr = fopen("tests/01-simple-in.snk", 'r');
    if (ptr == NULL)
```

![image-20220908160551979](https://s2.loli.net/2022/09/09/NL9dy6zPlDwY5hR.png)

正确：

```c
    FILE *ptr;
    ptr = fopen("tests/01-simple-in.snk", "r");
    if (ptr == NULL)
```

之前写的是python，单引号双引号常常混用，c语言里可得分清楚了，单引号括起来的字符相当于整数。

## Task 6: `initialize_snake`

感觉理解不难，就是要精心雕琢一些细节，理清楚逻辑。

```c
/*
  Task 6.1

  Helper function for initialize_snakes.
  Given a snake struct with the tail row and col filled in,
  trace through the board to find the head row and col, and
  fill in the head row and col in the struct.
*/
static void find_head(game_state_t *state, unsigned int snum)
{
  // 不断更新尾巴的下一个字符，直到找到头
  snake_t *snake = &(state->snakes[snum]);
  unsigned int ansR = snake->tail_row, ansC = snake->tail_col;
  char cur = get_board_at(state, ansR, ansC);

  while(!is_head(cur)){
    ansR = get_next_row(ansR, cur), ansC = get_next_col(ansC, cur);
    cur = get_board_at(state, ansR, ansC);
  }

  snake->head_col = ansC, snake->head_row = ansR;

  return;
}

/* Task 6.2 */
game_state_t *initialize_snakes(game_state_t *state)
{
  unsigned int r = state->num_rows, cnt = 0;

  // 先确定cnt才能确定要容纳多少条蛇
  for (int i = 0; i < r; i ++ ){
    for (int j = 0; ; j ++ ){
      char c = state->board[i][j];
      if (c == '\0') break;
      if (is_tail(c)) cnt ++;
    }
  }

  snake_t *snake = malloc(sizeof(snake_t) * cnt);
  state->snakes = snake;
  state->num_snakes = cnt;

  // 遍历board，找到尾巴就进行操作
  cnt = 0;  // BUG: 漏了重置cnt，导致段错误
  for (unsigned int i = 0; i < r; i ++ ){
    for (unsigned int j = 0; ; j ++ ){
      char c = state->board[i][j];
      if (c == '\0') break;
      if (is_tail(c)){
        snake[cnt].tail_row = i;
        snake[cnt].tail_col = j;
        snake[cnt].live = true;
        cnt ++ ;
      }
    }
  }

  for (unsigned int i = 0; i < cnt; i ++ ){
    find_head(state, i);  // BUG: 把 i 写成 cnt...
  }
  return state;
}
```

