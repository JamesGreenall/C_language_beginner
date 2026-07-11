# C35. 动态内存 malloc/free

## 节点定位

- 所属模块：内存与指针
- 学习难度：重点进阶
- 建议学习时间：120-150 分钟
- 典型用途：在程序运行期间根据实际需要申请、调整和释放内存。

## 学习目标

- 理解固定长度数组的局限，以及动态内存解决的问题。
- 会使用 `malloc` 申请空间，并检查申请是否成功。
- 会使用 `free` 释放空间，理解内存泄漏、悬空指针和重复释放。
- 理解 `calloc` 与 `malloc` 的差别。
- 掌握 `realloc` 的正确接收方式及失败处理。
- 能把元素个数、元素类型、字节数和访问边界联系起来。

## 一、为什么需要动态内存

普通数组的长度通常要在定义时确定：

```c
int scores[30];
```

如果程序运行前无法知道学生人数，就会遇到两种问题：

- 数组开得太小，无法保存全部数据；
- 数组开得过大，长期占用不需要的空间。

动态内存允许程序先读取实际数量，再按需要申请空间：

```text
读取元素个数 -> 计算所需字节数 -> 申请内存 -> 使用 -> 释放
```

动态内存提供了灵活性，同时把管理责任交给程序员。每一块成功申请的空间都必须明确由谁使用、何时释放。

## 二、自动存储与动态存储的基本区别

函数中的普通局部变量通常具有自动存储期：

```c
void example(void) {
    int value = 10;
    int arr[5];
}
```

函数执行结束时，这些对象的生命周期自动结束。

动态内存则由程序显式申请和释放：

```c
int *p = malloc(5 * sizeof *p);
free(p);
```

它不会因为申请它的函数返回就自动释放。若程序丢失了这块内存的地址，又没有调用 `free`，空间会一直占用到程序结束，这就是内存泄漏。

动态申请的区域通常称为堆。学习初期应重点掌握其生命周期规则，不必依赖某种具体操作系统的内存布局图。

## 三、malloc 的基本用法

`malloc` 声明在 `<stdlib.h>` 中，基本形式为：

```c
指针 = malloc(所需字节数);
```

例如，申请能够保存 5 个 `int` 的空间：

```c
#include <stdlib.h>

int *numbers = malloc(5 * sizeof *numbers);
```

表达式可以分解为：

- `sizeof *numbers`：一个 `int` 元素占用的字节数；
- `5 * sizeof *numbers`：5 个元素需要的总字节数；
- `malloc(...)`：申请至少这么多连续字节；
- `numbers`：保存申请所得空间的起始地址。

在 C 语言中，不需要把 `malloc` 的返回值强制转换为 `int *`。正确包含 `<stdlib.h>` 后，`void *` 可以自动转换为对象指针类型。

## 四、malloc 可能失败

当系统无法提供所需空间时，`malloc` 返回空指针。因此必须检查：

```c
int *numbers = malloc(5 * sizeof *numbers);

if (numbers == NULL) {
    printf("内存申请失败\n");
    return 1;
}
```

只有确认 `numbers != NULL` 后，才能访问 `numbers[0]` 等元素。先使用后检查没有意义。

## 五、malloc 得到的内容未初始化

`malloc` 只提供指定大小的空间，不保证其中原有字节是什么：

```c
int *numbers = malloc(5 * sizeof *numbers);
```

此时不能直接读取 `numbers[0]`。必须先赋值：

```c
for (int i = 0; i < 5; i++) {
    numbers[i] = 0;
}
```

读取尚未初始化的动态对象会得到不确定结果，并可能引发未定义行为。不要把某次运行中碰巧出现的 0 当作保证。

## 六、动态内存可以像数组一样使用

