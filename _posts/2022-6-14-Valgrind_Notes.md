---
layout : post
titile : valgrind note
tags : CS61C
---


## Using Valgrind to find segfaults

1. Edit the `Makefile` to include the `-g` flag in `CFLAGS` to provide debugging information to Valgrind.

2. Compile

   ```
   linked_list.c
   ```

   and

   ```
   test_linked_list.c
   ```

   by executing

   ```bash
   make linked_list
   ```

By default, memcheck is the tool that is run when you invoke Valgrind. The documentation on Valgrind's [memcheck](https://valgrind.org/docs/manual/mc-manual.html) is very useful, as it provides examples of the most common error messages, what they mean, and some optional arguments you can use to help debug them.

![img](https://cs61c.org/sp22/labs/lab02/valgrind_output.png)

Box 1. This shows us the command that we are running through Valgrind.

Box 2. This is a print statement from our program.

Box 3. We are reading 8 bytes from an invalid memory address on linked_list.c line 62.

Box 4. Our program received a segfault by accessing invalid memory on linked_list.c line 62

Box 5. There were no memory leaks at the time that the program exited.

Box 6. We encountered 1 error.

1. Remember from Lab01 that `(*head)->next` produced a segfault because we forgot to check if `*head==NULL`. Fix this error, run your code through Valgrind again, and you should see that there are no errors reported by valgrind.

### Using Valgrind to detect memory leaks

1. Let's cause a memory leak in `test_linked_list.c`. Comment out the line that calls `free_list`.

2. Run `make linked_list` to compile your code.

3. Run valgrind

   ```bash
   valgrind ./linked_list
   ```

4. We can see that our program is still producing the correct result based on the printed messages "Congrats..."; however, we are now experiencing memory leaks. Valgrind tells us to "Rerun with --leak-check=full to see details of leaked memory", so let's do that

   ```bash
   valgrind --leak-check=full ./linked_list
   ```

![img](https://cs61c.org/sp22/labs/lab02/valgrind_output_2.png)

Box 1. Summary of heap usage. There were 80 bytes allocated in 5 different blocks in the heap at the time of exit.

Box 2. Stack trace showing where the unfreed blocks were allocated.

* Direct blocks are those which are root nodes (blocks of memory that the programmer has direct access to, ex stack/global pointer to the heap).
* Indirect blocks are those which are not root nodes (ex a pointer inside of a struct).

Box 3. Summary of leak. You can find more info about memory leaks [in the Valgrind docs](https://www.valgrind.org/docs/manual/mc-manual.html#mc-manual.leaks)

You can use the stack trace to see where the unfreed blocks were allocated. Hopefully this example will help you understand Valgrind messages when you are completing your projects
