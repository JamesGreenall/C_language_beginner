# C21. break 与 continue

## 节点定位

- 所属模块：控制流程
- 学习难度：基础
- 建议学习时间：30-40 分钟
- 典型用途：在循环或 `switch` 中提前改变当前执行流程。

## 学习目标

- 能说明 `break` 和 `continue` 分别解决什么问题。
- 能理解 `break` 会结束当前循环或 `switch`。
- 能理解 `continue` 会跳过本轮循环剩余语句，进入下一轮。
- 能区分 `break`、`continue` 和普通条件判断的作用。
- 能说明嵌套循环中它们只影响最近的一层结构。
- 能避免因 `continue` 跳过状态更新而造成死循环。

## 初学者导读

前面已经学习了 `while`、`for`、`do while` 和 `switch`。这些结构通常按照固定规则执行：

- 循环会根据条件反复执行。
- `switch` 会根据固定值进入对应 `case`。

但有时程序需要更灵活：

- 找到目标后，立刻停止循环。
- 遇到不需要处理的数据，跳过本轮。
- 在 `switch` 的某个 `case` 执行完后，立即离开 `switch`。

这时就会用到 `break` 和 `continue`。

需要注意：它们会改变正常执行流程，因此应在逻辑清楚的情况下使用。初学阶段不要为了“显得高级”而滥用。

## 关键术语说明

- `break`：立即结束当前循环或当前 `switch`。
- `continue`：跳过本轮循环后面的语句，进入下一轮循环。
- 当前层：离 `break` 或 `continue` 最近的那一层循环或 `switch`。
- 提前结束：在循环条件自然变假之前结束循环。
- 跳过本轮：当前这一轮不再执行剩余语句，但循环本身还可能继续。
- 贯穿执行：`switch` 中缺少 `break` 时继续执行后续 `case` 的现象。
- 可读性：代码是否容易被人顺着流程理解。

## 学习思路

学习本知识点时，建议先记住一句话：

- `break` 是“这层结构到此为止”。
- `continue` 是“本轮到此为止，下一轮继续”。

然后分别放到三种场景中理解：

1. 在 `switch` 中使用 `break`。
2. 在循环中使用 `break`。
3. 在循环中使用 `continue`。

## 核心概念

`break` 和 `continue` 都是流程控制语句，但影响范围不同。

| 语句 | 可用位置 | 作用 |
|---|---|---|
| `break` | 循环、`switch` | 结束当前循环或当前 `switch` |
| `continue` | 循环 | 跳过本轮循环剩余部分，进入下一轮 |

`continue` 不能用于普通 `if`，也不能用于 `switch` 本身。它必须位于循环结构中。

## break 在 switch 中的作用

在 `switch` 中，`break` 用于结束当前 `switch`。

```c
switch (choice) {
case 1:
    printf("start\n");
    break;
case 2:
    printf("settings\n");
    break;
default:
    printf("unknown\n");
    break;
}
```

如果 `choice` 是 1，程序会执行 `case 1` 中的语句，然后遇到 `break`，离开整个 `switch`。

如果不写 `break`，程序会继续执行后面的 `case`，这就是 C17 中提到的贯穿执行。

## break 在循环中的作用

在循环中，`break` 用于提前结束循环。

```c
for (int i = 1; i <= 10; i++) {
    if (i == 6) {
        break;
    }
    printf("%d\n", i);
}
```

输出结果：

```text
1
2
3
4
5
```

当 `i` 等于 6 时，执行 `break`，整个 `for` 循环立即结束，后面的 6 到 10 都不会输出。

## break 的典型用途

### 1. 找到目标后停止查找

```c
int target = 7;
int found = 0;

for (int i = 1; i <= 10; i++) {
    if (i == target) {
        found = 1;
        break;
    }
}
```

找到目标后继续循环没有必要，因此可以用 `break` 提前结束。

### 2. 输入结束标记后停止

```c
int x;

while (1) {
    scanf("%d", &x);

    if (x == 0) {
        break;
    }

    printf("input = %d\n", x);
}
```

这里 `while (1)` 表示条件恒为真，循环本身不会自然结束。程序依靠输入 0 时的 `break` 退出循环。

