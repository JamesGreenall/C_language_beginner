# C37. 枚举 enum

## 节点定位

- 所属模块：自定义类型
- 学习难度：综合应用
- 建议学习时间：60-90 分钟
- 典型用途：为一组有限且相关的整数状态命名，使条件判断和程序状态更清晰。

## 学习目标

- 理解枚举适合表示“有限选项集合”的原因。
- 会定义枚举类型、声明枚举变量并使用枚举成员。
- 理解枚举成员是有名称的整数常量，不是字符串。
- 掌握默认编号、显式赋值和后续值递增规则。
- 会结合 `switch` 处理枚举状态。
- 理解枚举能提升可读性，但不能自动阻止所有无效整数。

## 一、为什么需要枚举

假设程序用整数表示订单状态：

```c
int status = 2;
```

仅看这条语句，无法判断 2 表示“待付款”“已发货”还是“已完成”。程序中还可能出现许多不易理解的数字判断：

```c
if (status == 3) {
    /* 处理某种状态 */
}
```

这种没有直接说明含义的数字常被称为魔法数字。枚举可以为有限状态命名：

```c
enum OrderStatus {
    ORDER_PENDING,
    ORDER_PAID,
    ORDER_SHIPPED,
    ORDER_COMPLETED
};
```

现在可以写成：

```c
enum OrderStatus status = ORDER_SHIPPED;
```

代码直接表达“订单处于已发货状态”，不必记忆数字 2 的含义。

## 二、定义枚举类型

基本形式为：

```c
enum 枚举标签 {
    枚举成员1,
    枚举成员2,
    枚举成员3
};
```

例如：

```c
enum TrafficLight {
    LIGHT_RED,
    LIGHT_YELLOW,
    LIGHT_GREEN
};
```

其中：

- `enum` 是枚举关键字；
- `TrafficLight` 是枚举标签；
- `LIGHT_RED` 等名称是枚举成员，也称枚举常量；
- 定义末尾需要分号。

定义类型后，声明枚举变量：

```c
enum TrafficLight light = LIGHT_RED;
```

## 三、枚举成员实际是整数常量

枚举成员不是字符串。默认情况下，第一个成员的值为 0，后续成员依次加 1：

```c
enum TrafficLight {
    LIGHT_RED,     // 0
    LIGHT_YELLOW,  // 1
    LIGHT_GREEN    // 2
};
```

可以使用 `%d` 观察它们的整数值：

```c
printf("%d\n", LIGHT_RED);
printf("%d\n", LIGHT_YELLOW);
printf("%d\n", LIGHT_GREEN);
```

但实际程序应优先使用枚举成员名进行判断，而不是反过来依赖对应数字。

以下写法错误，因为枚举成员不是文本：

```c
// printf("%s\n", LIGHT_RED);  // 错误
```

如果需要显示中文或英文名称，应编写转换函数，后文会给出示例。

## 四、手动指定枚举值

可以显式指定成员对应的整数：

```c
enum HttpResult {
    RESULT_OK = 200,
    RESULT_NOT_FOUND = 404,
    RESULT_SERVER_ERROR = 500
};
```

也可以只指定部分成员。未指定的成员在前一个成员基础上加 1：

```c
enum Level {
    LEVEL_LOW = 1,
    LEVEL_MEDIUM,     // 2
    LEVEL_HIGH = 10,
    LEVEL_EXTREME     // 11
};
```

因此不能简单认为所有枚举都从 0 连续编号。阅读枚举定义时要检查是否存在显式赋值。

## 五、枚举值可以重复

C 语言允许不同枚举成员具有相同整数值：

```c
enum Result {
    RESULT_SUCCESS = 0,
    RESULT_OK = 0,
    RESULT_ERROR = 1
};
```

这可以为同一个值提供不同语义名称，但也可能造成判断和 `switch` 分支冲突。除非确有明确用途，否则初学阶段建议让每个成员使用不同值。

## 六、枚举成员名的作用范围

枚举成员名属于普通标识符命名空间。同一作用域中不能随意重复使用相同名称：

```c
enum First {
    STATUS_OK
};

// enum Second {
//     STATUS_OK  // 同一作用域内重名
// };
```

C 语言没有自动把成员名限制在枚举标签内部，所以常使用统一前缀：

```c
ORDER_PENDING
ORDER_PAID
ORDER_SHIPPED
```

前缀既能避免重名，也能帮助读者判断成员属于哪一组枚举。

