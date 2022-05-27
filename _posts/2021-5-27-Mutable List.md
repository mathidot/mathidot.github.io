---
layout : post
titile : Mutable-List
tags : CS61A Python
---

We will represent a mutable linked list by a function that has a linked list as its local state. Lists need to have an identity, like any mutable value. In particular, we cannot use None to represent an empty mutable list, because two empty lists are not literal values.  but None is None. On the other hand, two different functions that each have empty as their local state will suffice to distinguish two empty lists.

If a mutable linked list is a function, what arguments does it take? The answer exhibits a general pattern in programming: the function is a dispatch and it arguments are first a message followed by

Computation by such a network proceeds as follows: When a connector is given a value(by the user or by a constraint box to which it is linked) it awakens all of its associated constraints(except for the constraint box to which it is linked) it awakens all of its associate

Each awakened constraint box then polls its connectors to see if there is enough information to determine a value for a connector. If so, the box sets that connector, which then awakens all of its associated constraints, and so on. For instance, in conversion

Constraints are dictionaries that do not hold local states themselves. Their response to messages are non-pure functions that change the connectors that they constrain.

Connectors are dictionaries that hold a current value and respond to messages that manipulate value. Constraints will not change the value of connectors directly, but instead will do so by send messages, so that the connector can notify other constraints in response to the change. In this way connector represents a number, but also encapsulates connector behavior.

![image-20220504085048237](C:\Users\mathidot\AppData\Roaming\Typora\typora-user-images\image-20220504085048237.png)

```python
from operator import add, sub
def adder(a,b,c):
    """The constraint that a + b = c"""
    return make_ternary_constraint(a,b,c,add,sub,sub)

def make_ternary_constraint(a,b,c,ab,ca,cb):
    def new_value():
        av,bv,cv = [connector['has_val']() for connector in (a,b,c)]
        if av and bv:
            c['set_val'](constraint,ab(a['val'],b['val']))
        elif av and cv:
            b['set_val'](constraint,ca(c['val'],a['val']))
        elif av and bv:
            c['set_val'](constraint,cb(c['val'],b['val']))
     def forget_value():
        for connector in (a,b,c):
            connctor['forget'](constraint)
     constraint = ['new_val':new_value,'forget':forget value]
    for connector in (a,b,c):
        connector['connect'](constraint)
    return constraint

def constant(connector,value):
    constraint = {}
    connector['set_val'](constraint, value)
    connector['connect'](constraint)
    return constraint

A connector is represented as a dictionary that contains a value, but has response functions with local state. The connector must track the informant that gave it its current value, and a list of constraints in which it participates.
The constructor has local functions for setting and forgetting values, which are the response to messages from constraints.

def connector(name = None):
    """A connector between constraints"""
	informant = None
    constraints = []
    def set_value(soucre,value):
        nonlocal informant
        val = connector['val']
        if val is None:
            informant, connector['val'] = source,value
            if name is not None:
                print(name, '=', value)
            inform_all_except(source,'new_val',constraints)
        else:
            if val != value:
                print('Contradiction detected:',val,'vs','value')
     def forget_vale(source):
        nonlocal informant
        if informant == souce:
            informant, connector['val'] = None, None
            if name is not None:
                print(name,'is forgetten')
            inform_all_except(source,'forget',constraints)
	connector = {'val': None,
                 'set_val': set_value,
                 'forget': forget_value,
                 'has_val': lambda: connector['val'] is not None},
    			 'connect': lambda source: constraints.append(source)
        
def inform_all_except(source,message,constraints):
    """Inform all constraints of the message, except source"""
    for c in constraints:
        if c != source:
            c[messgae]()




```

Arithmetic. Special methods can also define the behavior of built-in operators applied to user-defined objects. In order to provide this generality. Python follows specify protocols to apply each operator. For example, to evaluate expression that contain the + operator. Python checks for special methods on both the left and right operands  of the expression. First python checks for an\__add\__ method on the value of the right operand as its argument. Some examples are given in the following sections.

## Multiple Representations

Abstraction barriers allow us to separate the use and representation of data. However in large programs. It may not always  make sense to speak of "the underlying representation" for a data type in a program. For one thing, there might be more than one useful representation for a data object,  and we might like to design systems that can deal with multiple representations.

To take a simple example. complex numbers may represented in two almost equivalent ways: in rectangular form and in polar form. Sometimes the rectangular form is more appropriate and sometimes the polar form(magnitude and angle) Indeed, it is perfectly plausible to imagine a system in which complex numbers are represented in both ways, and in which the functions for manipulating complex numbers are represented in both ways. and in which the functions for manipulating complex numbers work with either representation. We implement such a system below. As a side note, we are developing a system that performs arithmetic operations on complex numbers as a simple but unrealistic example of a program that uses generic  operations.

The idea of allowing multiple representations of data arises regularly. Large software systems are often designed by many people working over extended periods of time. subject to requirements that change over time. In such environment, it is simply not possible for everyone to agree in advance We need abstraction barriers that isolate different design choices from each other and permit different choices to coexist in a single program

