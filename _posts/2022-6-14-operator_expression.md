---
layout : post
titile : operators and expressions
tags : CS61C
---

## 条件操作符

```c
expression1? expression2: expression3
// expression1为真， 那么整个表达式的值就是expression2,否则整个表达式的值就是expreesion3
// Example
if(a > 5)
    b[2*c + d * (e / 5)] = 3
else
    b[2*c + d * (e / 5)] = 20
b[2*c + d * (e / 5)] = a > 5 ? 3 : 20
```

## 逗号操作符

```C
expression1, expression2, ... ,,expressionN
/* 
逗号操作符将两个或多个表达式分开来。这些表达式自左向右逐个进行求值，整个逗号表达式的值就是最后那个表达	   式的值
*/
//Example
a = get_value();
count_value(a);
while(a > 0){
    ...
    a = get_value();
    count_value(a);
}

while(a = get_value(),count_value(a),a > 0){
    ...
}

while(count_value(a = get_value()), a > 0){
    ...
}

```

