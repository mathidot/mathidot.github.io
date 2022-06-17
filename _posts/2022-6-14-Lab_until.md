​	xv6 takes the traditional form of `kernel`, a special program that provides services to running programs. Each running program, called a process, has memory containing instructions, data and a stack. The instructions implement the program;s computation. The data are the variables on which the computation acts. The  stack organizes the program's procedure calls.

## Process and memory

​	A xv6 process consists of user-space memory(instructions, data, and stack) and per-process state private to the kernel. Xv6 times-shares process: it transparently switches the available CPUs among the set of processes waiting to execute. When a process is not executing, xv6 saves its CPU registers, restoring them when it next runs the process. The kernel associates a process identifier, or PID, with each process.

​	A process may create a new process using the `fork` system call. `Fork` gives the new process exactly the same memory contents(both instructions and data) as the calling process. `fork` returns the new process's PID. In the new process, `fork` returns zero. The original and new processes are often called parent and child.

```c
if(pid > 0){
	printf("parent: child=%d\n", pid);
	pid = wait((int *) 0);
	printf("child %d is done\n", pid);
} else if(pid == 0){
	printf("child: exiting\n");
	exit(0);
} else {
	printf("fork error\n");
}
```

