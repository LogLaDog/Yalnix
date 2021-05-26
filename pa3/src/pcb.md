# Process Control Block 
```C
typedef struct pcb
{
    
    int pid; // process id 
    UserContext usercontext; // user context
    KernelContext kernelcontext; // kernel context
	int exitStatus; // exit status of this process
    int clock_ticks; // number of clock ticks 
    Queue *children;
    pcb *parent; // pointer to parent of this process
    void *brk; // address of the lowest location not used by the program
    pte kernelstack[PAGESIZE];
  
    
} pcb;
```