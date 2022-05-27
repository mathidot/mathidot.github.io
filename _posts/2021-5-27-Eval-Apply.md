---
layout : post
titile : EVAL-APPLY
tags : CS61A PYTHON
---

## Eval

Base cases:

* Primitive  values(numbers)
* Look up  values bound to symbols

Recursive calls:

* Eval(operator, operands) of call expressions

* Apply(procedure, arguments)
* Eval(sub-expression) of special forms

# Apply

Base cases:

* Built-in primitive procedures

Recursive calls:

* Eval(body) of user-defined procedures

# Scheme evaluation

The _scheme_val function chooses behavior based on expression form:

* Symbols are looked up in the current environment
* Self-evaluating expressions(booleans, numbers, nil) are returned as values
* All other legal expressions are represented as Scheme lists, called combinations

## Evaluating combinations

The special forms can all be identified by the first elements:

```scheme
(if <predicate> <consequent> <alternative>)
(lambda (<formal-parameters> <body>))
(define <name> <expression>)
```

Any combination that is not a known special form must be a call expression

```scheme
(define (demo s)
  	(if (null? s)
        '(3)
        (cons (car s) (demo (cdr s)))));
(demo (list 1 2))
```

Logical special forms

Logical forms are special forms that may only evaluate some sub-expressions.

Quotation

'<expression> is shorthand for (quote <expression>) '(1 2) is equivalent to (quote (1 2))

Define Expressions

Define binds a symbol to a value in the first frame of the current environment

(define <name> <expression>) 

* Evaluate the <expression>
* Bind <name> to its value in the current frame

Procedure definition is shorthand of define with a lambda expression

(define (<name> <formal parameters>) <body>)

(define <name> (lambda (<formal parameters>) <body>))

## Lambda Expressions

Lambda expressions evaluate to user-defined procedures

(lambda (<formal-parameters>) <body> ...)

```python
class LambdaProcedure:
    def __init__(self,formals,body,env):
        self.formals = formals
        self.body = body
        self.env = env
```

## 尾递归调用

In Scheme interpreters, a tail-recursive function should only require a constant number of active frames.

```scheme
(define (factorial n k)
  	(if (= n 0)
        k
        (factorial (- n 1) (* k n))))
```

A **tail call** is a call expression in a **tail context**:

* The last body sub-expression in a lambda expression
* Sub-expression 2&3 in a tail context if expression
* All non-predicate sub-expression in a tail context
* The last sub-expression in a tail context and, or, begin or let.

```scheme
(define (length s)
  (if (null? s) 0
  (+ 1 (length (cdr s)))))
//A call expression is not a tail call if more computation is stil required in the calling procedure
(define (length-tail s)
  (define (length-iter s n)
    (if (null? s) n
    (length-iter (cdr s) (+ 1 n))))
  (length-iter s 0))

```

## Reduce

```scheme
(reduce * '(3 4 5) 2) 120
(define (reduce procedure s start)
  	(if (null? s) start
        (reduce procedure
             (cdr s)
             (procedure start (car s)))))
```

Is is tail recursive?

Yes! Because reduce is in a tail context.

However, if **procedure** is not tail recursive, then this may still require more than constant space for execution.

```scheme
(map (lambda (x) (- 5 x)) (list 1 2))
(define (map procedure s)
  	(if (null? s)
        nil
        (cons (procedure (car s))
              (map procedure (cdr s)))))
```

![image-20220513154909625](C:\Users\mathidot\AppData\Roaming\Typora\typora-user-images\image-20220513154909625.png)

Is it tail recursive?

×No, because **map** is not in a tail context

```scheme
(define (map procedure s)
  (define (map-reverse s m)
    (if (null? s)
        m
        (map-reverse (cdr s) (cons (procedure (car s)) m))))
  (reverse (map-reverse s nil)))

(define (reverse s)
  (define (reverse-iter s r)
  (if (null? s)
      r
      (reverse-iter (cdr s) (cons (car s) r))))
  (reverse-iter s nil))

(map (lambda (x) (- 5 x)) (list 1 2))
```

## Tail call optimization with trampolining

### What the thunk?

Thunk: An expression wrapped in an argument-less function.

Making thunks in Python:

```scheme
thunk1 = lambda: 2 * (3 + 4)
thunk2 = lambdaL add(2, 4)
```

Calling a thunk later:

```scheme
thunk1()
thunk2()
```

## Trampolining

Trampoline: A loop that iteratively invokes thunk-returning functions.

```python
def trampoline(f, *args):
	v = f(*args)
    while callable(v):
        v = v()
    return v
```

The function needs to be thunk-returning! One possibility:

```python
def factorial_thunked(n, k):
    if n == 0:
        return k
    else:
        return lambda: factorial_thunked(n - 1, k * n)
    
trampoline(factorial_thunked,3, 1)
```

## Tail Context

When trying to identify whether a given function call within the body of a function is a tail call, we look for whether the call expression is in **tail context**.

Given that each of the following expressions is the last expression in the body of the function, the following expressions are tail contexts:

1. the second or third operand in an `if` expression
2. any of the non-predicate sub-expressions in a `cond` expression (i.e. the second expression of each clause)
3. the last operand in an `and` or an `or` expression
4. the last operand in a `begin` expression's body
5. the last operand in a `let` expression's body

For example, in the expression `(begin (+ 2 3) (- 2 3) (* 2 3))`, `(* 2 3)` is a tail call because it is the last operand expression to be evaluated.	
