# C36. 结构体 struct

## 节点定位

- 所属模块：自定义类型
- 学习难度：综合应用
- 建议学习时间：120-150 分钟
- 典型用途：把描述同一对象的多个不同类型数据组合起来，建立清晰的数据模型。

## 学习目标

- 理解结构体要解决的问题，并区分结构体类型与结构体变量。
- 会定义结构体、声明变量、初始化成员和访问成员。
- 会使用结构体数组表示一组同类对象。
- 会使用结构体指针及 `->` 运算符。
- 理解结构体赋值、函数传参和函数返回结构体的基本规则。
- 初步理解结构体大小、内存对齐和成员生命周期。

## 一、为什么需要结构体

假设程序要记录一名学生的信息，包括学号、姓名和成绩。可以分别定义三个变量：

```c
int student_id = 1001;
char student_name[20] = "Li Ming";
double student_score = 92.5;
```

这种写法在只有一名学生时尚可使用，但人数增加后，数据之间的对应关系会变得不清楚：

```c
int ids[50];
char names[50][20];
double scores[50];
```

三个数组的同一下标共同描述一名学生。程序员必须始终保证它们同步，排序或交换学生信息时也要同时处理三个数组。

结构体可以把属于同一对象的数据组合为一个整体：

```c
struct Student {
    int id;
    char name[20];
    double score;
};
```

现在，一份 `struct Student` 数据就能完整表示一名学生。

## 二、结构体类型与结构体变量

下面的代码只定义了一种结构体类型，并没有创建学生变量：

```c
struct Student {
    int id;
    char name[20];
    double score;
};
```

其中：

- `struct` 是定义结构体的关键字；
- `Student` 是结构体标签；
- `id`、`name`、`score` 是成员；
- 右花括号后的分号不能省略。

定义类型之后，还要声明变量：

```c
struct Student student1;
struct Student student2;
```

可以类比基本类型：

```text
int             是类型，number   是变量
struct Student  是类型，student1 是变量
```

每个结构体变量都有自己独立的一组成员。修改 `student1.score` 不会影响 `student2.score`。

## 三、结构体成员可以使用哪些类型

成员可以是基本类型、数组、指针，也可以是已经完整定义的其他结构体类型：

```c
struct Date {
    int year;
    int month;
    int day;
};

struct Student {
    int id;
    char name[20];
    double score;
    struct Date birthday;
};
```

成员名只需在当前结构体内部保持唯一。不同结构体可以拥有同名成员，例如 `Student` 和 `Teacher` 都可以有 `name` 成员。

结构体不能直接包含一个与自身类型完全相同的成员，因为那会导致大小无限递归：

```c
struct Node {
    int value;
    // struct Node next;  // 错误：无法确定结构体大小
};
```

但可以包含指向自身类型的指针，因为指针大小是确定的：

```c
struct Node {
    int value;
    struct Node *next;
};
```

这种形式常用于链表，属于后续数据结构内容。

## 四、使用点运算符访问成员

结构体变量通过点运算符 `.` 访问成员：

```c
struct Student student;

student.id = 1001;
student.score = 92.5;
```

点号左边必须是结构体变量，右边是该结构体中存在的成员名。

字符数组成员不能使用普通赋值直接接收字符串：

```c
// student.name = "Li Ming";  // 错误：数组不能整体赋值
```

可以在初始化结构体时填写字符串，也可以使用 `<string.h>` 中的字符串处理函数，并确保目标数组空间足够。

## 五、初始化结构体

### 1. 按成员顺序初始化

```c
struct Student student = {1001, "Li Ming", 92.5};
```

初始值必须按照成员定义顺序填写。若以后调整成员顺序，初始化代码也可能需要修改。

### 2. 只提供部分初始值

```c
struct Student student = {1001};
```

第一个成员 `id` 初始化为 1001，其余未明确提供初值的成员会被零初始化。字符数组会得到空字符串所需的初始零字节，`score` 会得到 0.0。

### 3. 全部初始化为零

```c
struct Student student = {0};
```

这是常用的整体零初始化写法。

### 4. 指定成员初始化

```c
struct Student student = {
    .name = "Li Ming",
    .score = 92.5,
    .id = 1001
};
```

指定成员初始化使用 `.成员名 = 初值`，不必依赖成员定义顺序，阅读时也更清楚。

## 六、完整示例：记录并输出学生信息

```c
#include <stdio.h>

struct Student {
    int id;
    char name[20];
    double score;
};

int main(void) {
    struct Student student = {
        .id = 1001,
        .name = "Li Ming",
        .score = 92.5
    };

    printf("学号：%d\n", student.id);
    printf("姓名：%s\n", student.name);
    printf("成绩：%.1f\n", student.score);

    student.score = 95.0;
    printf("修改后的成绩：%.1f\n", student.score);

    return 0;
}
```

