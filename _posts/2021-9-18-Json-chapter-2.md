---
layout : post
titile : Json解析数字
tags : C Json
---

# Json 解析数字

## 初探重构

在讨论解析数字之前，我们再补充 TDD 中的一个步骤──重构（refactoring）。根据[1]，重构是一个这样的过程：

> 在不改变代码外在行为的情况下，对代码作出修改，以改进程序的内部结构。

在 TDD 的过程中，我们的目标是编写代码去通过测试。但由于这个目标的引导性太强，我们可能会忽略正确性以外的软件品质。在通过测试之后，代码的正确性得以保证，我们就应该审视现时的代码，看看有没有地方可以改进，而同时能维持测试顺利通过。我们可以安心地做各种修改，因为我们有单元测试，可以判断代码在修改后是否影响原来的行为。

那么，哪里要作出修改？Beck 和 Fowler（[1] 第 3 章）认为程序员要培养一种判断能力，找出程序中的坏味道。例如，在第一单元的练习中，可能大部分人都会复制 lept_parse_null() 的代码，作一些修改，成为 lept_parse_true() 和lept_parse_false()。如果我们再审视这 3 个函数，它们非常相似。这违反编程中常说的 DRY（don't repeat yourself）原则。本单元的第一个练习题，就是尝试合并这 3 个函数。

另外，我们也可能发现，单元测试代码也有很重复的代码，例如 test_parse_invalid_value() 中我们每次测试一个不合法的 JSON 值，都有 4 行相似的代码。我们可以把它用宏的方式把它们简化：

```c
#define TEST_ERROR(error, json)\
    do {\
        lept_value v;\
        v.type = LEPT_FALSE;\
        EXPECT_EQ_INT(error, lept_parse(&v, json));\
        EXPECT_EQ_INT(LEPT_NULL, lept_get_type(&v));\
    } while(0)

static void test_parse_expect_value() {
    TEST_ERROR(LEPT_PARSE_EXPECT_VALUE, "");
    TEST_ERROR(LEPT_PARSE_EXPECT_VALUE, " ");
}
```

# 2. JSON 数字语法

回归正题，本单元的重点在于解析 JSON number 类型。我们先看看它的语法：

```text
number = [ "-" ] int [ frac ] [ exp ]
int = "0" / digit1-9 *digit
frac = "." 1*digit
exp = ("e" / "E") ["-" / "+"] 1*digit
```

number 是以十进制表示，它主要由 4 部分顺序组成：负号、整数、小数、指数。只有整数是必需部分。注意和直觉可能不同的是，正号是不合法的。

整数部分如果是 0 开始，只能是单个 0；而由 1-9 开始的话，可以加任意数量的数字（0-9）。也就是说，0123 不是一个合法的 JSON 数字。

小数部分比较直观，就是小数点后是一或多个数字（0-9）。

JSON 可使用科学记数法，指数部分由大写 E 或小写 e 开始，然后可有正负号，之后是一或多个数字（0-9）。