成功申请多个连续同类型元素的空间后，可以使用下标访问：

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int count = 5;
    int *numbers = malloc((size_t)count * sizeof *numbers);

    if (numbers == NULL) {
        printf("内存申请失败\n");
        return 1;
    }

    for (int i = 0; i < count; i++) {
        numbers[i] = (i + 1) * 10;
    }

    for (int i = 0; i < count; i++) {
        printf("%d ", numbers[i]);
    }
    printf("\n");

    free(numbers);
    numbers = NULL;

    return 0;
}
```

`numbers` 是指针，不是数组对象，但它指向一块能容纳多个连续 `int` 的空间，因此可以用数组下标语法访问。

## 七、free 的作用

`free` 把动态申请的空间归还给内存管理系统：

```c
free(numbers);
```

释放后必须明确以下事实：

- 该动态对象的生命周期已经结束；
- 原空间中的数据不能再访问；
- 指针变量 `numbers` 本身仍然存在；
- `numbers` 中可能还保留旧地址，但旧地址已经无效。

这种仍保存失效地址的指针称为悬空指针。常见做法是在释放后立即置空：

```c
free(numbers);
numbers = NULL;
```

置空不能修复其他仍指向同一空间的指针，但可以减少当前指针被误用的风险。

## 八、内存泄漏、释放后使用和重复释放

### 1. 内存泄漏

```c
int *p = malloc(100 * sizeof *p);
p = NULL;  // 原地址丢失，无法再 free
```

申请的空间仍被占用，但程序已经找不到它。长时间运行或反复发生泄漏时，程序占用内存会不断增加。

### 2. 释放后继续使用

```c
free(p);
printf("%d\n", p[0]);  // 未定义行为
```

释放后的数据不再属于程序，不能读取或写入。

### 3. 重复释放

```c
free(p);
free(p);  // 未定义行为
```

同一块动态内存只能释放一次。若释放后执行 `p = NULL`，再次 `free(p)` 本身是允许的，因为 `free(NULL)` 不执行任何操作；但这不表示可以忽视所有权管理。

## 九、calloc：申请并清零

`calloc` 接收元素个数和每个元素的字节数：

```c
int *numbers = calloc(5, sizeof *numbers);
```

它申请能够容纳 5 个元素的空间，并把所有字节初始化为零。对于常见整数类型，这意味着初始整数值为 0。

比较：

| 函数 | 参数形式 | 初始内容 |
| --- | --- | --- |
| `malloc` | 总字节数 | 未初始化 |
| `calloc` | 元素个数、单个元素大小 | 所有字节为零 |

两者都可能失败，也都必须用 `free` 释放。

## 十、realloc：调整空间大小

当原有空间不足时，可以使用 `realloc` 调整大小：

```c
int *new_numbers = realloc(numbers, 10 * sizeof *numbers);
```

`realloc` 可能：

- 在原位置扩展空间；
- 申请新空间，把原数据复制过去，再释放旧空间；
- 因申请失败返回 `NULL`，同时原空间仍然有效。

因此不要直接覆盖原指针：

```c
numbers = realloc(numbers, new_size);  // 不推荐
```

如果失败，`numbers` 会被改成 `NULL`，原地址丢失，造成内存泄漏。应使用临时指针：

```c
int *temp = realloc(numbers, 10 * sizeof *numbers);

if (temp == NULL) {
    printf("调整内存失败\n");
    free(numbers);
    numbers = NULL;
    return 1;
}

numbers = temp;
```

扩展后，原有元素的值会保留，但新增部分未初始化，使用前要赋值。

## 十一、完整示例：根据输入动态创建数组

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int count;

    printf("请输入成绩数量：");
    if (scanf("%d", &count) != 1 || count <= 0) {
        printf("数量必须是正整数\n");
        return 1;
    }

    int *scores = malloc((size_t)count * sizeof *scores);
    if (scores == NULL) {
        printf("内存申请失败\n");
        return 1;
    }

    int sum = 0;

    for (int i = 0; i < count; i++) {
        printf("请输入第 %d 个成绩：", i + 1);
        if (scanf("%d", &scores[i]) != 1) {
            printf("输入无效\n");
            free(scores);
            scores = NULL;
            return 1;
        }
        sum += scores[i];
    }

    printf("平均分：%.2f\n", (double)sum / count);

    free(scores);
    scores = NULL;
    return 0;
}
```

这个程序体现了完整生命周期：

1. 验证元素个数；
2. 计算并申请所需空间；
3. 检查申请结果；
4. 在有效范围内使用空间；
5. 正常结束或提前返回前都释放空间。

## 十二、元素个数与字节数的安全问题

假设要申请 `count` 个元素：

```c
malloc((size_t)count * sizeof *numbers)
```

程序必须先确保 `count` 是合理的正数。如果负数直接转换为无符号的 `size_t`，可能变成极大的数。

此外，两个很大的数相乘还可能超出 `size_t` 的表示范围。较严谨的检查方式是：