这个示例体现了“定义类型、创建变量、初始化成员、读取成员和修改成员”的完整过程。

## 七、嵌套结构体

结构体成员可以是另一个结构体：

```c
struct Date {
    int year;
    int month;
    int day;
};

struct Student {
    int id;
    char name[20];
    struct Date birthday;
};
```

初始化方式如下：

```c
struct Student student = {
    .id = 1001,
    .name = "Li Ming",
    .birthday = {2009, 5, 18}
};
```

访问嵌套成员时连续使用点运算符：

```c
printf("%d\n", student.birthday.year);
```

含义是先访问 `student` 的 `birthday` 成员，再访问其中的 `year` 成员。

## 八、结构体数组

结构体数组适合保存多个同类对象：

```c
struct Student class_members[3] = {
    {1001, "Li Ming", 92.5},
    {1002, "Wang Hua", 88.0},
    {1003, "Chen Yu", 95.5}
};
```

访问某名学生的信息：

```c
printf("%s %.1f\n",
       class_members[1].name,
       class_members[1].score);
```

这里先使用 `[1]` 取得数组中的第二个结构体变量，再使用 `.name` 或 `.score` 访问成员。

### 遍历结构体数组

```c
for (int i = 0; i < 3; i++) {
    printf("%d %s %.1f\n",
           class_members[i].id,
           class_members[i].name,
           class_members[i].score);
}
```

结构体数组使同一对象的各项信息始终存放在一起。交换两个数组元素时，整名学生的信息会一起交换。

## 九、结构体整体赋值

同一种结构体类型的变量可以整体赋值：

```c
struct Student a = {1001, "Li Ming", 92.5};
struct Student b;

b = a;
```

赋值会逐成员复制数据，包括结构体中的数组成员。因此 `b` 得到一份独立副本，之后修改 `b.score` 不会改变 `a.score`。

但如果结构体成员是指针，整体赋值只复制指针中保存的地址，不会自动复制指针所指的动态数据。这种情况称为浅复制，需要结合指针和动态内存知识谨慎处理。

## 十、结构体不能直接使用 == 比较

C 语言不允许直接用 `==` 比较两个结构体变量：

```c
// if (a == b) { }  // 错误
```

应按照程序需要逐个比较成员：

```c
#include <string.h>

if (a.id == b.id &&
    strcmp(a.name, b.name) == 0 &&
    a.score == b.score) {
    printf("信息相同\n");
}
```

实际程序还应考虑浮点数比较精度，不能总是依赖 `double` 的直接相等比较。

## 十一、结构体指针与箭头运算符

指针可以保存结构体变量的地址：

```c
struct Student student = {1001, "Li Ming", 92.5};
struct Student *p = &student;
```

通过指针访问成员有两种等价写法：

```c
(*p).score
p->score
```

通常使用更简洁的 `->`：

```c
p->score = 96.0;
printf("%.1f\n", p->score);
```

`p->score` 可理解为“访问 `p` 所指结构体中的 `score` 成员”。

为什么 `(*p).score` 需要括号？因为点运算符的优先级高于一元 `*`。若写成 `*p.score`，编译器会先处理 `p.score`，但 `p` 是指针而不是结构体变量。

## 十二、区分 . 和 ->

| 左侧表达式 | 成员访问形式 | 示例 |
| --- | --- | --- |
| 结构体变量 | `.` | `student.score` |
| 结构体指针 | `->` | `p->score` |

判断时先看左侧表达式是什么类型，不要只凭变量名称猜测。

## 十三、结构体作为函数参数

### 1. 按值传递结构体

```c
void print_student(struct Student student) {
    printf("%d %s %.1f\n",
           student.id, student.name, student.score);
}
```

函数得到整个结构体的一份副本。在函数内部修改 `student` 不会影响调用者变量。小型结构体使用这种方式简单直观，但大型结构体的复制成本可能较高。

### 2. 传入结构体地址

```c
void print_student(const struct Student *student) {
    if (student == NULL) {
        return;
    }

    printf("%d %s %.1f\n",
           student->id, student->name, student->score);
}
```

调用方式：

```c
print_student(&student);
```

使用指针可以避免复制整个结构体。`const` 表明函数只读取信息，不通过该指针修改结构体。

### 3. 通过指针修改结构体

```c
void update_score(struct Student *student, double new_score) {
    if (student != NULL) {
        student->score = new_score;
    }
}
```

调用时传入地址，函数修改的是调用者原结构体。

## 十四、函数返回结构体

函数可以直接返回结构体值：

```c
struct Student create_student(int id, double score) {
    struct Student student = {0};
    student.id = id;
    student.score = score;
    return student;
}
```

调用方式：

