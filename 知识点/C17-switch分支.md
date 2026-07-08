# C17. switch 分支

## 节点定位

- 所属模块：控制流程
- 学习难度：基础
- 建议学习时间：30-40 分钟
- 典型用途：处理一个表达式对应多个固定取值的分支场景。

## 学习目标

- 能说明 `switch` 适合解决什么问题。
- 能写出包含 `case`、`break`、`default` 的基本 `switch` 结构。
- 能理解 `case` 匹配的是固定值，而不是复杂条件。
- 能说明忘记 `break` 会导致什么结果。
- 能区分 `switch` 和 `if/else if` 的适用场景。
- 能使用 `switch` 处理菜单、编号、字符选项等问题。

## 初学者导读

C16 学习了 `if/else`，它可以处理各种条件判断，例如范围判断、大小比较、多个条件组合等。`switch` 也是分支结构，但它的适用范围更明确：当一个变量可能等于几个固定值时，`switch` 往往更清晰。

例如菜单程序中，用户输入：

- `1` 表示开始。
- `2` 表示设置。
- `3` 表示退出。

这种“根据一个固定编号选择不同操作”的场景，就很适合使用 `switch`。

## 关键术语说明

- `switch` 表达式：放在 `switch (...)` 中，用于和各个 `case` 比较的表达式。
- `case`：表示一种具体匹配情况。
- `break`：结束当前 `switch`，跳出整个分支结构。
- `default`：所有 `case` 都不匹配时执行的兜底分支。
- 贯穿执行：某个 `case` 执行后，如果没有 `break`，会继续执行后面的 `case`。
- 整型常量表达式：`case` 后面通常需要能在编译时确定的整数类常量，如 `1`、`'A'`、枚举常量。

## 学习思路

学习本知识点时，建议按照以下顺序理解：

1. 先明确 `switch` 用于“固定值匹配”。
2. 再掌握 `switch`、`case`、`break`、`default` 的结构。
3. 通过示例观察匹配成功后会执行哪一段代码。
4. 重点理解忘记 `break` 会造成贯穿执行。
5. 最后比较 `switch` 与 `if/else if` 的区别。

## 核心概念

`switch` 会计算括号中的表达式，然后把结果依次与各个 `case` 后的常量比较。匹配成功后，从对应的 `case` 开始执行。

基本形式：

```c
switch (表达式) {
case 常量1:
    语句;
    break;
case 常量2:
    语句;
    break;
default:
    语句;
    break;
}
```

`switch` 不是用来判断复杂范围的。它更适合判断“等于哪一个固定值”。

## 最小示例

```c
int choice = 1;

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

如果 `choice` 的值是 1，就会匹配 `case 1`，输出 `start`，然后遇到 `break` 跳出 `switch`。

## switch 的执行过程

以如下代码为例：

```c
switch (choice) {
case 1:
    printf("start\n");
    break;
case 2:
    printf("settings\n");
    break;
case 3:
    printf("exit\n");
    break;
default:
    printf("unknown\n");
    break;
}
```

执行过程可以分为四步：

1. 先计算 `choice` 的值。
2. 将这个值与 `case 1`、`case 2`、`case 3` 依次比较。
3. 如果找到相等的 `case`，从该位置开始执行。
4. 遇到 `break` 后跳出整个 `switch`。

如果没有任何 `case` 匹配，就执行 `default`。

## case 后面写什么

`case` 后面通常写固定值，而不是条件表达式。

可以写：

```c
case 1:
case 2:
case 'A':
case 'q':
```

不应写：

```c
case score >= 60: // 错误
```

`switch` 不是这样判断范围的。如果要判断分数是否大于等于 60，应该使用 `if/else`。

```c
if (score >= 60) {
    printf("pass\n");
}
```

## break 的作用

`break` 用于结束当前 `switch`。

```c
case 1:
    printf("start\n");
    break;
```

如果没有 `break`，程序不会自动停在当前 `case`，而是会继续向下执行后面的语句。

## 忘记 break 的结果

示例：

```c
int choice = 1;

switch (choice) {
case 1:
    printf("start\n");
case 2:
    printf("settings\n");
default:
    printf("unknown\n");
}
```

如果 `choice` 是 1，输出结果会是：

```text
start
settings
unknown
```

原因是：程序从 `case 1` 开始执行后，没有遇到 `break`，于是继续执行后面的 `case 2` 和 `default`。

这种现象称为贯穿执行。多数初学场景中，贯穿执行是错误，因此建议每个 `case` 末尾写 `break`。

## 有意使用贯穿执行

有时可以故意让多个 `case` 共用同一段代码。

```c
switch (grade) {
case 'A':
case 'B':
case 'C':
    printf("pass\n");
    break;
case 'D':
case 'F':
    printf("fail\n");
    break;
default:
    printf("unknown grade\n");
    break;
}
```

这里 `A`、`B`、`C` 都表示通过，因此它们共用同一个输出语句。这样的贯穿执行是有意设计的。

如果有意省略 `break`，建议在代码旁边加清楚注释，避免被误认为错误。

## default 的作用

`default` 用于处理所有未匹配的情况。

```c
default:
    printf("unknown choice\n");
    break;