```c
#include <stdint.h>
#include <stdlib.h>

if ((size_t)count > SIZE_MAX / sizeof *numbers) {
    printf("申请规模过大\n");
    return 1;
}
```

对于普通入门练习，通常会人为限制 `count` 的最大值，例如不超过 10000。理解上述风险有助于认识：动态内存安全不仅是检查 `NULL`，还包括申请大小计算正确。

## 十三、零字节申请应如何处理

`malloc(0)` 和 `calloc(0, size)` 的结果具有实现相关特点，返回值可能是 `NULL`，也可能是一个不能解引用但可传给 `free` 的指针。

因此，初学程序应先规定元素个数必须大于 0，不依赖零字节申请的具体表现：

```c
if (count <= 0) {
    return 1;
}
```

## 十四、谁负责释放内存

动态内存需要明确所有权，也就是“哪个部分负责最终调用 `free`”。常见规则包括：

- 谁申请，谁释放；
- 函数申请并返回地址，同时在说明中明确调用者负责释放；
- 对象交给另一个模块管理后，原持有者不再释放。

初学阶段采用“谁申请，谁释放”最容易检查。但当函数返回动态内存时，调用者必须承担释放责任：

```c
int *create_array(int count) {
    if (count <= 0) {
        return NULL;
    }
    return malloc((size_t)count * sizeof(int));
}
```

调用者使用结束后必须 `free` 返回的地址。

## 十五、动态内存与 sizeof

```c
int *numbers = malloc(10 * sizeof *numbers);
printf("%zu\n", sizeof numbers);
```

`sizeof numbers` 得到的是指针变量大小，不是已申请空间的大小。C 语言指针不会自动记录“后面有多少个元素”。程序必须使用额外变量保存长度：

```c
int count = 10;
```

因此，动态数组通常至少需要成对管理：

```text
起始地址 numbers + 元素个数 count
```

## 十六、常见错误检查表

### 申请前

- 元素个数是否大于 0；
- 元素个数是否有合理上限；
- 字节数乘法是否可能溢出。

### 申请后

- 是否检查返回值为 `NULL`；
- 是否在读取前初始化数据；
- 所有下标是否在 `0` 到 `count - 1` 之间。

### 释放时

- 是否只释放 `malloc`、`calloc` 或 `realloc` 得到的动态地址；
- 是否只释放一次；
- 是否还有其他指针继续使用该空间；
- 正常路径和错误返回路径是否都能释放。

## 十七、不能传给 free 的地址

以下地址不能传给 `free`：

```c
int local = 10;
int arr[5];

free(&local);  // 错误：局部变量不是动态申请的
free(arr);     // 错误：普通数组不是动态申请的
```

此外，也不能释放动态块中间某个元素的地址：

```c
int *p = malloc(5 * sizeof *p);
free(p + 1);  // 错误
free(p);      // 应释放最初返回的起始地址
```

传给 `free` 的参数必须是尚未释放的动态内存起始地址，或者 `NULL`。

## 十八、与相关知识的联系

- C10 作用域与生命周期：指针变量离开作用域，不等于其所指动态内存自动释放。
- C31 地址与内存：动态内存函数返回所申请空间的起始地址。
- C32 指针基础：必须用指针保存和访问动态空间。
- C33 指针与数组：连续动态空间可通过下标或指针运算访问。
- C34 指针与函数：函数可以接收、返回或通过参数修改动态内存指针。
- C44 常见未定义行为：越界、释放后使用、重复释放都可能造成未定义行为。

## 分层练习

1. 动态申请 5 个 `int`，依次赋值为 1 到 5，输出后释放。
2. 读取正整数 `count`，使用 `calloc` 创建数组，验证初始元素是否为 0。
3. 动态保存若干成绩，计算最大值、最小值和平均值。
4. 先申请 5 个整数，再用 `realloc` 扩展到 10 个；保留原数据并初始化新增部分。
5. 分别解释内存泄漏、悬空指针、重复释放和越界访问，并为每种问题写出修正方案。

## 小结

动态内存的核心不是记住几个函数名，而是管理一段明确的生命周期：根据有效规模计算字节数，申请后检查结果，初始化并在边界内使用，最后恰好释放一次。程序还必须保存元素个数，因为指针只记录地址，不记录动态空间的长度。