We will begin our implementation at the highest level of abstraction and work towards concrete representations. A complex number is a Number, and numbers can be added or multiplied together. How numbers can be added or multiplied is abstracted by the method names add and mul.

```python
class Number:
    def __add__(self,other):
        return self.add(other)
    def __mul__(self,other):
        return self.mul(other)
```

This class requires that Number objects have add and mul methods, but does not define them. Moreover, it does not have and __init__ method. The purpose of Number is not to be instantiated directly, but instead to serve as a superclass of specific number classes.

While multiplying complex numbers, it is more natural thinks in terms of representing a complex number in polar form, as a magnitude and an angle. The product of two complex numbers is the vector obtained 

```python
class Complex(Number):
    def add(self,other):
        return ComplexRI(self.real + other.real, self.imag + other.imag)
   	def mul(self,other):
        magnitutde = self.magnitude * other.magnitude
        return ComplexMA(magnitude, self.angle + other.angle)
```

* __ComplexRI__ constructs a complex number from real and imaginary parts
* __ComplexMA__ constructs a complex number from a magnitude and angle

interface. Objects attributes, which are a form of message passing, allows different data types to respond the same message in different ways. A shared set of messages that elicit similar behavior from different classes is a powerful method of abstraction. An interface is a set of shared attribute names. along with a specification of their behavior. In the case of complex numbers., the interface needed to implement arithmetic consists of four attributes: real, image, magnitude and angle.

Properties. The requirement that two or more attributes values maintain a fixed relationship with each other is a new problem. One solution is to store attributes values for only one representation and compute the other representation whenever it is needed.

```python
from math import atan2
class ComplexRI(complex):
    def __init__(self,real,imag):
        self.real = real
        self.image = imag
    @property
    def magnitude(self):
        return (self.real**2 + self.image**2)**0.5
    @property
    def angle(self):
        return atan2(self.image,self.real)
    def __repr__(self):
        return 'ComplexRI({0:g}, {1:g})'.format(self.real, self.imag)
    
from math import sin, cos, pi
class ComplexMA(Complex):
        def __init__(self, magnitude, angle):
            self.magnitude = magnitude
            self.angle = angle
        @property
        def real(self):
            return self.magnitude * cos(self.angle)
        @property
        def imag(self):
            return self.magnitude * sin(self.angle)
        def __repr__(self):
            return 'ComplexMA({0:g}, {1:g} * pi)'.format(self.magnitude, self.angle/pi)
```

The interface approach to encoding multiple representations has appealing properties. The class for each representation can be developed separately, they must only on the names of the attributes they share, as well as any behavior conditions for those attribute. The interface is also additive. If another programmer wanted to add a third representation of complex numbers to the same program, they would only have to create another class with the same attributes.

## Generic Functions

Generic Functions are methods or functions that apply to arguments of different types. We have seen many examples already. The complex.add method is generic, because it can take either a ComplexRI and ComplexMA as the value of other. This flexibility was ensuring that both ComplexRI and ComplexMA share an interface. 

```python
from fractions import gcd
class Rational(Number):
    def __init__(self,number, denom):
        g = gcd(number, denom)
        self.number = number // g
        self.denom = denom // g
    def __repr__(self):
        return 'Rational({0}, {1})'.format(self.numer, self.denom)
    def add(self, other):
            nx, dx = self.numer, self.denom
            ny, dy = other.numer, other.denom
            return Rational(nx * dy + ny * dx, dx * dy)
    def mul(self, other):
            numer = self.numer * other.numer
            denom = self.denom * other.denom
            return Rational(numer, denom)
```

```python
>>> Rational(2, 5) + Rational(1, 10)
Rational(1, 2)
>>> Rational(1, 4) * Rational(2, 3)
Rational(1, 6)
```

## Type dispatching. 

One way to implement cross-type operations is to select behavior based on the types of the arguments to a function or method. The idea of type dispatching is to write functions that inspect the type arguments they receive, then execute that is appropriate for those types.

```python
def is_read(c):
	if isinstance(c, complexRI):
		return c.image == 0
    elif isinstance(c,complexMA):
        return c.angle % pi == 0
```

```python
def add_complex_and_rational(c,r):
    return ComplexRI(c.real + r.number / r.denom, c.imag)

def mul_complex_and_rational(c,r):
    return ComplexMA(c.magnitude * r.magnitude, c.angle + r.angle)
```

The role of type dispatching is to ensure that these cross-type operations are used at appropriate times. Below, we rewrite the Number superclass to use type dispatching for its __add__ and __mul__ methods.

We use the type_tag attribute to distinguish types of arguments. One could directly use the built-in isinstance method as well.

The __add__ method  considers two cases.