## 七、枚举变量与赋值

```c
enum TrafficLight light = LIGHT_GREEN;
```

随后可以修改状态：

```c
light = LIGHT_YELLOW;
```

从程序设计角度看，枚举变量应只使用该枚举中有意义的成员值。但是 C 语言对枚举与整数之间的限制相对宽松，编译器不一定能阻止所有无效赋值。因此，下面的代码可能通过编译，却不代表设计合理：

```c
light = 100;
```

程序仍需验证外部输入，并为未知状态准备处理方案。

## 八、枚举与 switch 配合

枚举适合表示有限状态，`switch` 适合按离散值分支，二者经常一起使用：

```c
#include <stdio.h>

enum TrafficLight {
    LIGHT_RED,
    LIGHT_YELLOW,
    LIGHT_GREEN
};

void print_action(enum TrafficLight light) {
    switch (light) {
        case LIGHT_RED:
            printf("停止\n");
            break;

        case LIGHT_YELLOW:
            printf("注意信号变化\n");
            break;

        case LIGHT_GREEN:
            printf("确认安全后通行\n");
            break;

        default:
            printf("未知信号状态\n");
            break;
    }
}
```

`default` 用于处理不在预期集合中的值。即使程序认为状态一定有效，保留明确的异常处理通常也有助于发现错误。

## 九、把枚举转换为可显示文本

枚举成员本身不是字符串。若要显示状态名称，可以编写函数：

```c
const char *light_name(enum TrafficLight light) {
    switch (light) {
        case LIGHT_RED:
            return "红灯";
        case LIGHT_YELLOW:
            return "黄灯";
        case LIGHT_GREEN:
            return "绿灯";
        default:
            return "未知状态";
    }
}
```

使用方式：

```c
printf("当前状态：%s\n", light_name(LIGHT_GREEN));
```

这里函数返回字符串字面量的地址，调用者只读取文本，不应尝试修改它。

## 十、枚举表示菜单选项

```c
enum MenuChoice {
    MENU_EXIT = 0,
    MENU_ADD = 1,
    MENU_SEARCH = 2,
    MENU_DELETE = 3
};
```

用户输入首先是普通整数。验证后再按照枚举含义处理：

```c
int input;

if (scanf("%d", &input) != 1) {
    printf("输入格式错误\n");
    return 1;
}

switch (input) {
    case MENU_EXIT:
        printf("退出程序\n");
        break;
    case MENU_ADD:
        printf("添加记录\n");
        break;
    case MENU_SEARCH:
        printf("查询记录\n");
        break;
    case MENU_DELETE:
        printf("删除记录\n");
        break;
    default:
        printf("没有这个菜单选项\n");
        break;
}
```

枚举为选项数字提供名称，但不会替代输入验证。

## 十一、枚举表示程序状态

枚举特别适合描述一个对象在不同时刻所处的状态：

```c
enum DownloadState {
    DOWNLOAD_NOT_STARTED,
    DOWNLOAD_RUNNING,
    DOWNLOAD_PAUSED,
    DOWNLOAD_FINISHED,
    DOWNLOAD_FAILED
};
```

状态变化可以明确写出：

```c
enum DownloadState state = DOWNLOAD_NOT_STARTED;

state = DOWNLOAD_RUNNING;

if (error_occurred) {
    state = DOWNLOAD_FAILED;
}
```

此处 `error_occurred` 仅表示一个已经计算出的条件。使用枚举后，程序中的每种状态都具有可读名称，也便于检查是否遗漏某种情况。

## 十二、枚举表示返回状态

当函数可能产生多种结果时，可以用枚举作为返回值：

```c
enum ParseResult {
    PARSE_OK,
    PARSE_EMPTY_INPUT,
    PARSE_INVALID_FORMAT,
    PARSE_OUT_OF_RANGE
};
```

与只返回 0 或 1 相比，这种设计可以说明失败的具体原因。调用者再使用 `switch` 分别处理各类结果。

枚举返回值描述“发生了哪一种情况”，输出参数则可以带回成功时的计算结果。

## 十三、枚举与结构体组合

结构体描述对象的多项属性，枚举描述其中一个有限状态：

```c
enum StudentStatus {
    STUDENT_ACTIVE,
    STUDENT_GRADUATED,
    STUDENT_SUSPENDED
};

struct Student {
    int id;
    char name[20];
    enum StudentStatus status;
};
```