这种写法可以使用，但初学阶段要保证退出条件清楚。

### 3. 遇到错误时停止处理

```c
for (int i = 1; i <= 5; i++) {
    int score;
    scanf("%d", &score);

    if (score < 0 || score > 100) {
        printf("invalid score\n");
        break;
    }

    printf("score = %d\n", score);
}
```

如果输入非法分数，直接停止后续处理。

## continue 的作用

`continue` 用于跳过本轮循环剩余语句，进入下一轮循环。

```c
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        continue;
    }
    printf("%d\n", i);
}
```

输出结果：

```text
1
3
5
7
9
```

当 `i` 是偶数时，执行 `continue`，后面的 `printf` 被跳过。循环继续进入下一轮。

## continue 的典型用途

### 1. 跳过不需要处理的数据

```c
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        continue;
    }
    printf("%d\n", i);
}
```

偶数不处理，奇数才输出。

### 2. 跳过非法输入

```c
for (int i = 1; i <= 5; i++) {
    int score;
    scanf("%d", &score);

    if (score < 0 || score > 100) {
        printf("invalid, skipped\n");
        continue;
    }

    printf("valid score = %d\n", score);
}
```

非法分数不参与后续处理，但循环继续读取下一次输入。

### 3. 减少嵌套层次

有时 `continue` 可以减少一层 `else`。

不用 `continue`：

```c
for (int i = 1; i <= 10; i++) {
    if (i % 2 != 0) {
        printf("%d\n", i);
    }
}
```

使用 `continue`：

```c
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        continue;
    }
    printf("%d\n", i);
}
```

两种写法都可以。初学阶段应优先选择自己更容易解释清楚的写法。

## break 与 continue 的区别

| 对比点 | `break` | `continue` |
|---|---|---|
| 作用 | 结束当前循环或 `switch` | 跳过本轮循环剩余语句 |
| 循环是否继续 | 不继续当前循环 | 可能继续下一轮 |
| 可用于 `switch` | 可以 | 不可以单独用于 `switch` |
| 常见用途 | 找到目标后停止、菜单退出 | 跳过不需要处理的数据 |

示例对比：

```c
for (int i = 1; i <= 5; i++) {
    if (i == 3) {
        break;
    }
    printf("%d\n", i);
}
```

输出：

```text
1
2
```

如果改成：

```c
for (int i = 1; i <= 5; i++) {
    if (i == 3) {
        continue;
    }
    printf("%d\n", i);
}
```

输出：

```text
1
2
4
5
```

`break` 是结束整个循环；`continue` 只是跳过 `i == 3` 这一轮。

## 在 while 中使用 continue 的风险

在 `for` 循环中，执行 `continue` 后会先执行更新表达式，再判断下一轮条件。

但在 `while` 循环中，`continue` 会直接回到条件判断。如果状态更新写在 `continue` 后面，可能被跳过。

错误示例：

```c
int i = 1;

while (i <= 10) {
    if (i % 2 == 0) {
        continue;
    }

    printf("%d\n", i);
    i++;
}
```

当 `i` 变成 2 时，执行 `continue`，后面的 `i++` 被跳过，`i` 一直是 2，循环无法结束。

修正方式：

```c
int i = 1;

while (i <= 10) {
    if (i % 2 == 0) {
        i++;
        continue;
    }

    printf("%d\n", i);
    i++;
}
```

或者重新组织逻辑，避免在 `continue` 前漏掉状态更新。

## 嵌套循环中的影响范围

如果循环里面还有循环，`break` 和 `continue` 只影响离它最近的一层循环。

```c
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        if (j == 2) {
            break;
        }
        printf("i=%d, j=%d\n", i, j);
    }
}
```

这里的 `break` 只结束内层 `j` 循环，不会结束外层 `i` 循环。

输出中每一轮外层循环仍然会继续，只是内层在 `j == 2` 时提前结束。

如果确实要结束多层循环，通常需要使用标志变量或把逻辑封装成函数后用 `return`，这些内容会在后续知识点中逐步学习。

## 使用建议

