---
layout : post
titile : Backus-Naur-Form
tags : CS61A
---

# Backus-Naur-Form


## Basic BNF

The basic form of a grammar rule

```python
symbol0: symbol1 symbol2 ... symboln
```

Symbols represent sets of strings and come in 2 flavors

* Non-terminal symbols: Can expand into either non-terminal symbols or terminals
* Terminal symbols: Strings or regular expressions

```python
item* items: |items item
item+ items: item | items item
item? optitem: | it
```

