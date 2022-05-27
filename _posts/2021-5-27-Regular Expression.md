---
layout : post
titile : Regular Expressions
tags : CS61A
---

```python
\d # Matches any digit character. Equivalent to [0-9]
\D # Matches all non-digit characters
\w # Matches any word character. == [A-Za-z0-9_]
\s # Matches any whitespaces, tabs or line breaks
*  # Matches 0 or more of the previous pattern
+  # Matches 1 or more of the previous pattern
?  # Matches 0 or 1 of the previous pattern
{} # Use like {Min,Max}
^  # Matches the begining of a string
$  # Matches the end of a string
\b # Matches a world boundary, the begining or end of a word
```

# The re module

```python
re.search(pattern,string)  # returns a Match object representing the first occurrence of pattern within string
re.fullmatch(pattern,string) # returns a Match object, requiring that pattern matches the entirety of string
re.match(pattern,string) # returns a Match object, requiring that string starts with a substring that matches pattern
re.findall(pattern,string) # returns a list of strings representing all matches of pattern within string, from left to right
re.sub(pattern,repl,string) # substitutes all matches of pattern within 
```

