---
layout : post
titile : C string
tags : CS61C
---

* A **string** in C is just an array of characters.

  ​										char string[] = "abc";

* How do u tell how long a string is?

  * Last characters is followed by a 0 byte(null terminator)

    ```c
    int strlen(char s[])
    {
        int n = 0;
        while(s[n] != 0) n++;
        return n;
    }
    ```

    

## C String Standard Functions

```c
int strlen(char *string);
// compute the length of string
int strcmp(char *str1, char *str2);
// return 0 if str1 and str2 are identical(how is this different from str1 == str2?)
char *strcpy(char *dst, char *src);
/*
copy the contents of string src to the memory at dst. The caller must ensure that dst has enought memory to hold the data to be copied
*/
```

## Arrays(one elt past array must be valid)

* What if we have an array of large structs?
  * C take care of it: In reality, p+1 doesn't add 1 to the memory address, it adds the **size of the array element**.

```c
*p++ vs (*p)++
x = *p++   -> x = *p; p = p + 1
x = (*p)++ -> x = *p; *p = *p + 1
```

## Arrays vs. Pointers

* An array name is a read-only pointer to the 0th element of the array.
* An array parameter can be declared as an array or a pointer; an array argument can be passed as a pointer.

## Segmentation Fault vs Bus Error?

* Buss Error
  * A fatal failure in the execution of a machine language instruction resulting from the processor detecting an anomalous condition on its bus. Such conditions include **invalid address alignment**(accessing a multi-byte number at an odd address), accessing a physical address that does not correspond to any device. or some other device-specific hardware error. A bus error triggers a processor-level exception which Unix translates into a "SIGBUS" signal which, if not caught, will terminate the current process.

* Segmentation Fault
  * An error in which a running Unix program attempts to **access memory not allocated** to it and terminates with a segmentation violation error and usually a core dump.

## C Strings Headaches

* One common mistake is to forget to allocate an extra byte for the null terminator.
  * When creating a long string by concatenating serval smaller strings, the programmer must insure there is enough space to store the full string!
  * What if u don't know ahead of time how big your string will be?
  * Buffer overrun security holes!

## C structures: Overview

* A **struct** is a data structure composed from simpler data types.

  ```C
  struct point{
      int x;
      int y;
  }
  
  void PrintPoint(struct point p)
  ```

## Linked List Example 

```C
struct Node{
    char *value;
    struct Node *next;
}; // Recursive definition!
// "typedef" means define a new type
typedef struct Node NodeStruct;
//	...or...
typedef struct Node{
    char *value;
    struct Node *next;
}NodeStruct;
// ...Then
typedef NodeStruct * List;
typedef char * String;

/*
Note similarity! To define 2 nodes
*/
struct Node{
    char* value;
    struct Node *next;
}node1, node2;

// Add a string to an existing list
List cons(String s, List list)
{
    List node = (List)malloc(sizeof(NodeStruct));
    node->value =(String)malloc(strlen(s) + 1);
    strcpy(node->value,s);
    node->next = list;
    return node;
}
    
    
    
```