JSON 标准 [ECMA-404](https://link.zhihu.com/?target=http%3A//www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf) 采用图的形式表示语法，也可以更直观地看到解析时可能经过的路径：

![img](https://pic1.zhimg.com/80/v2-de5a6e279cbac2071284bfa7bb1e5730_720w.png)

上一单元的 null、false、true 在解析后，我们只需把它们存储为类型。但对于数字，我们要考虑怎么存储解析后的结果。

# 3. 数字表示方式

从 JSON 数字的语法，我们可能直观地会认为它应该表示为一个浮点数（floating point number），因为它带有小数和指数部分。然而，标准中并没有限制数字的范围或精度。为简单起见，leptjson 选择以双精度浮点数（C 中的 double 类型）来存储 JSON 数字。

我们为 lept_value 添加成员：



```c
typedef struct {
    double n;
    lept_type type;
}lept_value;
```

仅当 type == LEPT_NUMBER 时，n 才表示 JSON 数字的数值。所以获取该值的 API 是这么实现的：



```c
double lept_get_number(const lept_value* v) {
    assert(v != NULL && v->type == LEPT_NUMBER);
    return v->n;
}
```

使用者应确保类型正确，才调用此 API。我们继续使用断言来保证。

# 4. 单元测试

我们定义了 API 之后，按照 TDD，我们可以先写一些单元测试。这次我们使用多行的宏的减少重复代码：



```c
#define TEST_NUMBER(expect, json)\
    do {\
        lept_value v;\
        EXPECT_EQ_INT(LEPT_PARSE_OK, lept_parse(&v, json));\
        EXPECT_EQ_INT(LEPT_NUMBER, lept_get_type(&v));\
        EXPECT_EQ_DOUBLE(expect, lept_get_number(&v));\
    } while(0)

static void test_parse_number() {
    TEST_NUMBER(0.0, "0");
    TEST_NUMBER(0.0, "-0");
    TEST_NUMBER(0.0, "-0.0");
    TEST_NUMBER(1.0, "1");
    TEST_NUMBER(-1.0, "-1");
    TEST_NUMBER(1.5, "1.5");
    TEST_NUMBER(-1.5, "-1.5");
    TEST_NUMBER(3.1416, "3.1416");
    TEST_NUMBER(1E10, "1E10");
    TEST_NUMBER(1e10, "1e10");
    TEST_NUMBER(1E+10, "1E+10");
    TEST_NUMBER(1E-10, "1E-10");
    TEST_NUMBER(-1E10, "-1E10");
    TEST_NUMBER(-1e10, "-1e10");
    TEST_NUMBER(-1E+10, "-1E+10");
    TEST_NUMBER(-1E-10, "-1E-10");
    TEST_NUMBER(1.234E+10, "1.234E+10");
    TEST_NUMBER(1.234E-10, "1.234E-10");
    TEST_NUMBER(0.0, "1e-10000"); /* must underflow */
}
```

以上这些都是很基本的测试用例，也可供调试用。大部分情况下，测试案例不能穷举所有可能性。因此，除了加入一些典型的用例，我们也常会使用一些边界值，例如最大值等。练习中会让同学找一些边界值作为用例。

除了这些合法的 JSON，我们也要写一些不合语法的用例：



```abap
static void test_parse_invalid_value() {
    /* ... */
    /* invalid number */
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "+0");
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "+1");
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, ".123"); /* at least one digit before '.' */
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "1.");   /* at least one digit after '.' */
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "INF");
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "inf");
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "NAN");
    TEST_ERROR(LEPT_PARSE_INVALID_VALUE, "nan");
}
```

# 5. 十进制转换至二进制

我们需要把十进制的数字转换成二进制的 double。这并不是容易的事情 [2]。为了简单起见，leptjson 将使用标准库的[strtod()](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/c/string/byte/strtof) 来进行转换。strtod() 可转换 JSON 所要求的格式，但问题是，一些 JSON 不容许的格式，strtod() 也可转换，所以我们需要自行做格式校验。



```c
#include <stdlib.h>  /* NULL, strtod() */

static int lept_parse_number(lept_context* c, lept_value* v) {
    char* end;
    /* \TODO validate number */
    v->n = strtod(c->json, &end);
    if (c->json == end)
        return LEPT_PARSE_INVALID_VALUE;
    c->json = end;
    v->type = LEPT_NUMBER;
    return LEPT_PARSE_OK;
}
```

加入了 number 后，value 的语法变成：

```text
value = null / false / true / number
```

记得在第一单元中，我们说可以用一个字符就能得知 value 是什么类型，有 11 个字符可判断 number：

- 0-9/- ➔ number

但是，由于我们在 lept_parse_number() 内部将会校验输入是否正确的值，我们可以简单地把余下的情况都交给lept_parse_number()：



```c
static int lept_parse_value(lept_context* c, lept_value* v) {
    switch (*c->json) {
        case 't':  return lept_parse_true(c, v);
        case 'f':  return lept_parse_false(c, v);
        case 'n':  return lept_parse_null(c, v);
        default:   return lept_parse_number(c, v);
        case '\0': return LEPT_PARSE_EXPECT_VALUE;
    }
}
```

## 1. 重构合并

由于 true / false / null 的字符数量不一样，这个答案以 for 循环作比较，直至 '\0'。



```c
static int lept_parse_literal(lept_context* c, lept_value* v,
    const char* literal, lept_type type)
{
    size_t i;
    EXPECT(c, literal[0]);
    for (i = 0; literal[i + 1]; i++)
        if (c->json[i] != literal[i + 1])
            return LEPT_PARSE_INVALID_VALUE;
    c->json += i;
    v->type = type;
    return LEPT_PARSE_OK;
}

static int lept_parse_value(lept_context* c, lept_value* v) {
    switch (*c->json) {
        case 't': return lept_parse_literal(c, v, "true", LEPT_TRUE);
        case 'f': return lept_parse_literal(c, v, "false", LEPT_FALSE);
        case 'n': return lept_parse_literal(c, v, "null", LEPT_NULL);
        /* ... */
    }
}
```

注意在 C 语言中，数组长度、索引值最好使用 size_t 类型，而不是 int 或 unsigned。

你也可以直接传送长度参数 4、5、4，只要能通过测试就行了。

## 2. 边界值测试

这问题其实涉及一些浮点数类型的细节，例如 IEEE-754 浮点数中，有所谓的 normal 和 subnormal 值，这里暂时不展开讨论了。以下是我加入的一些边界值，可能和同学的不完全一样。

```c
/* the smallest number > 1 */
TEST_NUMBER(1.0000000000000002, "1.0000000000000002");
/* minimum denormal */
TEST_NUMBER( 4.9406564584124654e-324, "4.9406564584124654e-324");
TEST_NUMBER(-4.9406564584124654e-324, "-4.9406564584124654e-324");
/* Max subnormal double */
TEST_NUMBER( 2.2250738585072009e-308, "2.2250738585072009e-308");
TEST_NUMBER(-2.2250738585072009e-308, "-2.2250738585072009e-308");
/* Min normal positive double */
TEST_NUMBER( 2.2250738585072014e-308, "2.2250738585072014e-308");
TEST_NUMBER(-2.2250738585072014e-308, "-2.2250738585072014e-308");
/* Max double */
TEST_NUMBER( 1.7976931348623157e+308, "1.7976931348623157e+308");
TEST_NUMBER(-1.7976931348623157e+308, "-1.7976931348623157e+308");
```

另外，这些加入的测试用例，正常的 strtod() 都能通过。所以不能做到测试失败、修改实现、测试成功的 TDD 步骤。

有一些 JSON 解析器不使用 strtod() 而自行转换，例如在校验的同时，记录负号、尾数（整数和小数）和指数，然后 naive 地计算：

```c
int negative = 0;
int64_t mantissa = 0;
int exp = 0;

/* 解析... 并存储 negative, mantissa, exp */
v->n = (negative ? -mantissa : mantissa) * pow(10.0, exp);
```

这种做法会有精度问题。实现正确的答案是很复杂的，RapidJSON 的初期版本也是 naive 的，后来 RapidJSON 就内部实现了三种算法（使用 kParseFullPrecision 选项开启），最后一种算法用到了大整数（高精度计算）。有兴趣的同学也可以先尝试做一个 naive 版本，不使用 strtod()。之后可再参考 Google 的 [double-conversion](https://link.zhihu.com/?target=https%3A//github.com/google/double-conversion) 开源项目及相关论文。

## 3. 校验数字

这条题目是本单元的重点，考验同学是否能把语法手写为校验规则。我详细说明。

首先，如同 lept_parse_whitespace()，我们使用一个指针 p 来表示当前的解析字符位置。这样做有两个好处，一是代码更简单，二是在某些编译器下性能更好（因为不能确定 c 会否被改变，从而每次更改 c->json 都要做一次间接访问）。如果校验成功，才把 p 赋值至 c->json。



```c
static int lept_parse_number(lept_context* c, lept_value* v) {
    const char* p = c->json;
    /* 负号 ... */
    /* 整数 ... */
    /* 小数 ... */
    /* 指数 ... */
    v->n = strtod(c->json, NULL);
    v->type = LEPT_NUMBER;
    c->json = p;
    return LEPT_PARSE_OK;
}
```

我们把语法再看一遍：

```text
number = [ "-" ] int [ frac ] [ exp ]
int = "0" / digit1-9 *digit
frac = "." 1*digit
exp = ("e" / "E") ["-" / "+"] 1*digit
```

负号最简单，有的话跳过便行：



```c
    if (*p == '-') p++;
```

整数部分有两种合法情况，一是单个 0，否则是一个 1-9 再加上任意数量的 digit。对于第一种情况，我们像负数般跳过便行。对于第二种情况，第一个字符必须为 1-9，如果否定的就是不合法的，可立即返回错误码。然后，有多少个 digit 就跳过多少个。



```c
    if (*p == '0') p++;
    else {
        if (!ISDIGIT1TO9(*p)) return LEPT_PARSE_INVALID_VALUE;
        for (p++; ISDIGIT(*p); p++);
    }
```

如果出现小数点，我们跳过该小数点，然后检查它至少应有一个 digit，不是 digit 就返回错误码。跳过首个 digit，我们再检查有没有 digit，有多少个跳过多少个。这里用了 for 循环技巧来做这件事。



```c
    if (*p == '.') {
        p++;
        if (!ISDIGIT(*p)) return LEPT_PARSE_INVALID_VALUE;
        for (p++; ISDIGIT(*p); p++);
    }
```

最后，如果出现大小写 e，就表示有指数部分。跳过那个 e 之后，可以有一个正或负号，有的话就跳过。然后和小数的逻辑是一样的。



```c
    if (*p == 'e' || *p == 'E') {
        p++;
        if (*p == '+' || *p == '-') p++;
        if (!ISDIGIT(*p)) return LEPT_PARSE_INVALID_VALUE;
        for (p++; ISDIGIT(*p); p++);
    }
```

这里用了 18 行代码去做这个校验。当中把一些 if 用一行来排版，而没用采用传统两行缩进风格，我个人认为在不影响阅读时可以这样弹性处理。当然那些 for 也可分拆成三行：



```c
        p++;
        while (ISDIGIT(*p))
            p++;
```

## 4. 数字过大的处理

最后这题纯粹是阅读理解题。



```c
#include <errno.h>   /* errno, ERANGE */
#include <math.h>    /* HUGE_VAL */

static int lept_parse_number(lept_context* c, lept_value* v) {
    /* ... */
    errno = 0;
    v->n = strtod(c->json, NULL);
    if (errno == ERANGE && (v->n == HUGE_VAL || v->n == -HUGE_VAL))
        return LEPT_PARSE_NUMBER_TOO_BIG;
    /* ... */
}
```