```c
struct Student student = create_student(1001, 92.5);
```

返回结构体值是合法的，调用者得到结构体内容。与“返回局部变量地址”不同，这里返回的是值，而不是已经失效的局部地址。

## 十五、结构体的大小与内存对齐

可以使用 `sizeof` 查看结构体占用的字节数：

```c
struct Example {
    char flag;
    int count;
    double value;
};

printf("%zu\n", sizeof(struct Example));
```

结构体大小不一定等于各成员大小简单相加。编译器可能在成员之间或结构体末尾加入填充字节，使成员地址满足硬件的对齐要求。

例如，若 `char` 占 1 字节、`int` 占 4 字节，编译器可能在 `flag` 后加入若干填充字节，再存放 `count`。

初学阶段应掌握以下结论：

- 成员按定义顺序出现，但中间可能有填充；
- 使用 `sizeof` 获取实际大小，不要手工猜测；
- 不应直接比较结构体内存中的所有字节来判断成员是否相同，因为填充字节的内容可能没有确定意义；
- 不同编译环境中的布局可能不同。

## 十六、结构体中的字符数组与输入

```c
struct Student student;
scanf("%19s", student.name);
```

若 `name` 长度为 20，`%19s` 最多读取 19 个非空白字符，并为结尾的 `\0` 留出一个位置。数组名在这里会转换为首元素地址，所以不写 `&student.name`。

`%s` 遇到空白就停止，因此不能直接读取包含空格的完整姓名。若需要读取整行，应结合 `fgets`，并妥善处理输入缓冲区和换行符。

## 十七、常见错误与原因

### 1. 忘记定义末尾的分号

```c
struct Point {
    int x;
    int y;
};  // 分号不能省略
```

### 2. 只定义类型，没有声明变量

结构体定义只是建立类型模板。要保存数据，还要声明 `struct Student student;`。

### 3. 混淆 . 和 ->

结构体变量使用 `.`，结构体指针使用 `->`。

### 4. 给字符数组成员直接赋字符串

数组不能在定义完成后整体赋值。可以初始化，或使用安全的字符串复制方法。

### 5. 使用未初始化成员

只给部分成员逐项赋值时，其他成员可能仍是不确定值。建议在创建变量时完成初始化。

### 6. 假设结构体没有填充字节

结构体布局受类型和编译环境影响，应使用 `sizeof` 获取大小。

### 7. 忽略指针成员的所有权

结构体中的指针只保存地址。若它指向动态内存，必须明确由谁申请、复制和释放。

## 十八、完整示例：学生成绩统计

```c
#include <stdio.h>

struct Student {
    int id;
    char name[20];
    double score;
};

void print_student(const struct Student *student) {
    if (student != NULL) {
        printf("%d %-12s %.1f\n",
               student->id,
               student->name,
               student->score);
    }
}

int main(void) {
    struct Student students[] = {
        {1001, "Li Ming", 92.5},
        {1002, "Wang Hua", 88.0},
        {1003, "Chen Yu", 95.5}
    };
    int count = (int)(sizeof students / sizeof students[0]);
    int best_index = 0;
    double sum = 0.0;

    for (int i = 0; i < count; i++) {
        print_student(&students[i]);
        sum += students[i].score;

        if (students[i].score > students[best_index].score) {
            best_index = i;
        }
    }

    printf("平均分：%.2f\n", sum / count);
    printf("最高分学生：%s\n", students[best_index].name);

    return 0;
}
```

## 十九、与相关知识的联系

- C27 一维数组：结构体数组用于表示一组具有相同属性的对象。
- C29 字符串：字符数组成员常用于保存姓名、编号等文本。
- C32 指针基础：结构体指针保存结构体变量地址。
- C34 指针与函数：函数可通过结构体指针读取或修改对象。
- C38 typedef 类型别名：可以简化 `struct 类型名` 的重复书写。
- C45 小型程序设计：学生管理、通讯录和商品管理等程序通常使用结构体组织数据。

## 分层练习

1. 定义 `Book` 结构体，包含书名、价格和页数，初始化后输出全部成员。
2. 定义包含年、月、日的 `Date`，再把它作为 `Book` 的出版日期成员。
3. 创建 5 本书的结构体数组，查找价格最高的书。
4. 编写只读输出函数和修改价格函数，分别练习 `const struct Book *` 与 `struct Book *`。
5. 使用 `sizeof` 观察不同成员排列顺序下结构体大小是否变化，并解释填充字节可能产生的原因。

## 小结

结构体把描述同一对象的多个成员组合成一个完整数据类型。学习时必须区分类型和变量、结构体变量和结构体指针，并正确使用 `.` 与 `->`。结构体可以初始化、整体赋值、组成数组、传给函数或从函数返回，是 C 语言组织复杂数据的核心工具。