`break` 和 `continue` 很有用，但过度使用会让代码流程跳来跳去，难以阅读。

建议：

- 一个循环中不要随意写多个 `break` 和 `continue`。
- 使用前先问清楚：是要结束整个循环，还是只跳过本轮？
- 对于简单条件，普通 `if/else` 可能更直观。
- 在 `while` 中使用 `continue` 时，特别检查状态更新是否会被跳过。

## 示例

```c
#include <stdio.h>

int main(void) {
    for (int i = 1; i <= 10; i++) {
        if (i == 8) {
            break;
        }

        if (i % 2 == 0) {
            continue;
        }

        printf("%d\n", i);
    }

    return 0;
}
```

## 代码逐行解释

- `for (int i = 1; i <= 10; i++)`：让 `i` 从 1 变化到 10。
- `if (i == 8)`：当 `i` 为 8 时准备提前结束循环。
- `break;`：跳出当前 `for` 循环。
- `if (i % 2 == 0)`：判断 `i` 是否为偶数。
- `continue;`：偶数跳过本轮后面的输出。
- `printf("%d\n", i);`：只输出没有被跳过的奇数。

实际输出为：

```text
1
3
5
7
```

## 前置知识

- C17 switch 分支
- C18 while 循环
- C19 for 循环
- C20 do while 循环

## 关联知识点

- C13 语句
- C16 if/else 分支
- C22 函数定义与调用

## 常见错误

- 以为 `break` 会跳出所有嵌套循环。
- 在 `while` 中使用 `continue` 时跳过了状态更新，导致死循环。
- 把 `break` 和 `continue` 的作用混淆。
- 在普通 `if` 中单独使用 `continue`，但外层没有循环。
- 在本来可以清楚写成 `if/else` 的地方过度使用 `continue`。
- `switch` 中忘记 `break`，导致意外贯穿执行。

## 初学者常见误区

- `break` 是结束当前循环或 `switch`，不是结束整个程序。
- `continue` 不会结束循环，只是跳过当前这一轮。
- `continue` 不能用于没有循环包围的代码。
- 嵌套循环中，`break` 和 `continue` 默认只影响最近的一层。
- 在 `for` 和 `while` 中使用 `continue` 时，状态更新的位置不同，需要特别注意。

## 语法格式

```c
break;
continue;
```

在循环中的常见形式：

```c
for (初始化; 条件; 更新) {
    if (提前结束条件) {
        break;
    }

    if (跳过本轮条件) {
        continue;
    }

    正常处理语句;
}
```

在 `switch` 中的常见形式：

```c
switch (表达式) {
case 常量:
    语句;
    break;
default:
    语句;
    break;
}
```

## 与图谱的关系

### 直接前置或输入节点

- 前置依赖：C17 switch 分支
- 前置依赖：C18 while 循环
- 前置依赖：C19 for 循环
- 前置依赖：C20 do while 循环

### 后续影响或输出节点

- 支撑节点：C22 函数定义与调用
- 关联关系：C45 小型程序设计

## 练习题

1. 用 `break` 写一个循环，输出 1 到 10，但遇到 6 时停止。
2. 用 `continue` 输出 1 到 10 中的奇数。
3. 说明 `break` 和 `continue` 的区别。
4. 写一个 `switch`，每个 `case` 后都使用 `break`。
5. 分析嵌套循环中内层 `break` 会跳出哪一层。

## 分层练习

1. 基础：循环输出 1 到 10，遇到 5 时 `break`。
2. 基础：循环输出 1 到 10，跳过 5。
3. 提升：不断输入整数，输入 0 时用 `break` 结束。
4. 提升：输入 5 个分数，非法分数用 `continue` 跳过。
5. 进阶：在嵌套循环中使用 `break`，观察它只结束内层循环。
6. 思考：什么时候使用 `continue` 会让代码更清楚？什么时候会让代码更难读？

## 小结

`break` 和 `continue` 都会改变正常执行流程。`break` 用于结束当前循环或 `switch`，`continue` 用于跳过本轮循环剩余语句。学习时要重点关注它们的影响范围，尤其是在 `while` 和嵌套循环中的行为。