```python
class Number:
	def __add__(self,other):
        if self.type_tag == other.tag:
            return self.add(other)
        elif (self.type_tag,other.type_tag) in self.adders:
            return self.cross_apply(other, self.adder)
     def __mul__(self,other):
        if self.type_tag == other.tag:
            return self.mul(other)
        elif (self.type_tag,other.type_tag) in self.multiplies:
            return self.cross_apply(other,self.multiplies)
     def cross_apply(self,other,cross_fns):
        cross_fn = cross_fns[(self.type_tag,other.type_tag)]
        return cross_fn(self,other)
     adders = {("com", "rat"): add_complex_and_rational,
                  ("rat", "com"): add_rational_and_complex}
     multipliers = {("com", "rat"): mul_complex_and_rational,
                       ("rat", "com"): mul_rational_and_complex}
        
        
```

In this new definition of the Number class, all cross-type implementation are 

While we have introduced some complexity to the system, we can now mix types in addition and mutiplication expressions

Coercion. Often the different data types are not completely independent, and there may be ways by which objects of one type may be viewed as being of another type. This process is called coercion. For example, if we are asked to arithmetically combine a rational number with a complex number, we can view the rational number as a complex number a rational number with a complex number. we can view the rational numbers as a complex number

In general, we can implement this ideas by designing coercion functions that transform an object of one type into an equivalent object of another type.

```python
def rational_to_complex(r):
    return ComplexRI(r.numer / r.denom, 0)
```

The alternative definition of the Number class performs cross-type operations by attempting to coerce both arguments to the same type. The coercions dictionary indexes all possible coercions by a pair of type tags. Indicating that the corresponding value coerces a value of the first type to a value of the second type

The coerce method returns two values with the same type_tag. It inspects the type_tags of its argument, compares them to entries in the coercions dictionary. and converts one argument to the type of the other using coerce_to. Only 

```python
class Number:
    def __init__(self,other):
        x, y = self.coerce(other)
    def __mul__(self,other):
        x, y = self.coerce(other)
        return x.mul(y)
  	def coerce(self,other):
        if self.type_tag == other.type_tag:
            return self,other
        elif (self.type_tag,other.type_tag) in self.coercions:
            return (self.coerce_to(other.type_tag), other)
        elif (other.type_tag,self.type_tag) in self.coercions:
            return (self,other.coerce_to(self.type_tag))
     def coerce_to(self,other_tag):
        coercion_fn = self.coercions[(self.type_tag, other_tag)]
        return coercion_fn(self)
     coercions = [('rat', 'com'); rational_to_complex]
```

Further advantages come from extending coercion. Some more sophisticated coercions schemes do not just try to coerce one type into another. but instead may try to coerce two different types each into a third common type

## Streams

Streams offer another another way to represent sequential data implicitly. A stream is a lazily computed linked list. Like the _Link_ class. A stream instance responds to requests for its _first_ element and the _rest_ of the stream. Like an _Link_, the rest of a Stream is itself a Stream. Unlike an Link, the rest of a stream is only computed when it is looked up, rather than being stored in advance. That is, the rest of a stream is computed lazily

To achieve this lazy evaluation, a stream stores a function that computes the rest of the stream. Whenever this function is called, its returned value is cached as part of the stream in an attribute called \_rest named with an underscore to indicate that it should not be accessed directly.

```python
class Stream:
    """A lazily computed linked list"""
    class empty:
        def __repr__(self):
            return 'Stream empty'
    empty = empty()
    def __init__(self,first,compute_test=lambda: empty):
    	assert callable(compute_test), 'compute_rest must be callable'
        self.first = first
        self._compute_rest = compute_rest
    @property
    def rest(self):
        """Return the rest of the stream computing it if necessary"""
        if self._compute_rest is not None:
            self._rest = self._compute_rest()
            self._compute_rest = None
        return self._rest
   	def __repr__(self):
        return 'Stream({0},<...>)'.format(repr(self.first))
```

When a _Stream_ instance is constructed, the field _self.\_rest_ is None, signifying that the rest of the _Stream_ has not yet been computed. When the rest attribute is requested via a dot expression, the rest property method is invoked, which triggers computation with _self.\_rest = self.\_compute_rest()_ 

```python
def integer_stream(first):
    def compute_rest():
        return integer_stream(first+1)
   	return Stream(fist,compute_rest)
```

```python
def map_stream(fn,s):
    if s is Stream.empty:
        return s
    def compute_rest():
        return map_stream(fn,s.rest)
    return Stream(fn(s.first),compute_rest)
```

```python
def filter_stream(fn,s):
    if s is Stream.empty:
        return s
    def compute_test():
        return filter_stream(fn,s.rest)
    if fn(s.first):
        return Stream(s.first,compute_rest)
    else:
        return compute_rest()
```

We can use our _filter_stream_ function to define a stream of prime numbers using the sieve of Eratosthenes, which filters a stream of integers to remove all numbers that are multiples of its first element. By successively filtering with each prime, all composite numbers are removed from the stream.

```python
def primes(pos_stream):
    def not_divible(x):
        return x % pos_stream.first != 0
    def compute_rest():
        return primes(filter_stream(not_divible,pos_stream.rest))
    return Stream(pos_stream.first,compute_rest)
        
```
