`ugetpid`函数代码如下：

```c
int 
ugetpid(void)
{
    struct usyscall *u = (struct usyscall *)USYSCALL;
    return u->pid;
}
```

`proc_pagetable`函数

```c
// Create a user page table for a given process
// with no user memory, but with trampoline pages
pagetable_t
proc_pagetable(struct proc *p)
{
    pagetable_t pagetable;
    
    // An empty page table
    pagetable = uvmcreate();
    if(pagetable == 0)
        return 0;
    
    // map the trampoline code (for system call return)
    // at the highest user virtual address
    // only the supervisor uses it, on the way
    // to/from user space, so not PTE_U
    
    if(mappages(pagetable, TRAMPOLINE, PAGESIZE,
               	(uint64)trampoline, PTE_R | PTE_X) < 0){
        uvmfree(pagetable,0);
        return 0;
    }
    
    // map the trapframe just below TRAMPOLINE for trampoline.S
    if(mappages(pagetable, TRAPFRAME, PGSIZE,
               	(uint64)(p->trapframe), PTE_R | PTE_W) < 0){
        uvmunmap(pagetable, TRAMPOLINE, 1, 0);
        uvmfree(pagetable, 0);
        return 0;
    }
    
    return pagetable;
}
```

`mappages` 函数：

```c
// Create PTEs for virtual addresses starting at va that refer to 
// physical addresses starting at pa. va and size might not 
// be page-aligned. Returns 0 on success, -1 if walk() couldn't 
// allocate a needed page-table page.

int mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
    uint64 a, last;
    pte_t *pte;
    
    if(size == 0)
        panic("mappages: size");
    
    a = PGROUNDDOWN(va);
    last = PGROUNDDOWN(va + size - 1);
    for(;;){
        if((pte = walk(pagetable, a, 1)) == 0)
            return -1;
        if(*pte & PTE_V)
            panic("mappages: remap");
        *pte = PA2PTE(pa) | perm | PTE_V;
        if(a == last)
            break;
        a += PGSIZE;
        pa += PGSIZE;
    }
    return 0;
}
```