这样比使用 `int status` 更清楚，因为结构体定义直接说明该成员应采用哪一组状态。

## 十四、不要把枚举用于任意数值

枚举适合有限且具有名称的离散选项，例如：

- 星期；
- 颜色类别；
- 菜单选项；
- 程序状态；
- 错误结果。

枚举不适合表示任意变化的数值，例如温度、成绩、人数或距离。这些数据没有固定且较小的成员集合，应继续使用整数或浮点类型。

## 十五、枚举不是位标志的完整替代品

普通状态通常是“多个选项中只能选择一个”。例如一个交通灯状态不能同时既是红灯又是绿灯。

有些场景允许多个选项同时存在，例如“可读、可写、可执行”权限。这类组合状态常使用二进制位标志和位运算处理。虽然枚举成员可以给各个位值命名，但相关设计依赖 C11 运算符中的位运算知识，不应与普通单选状态混淆。

## 十六、sizeof 与底层表示

可以对枚举类型使用 `sizeof`：

```c
printf("%zu\n", sizeof(enum TrafficLight));
```

枚举在底层使用能够表示其成员值的整数类型，具体大小由实现决定。许多环境中结果与 `int` 相同，但不应把这一结果当作所有平台的固定规则。

对于初学程序，重要的是把枚举看作一组有名称的整数状态，而不是依赖其具体存储大小。

## 十七、常见错误与原因

### 1. 误认为枚举成员是字符串

枚举成员是整数常量。需要显示文本时，应自行建立转换关系。

### 2. 在代码中继续使用裸数字

```c
if (status == 2) { }
```

既然已经定义枚举，就应使用相应成员名，避免数字含义分散在代码中。

### 3. 假设所有枚举都从 0 连续编号

成员可以显式赋值，也可以重复。应阅读定义，不要猜测。

### 4. 不处理无效状态

外部输入、文件内容或程序错误都可能带来未知值。`switch` 中可使用 `default` 进行防御性处理。

### 5. 成员命名过于简单

`RED`、`OK` 等名称容易与其他模块冲突。使用 `LIGHT_RED`、`PARSE_OK` 等前缀更稳妥。

### 6. 把枚举当成严格的数据校验

枚举提升表达能力，但 C 编译器不一定阻止所有不在成员列表中的整数。有效性仍需程序保证。

## 十八、完整示例：订单状态处理

```c
#include <stdio.h>

enum OrderStatus {
    ORDER_PENDING,
    ORDER_PAID,
    ORDER_SHIPPED,
    ORDER_COMPLETED,
    ORDER_CANCELLED
};

const char *order_status_name(enum OrderStatus status) {
    switch (status) {
        case ORDER_PENDING:
            return "待付款";
        case ORDER_PAID:
            return "已付款";
        case ORDER_SHIPPED:
            return "已发货";
        case ORDER_COMPLETED:
            return "已完成";
        case ORDER_CANCELLED:
            return "已取消";
        default:
            return "未知状态";
    }
}

int main(void) {
    enum OrderStatus status = ORDER_PENDING;

    printf("初始状态：%s\n", order_status_name(status));

    status = ORDER_PAID;
    printf("付款后：%s\n", order_status_name(status));

    status = ORDER_SHIPPED;
    printf("发货后：%s\n", order_status_name(status));

    return 0;
}
```

## 十九、与相关知识的联系

- C16 if/else 分支：可用条件语句处理少量状态关系。
- C17 switch 分支：适合对枚举的各个离散成员分别处理。
- C36 结构体：枚举常作为结构体成员描述对象状态。
- C38 typedef 类型别名：可以简化枚举类型名称的书写。
- C45 小型程序设计：菜单、流程状态和错误处理都适合使用枚举。

## 分层练习

1. 定义表示一周七天的枚举，并输出每个成员的整数值。
2. 定义四季枚举，编写函数把枚举值转换为中文名称。
3. 定义考试结果状态，并使用 `switch` 分别输出“通过”“补考”“缺考”。
4. 为文件读取函数设计包含“成功、文件不存在、格式错误、读取失败”的返回状态枚举。
5. 解释枚举为何能减少魔法数字，以及为什么它仍不能替代输入验证。

## 小结

枚举为一组有限、相关的整数状态提供名称。它的主要价值是表达程序含义，而不是节省存储空间。使用枚举时应采用清楚的成员前缀，理解默认值与显式赋值规则，并通过 `switch` 和 `default` 完整处理有效与异常状态。