```

它类似于 `if/else` 中最后的 `else`，用于兜底。

建议保留 `default`，因为用户输入可能不符合预期。例如菜单只允许输入 1、2、3，但用户可能输入 9。

`default` 不一定必须写在最后，但初学阶段建议写在最后，结构更清楚。

## switch 可以匹配字符

`switch` 不只能处理数字，也常用于处理字符选项。

```c
char op = '+';

switch (op) {
case '+':
    printf("add\n");
    break;
case '-':
    printf("subtract\n");
    break;
case '*':
    printf("multiply\n");
    break;
case '/':
    printf("divide\n");
    break;
default:
    printf("unknown operator\n");
    break;
}
```

字符在 C 语言底层也是整数编码，因此可以用于 `switch`。

## switch 与 if/else 的区别

`switch` 和 `if/else` 都能处理分支，但适用场景不同。

| 场景 | 更适合使用 |
|---|---|
| 判断是否大于 60 | `if/else` |
| 判断是否在 0 到 100 之间 | `if/else` |
| 判断多个复杂条件组合 | `if/else` |
| 根据菜单编号 1、2、3 选择操作 | `switch` |
| 根据字符 `+`、`-`、`*`、`/` 选择操作 | `switch` |
| 根据枚举值选择状态处理 | `switch` |

简单记忆：

- 条件是“范围、大小、复杂逻辑”时，优先考虑 `if/else`。
- 条件是“等于某个固定值”时，可以考虑 `switch`。

## 示例

```c
#include <stdio.h>

int main(void) {
    int choice;

    printf("1. Start\n");
    printf("2. Settings\n");
    printf("3. Exit\n");
    scanf("%d", &choice);

    switch (choice) {
    case 1:
        printf("Start selected\n");
        break;
    case 2:
        printf("Settings selected\n");
        break;
    case 3:
        printf("Exit selected\n");
        break;
    default:
        printf("Invalid choice\n");
        break;
    }

    return 0;
}
```

## 代码逐行解释

- `int choice;`：声明变量保存用户选择。
- 三个 `printf`：输出菜单选项。
- `scanf("%d", &choice);`：读取用户输入的编号。
- `switch (choice)`：根据 `choice` 的值选择分支。
- `case 1`、`case 2`、`case 3`：分别处理三个固定选项。
- `break`：执行完当前选项后跳出 `switch`。
- `default`：处理不是 1、2、3 的输入。

## 前置知识

- C13 语句
- C14 输入输出 printf/scanf
- C15 关系与逻辑表达式
- C16 if/else 分支

## 关联知识点

- C21 break 与 continue
- C37 枚举 enum

## 常见错误

- 忘记 `break`，导致贯穿执行。
- 用 `switch` 判断复杂范围，如 `score >= 60`。
- 忘记写 `default`，导致意外输入没有处理。
- 在不同 `case` 中重复写大量相同代码。
- 把 `case 1:` 后面的冒号写成分号。
- 在 `case` 后面写变量或运行时条件，而不是常量。

## 初学者常见误区

- `switch` 判断的是是否等于某个固定值，不是判断范围。
- `case` 后面要写冒号 `:`，不是分号。
- 匹配到某个 `case` 后，如果没有 `break`，程序会继续向下执行。
- `default` 不是必须语法，但通常建议写。
- 多个 `case` 可以共用同一段代码，但应当保证意图清楚。
- `switch` 不一定比 `if/else` 高级，只是适用场景不同。

## 语法格式

```c
switch (表达式) {
case 常量1:
    语句;
    break;
case 常量2:
    语句;
    break;
default:
    语句;
    break;
}
```

多个 `case` 共用同一段代码：

```c
switch (value) {
case 1:
case 2:
case 3:
    语句;
    break;
default:
    语句;
    break;
}
```

## 与图谱的关系

### 直接前置或输入节点

- 前置依赖：C13 语句
- 前置依赖：C14 输入输出 printf/scanf
- 关联关系：C15 关系与逻辑表达式
- 关联关系：C16 if/else 分支

### 后续影响或输出节点

- 前置依赖：C21 break 与 continue
- 关联关系：C37 枚举 enum

## 练习题

1. 写一个 `switch`，根据输入的 1、2、3 输出不同菜单文字。
2. 故意删除某个 `case` 后的 `break`，观察输出结果。
3. 用 `switch` 判断字符 `+`、`-`、`*`、`/`，输出对应运算名称。
4. 说明为什么 `switch` 不适合直接判断 `score >= 60`。
5. 解释 `default` 的作用。

## 分层练习

1. 基础：输入数字 1、2、3，分别输出 `Start`、`Settings`、`Exit`。
2. 基础：输入字符 `y` 或 `n`，输出 `yes` 或 `no`。
3. 提升：输入星期编号 1 到 7，输出对应星期名称。
4. 提升：让多个 `case` 共用代码，把 1 到 5 判断为工作日，6 到 7 判断为周末。
5. 进阶：写一个简单计算器菜单，根据输入的运算符选择加、减、乘、除。
6. 思考：如果同一个问题既能用 `if/else` 写，也能用 `switch` 写，应如何选择？

## 小结

`switch` 是处理固定值多分支的常用结构。学习时要重点理解 `case` 的匹配方式、`break` 的作用和 `default` 的兜底意义。掌握它之后，菜单程序、字符选项和状态处理会变得更清晰。
