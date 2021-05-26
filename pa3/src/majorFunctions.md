# Pseudocode of the Major Functions within an OS

An operating system is built up of many different moving parts, but the main focus comes to three main components; process management, Scheduling, and memory management. Beyond these main components, the OS is also responsible for information protection and security, but that isn't touched upon in this high-level pseudocode view.

## Start Up

```C
int KernelStart()
{
    // The hardware will invoke this function to boot up the system.
    // Start in physical address space, but by the end the kernel needs to switch to using virtual memory. (Needs to activate this).

    long totalRam = testPhysicalMemoryOfComputer();
    physicalMemoryPageTable kernelMemory = kernelInitializePhysicalMemory(PMEM_BASE, PAGESIZE, totalRam); // Calls SetKernelBrk(), setting things like *kernel_orig_brk

    //Init the kernel stack at KERNEL_STACK_BASE
    kernelInitializeStack(KERNEL_STACK_BASE, KERNEL_STACK_MAXSIZE, KERNEL_STACK_LIMIT);
    Processor.PC = &kernelMemory;

    //Initialization of page tables. Region 0 is kernel space, 1 is user. These are basically arrays with page table entries.
    pageTable ptesRegion0 = initVirtualAddressSpace(PMEM_BASE, VMEM_0_LIMIT, PAGESIZE); //Holds kernel state. Starts in physical mem.
    pageTable ptesRegion1 = initVirtualAddressSpace(VMEM_1_BASE, VMEM_1_LIMIT, PAGESIZE); //Maps each process. This is just the basic region, maybe for the first process that will be run.

    //Tell the MMU where to find these page tables:
    REG_PTBR0 = &ptesRegion0;
    REG_PTBR1 = &ptesRegion1; //All will be free until we use some of this process.

    REG_PTLR0 = ptesRegion0.numVirtualPages; //Total amount of memory in bytes / PAGESIZE
    REG_PTLR1 = ptesRegion1.numVirtualPages;

    BitVector freeFrames0 = ptesRegion0.unused(); //MAGIC HERE... Either a frame is free or not though [0,1]
    BitVector freeFrames1 = ptesRegion1.unused();



    // Initialize the page table entries and page table base and limit registers. We want to ensure that for every page of physical memory (pfn) in use by the kernel, that a page table entry is built
    // so that the new virtual page number vpn = pfn maps to physical page number pfn. (Yalnix pdf page 23)
    for (every pfn in physicalMemoryPageTable):
        buildPageTableEntry(pfn, *ptesRegion0); //Virtual page number mapping to physical page number.

    //Init the interrupt vector table. We write the address of the table to the REG_VECTOR_BASE. (Page 27)

    vectorTable interruptVectorTable = initVectorTable(TRAP_VECTOR_SIZE);
    REG_VECTOR_BASE = &interruptVectorTable;

    //Switch to virtual memory.
    WriteRegister(REG_VM_ENABLE, 1);

    // When this function returns, the machine begins running in user mode at a specified UserContext.
    Processor.flipModeBit(); // 0 to 1
    return 1; // Return into User mode.
}

```

## Context Switching

```C

KernelContext * KCSwitch(KernelContext * kc_in, void * curr_pcb_p, void * next_pcp_p)
{
    // Carries out context switches.

    // Saves proc A's kernel context.
    // Changes kernel stack page table entries for B
    // Returns saved kernel context for B.

    //On a context switch between processes, you need to change the mappings for the kernel stack and all of Region 1. (See Figure 4.1 and Chapter 4)

    KernelContext copy = KCCopy(*kc_in, *curr_pcp_p, NULL); //Simply copies the kernel context from kc_in into the new pcb
    if (copy != 0):
        return -1;
    curr_pcb_p->kernelContext = copy;

    return next_pcp_p -> kernelContext;

}

int KernelContextSwitch(KCSFunc_t * func, void * curr, void * next)
{
    //This function performs switching (in kernel-level) from one process to another AND it clones a new (kernel-level) process in the first place so there is something to switch to.

    // Switching from process A to process B:
    // 1st. Process A picks some other process B to run next. Process B, in kernel mode, is currently in suspended animation (stored away in a data structure is: The physical frame numbers containing B's kernel stack, and the kernel context that should be restored when B starts running again)
    // 2nd. Process A saves it's current state.
    // 3rd. Restore B's two pieces that are stored in the data structure.

    KernelContext temp = currentKernelContext; //Save the old context
    currentKernelContext = KernelContext magicStack; //Go to a "special magic stack" (Yalnix pdf 41)
    newContext = func(*temp, curr, next); //Call the switching function
    if (newContext == -1):
        return 1; //Failed
    currentKernelContext = newContext; //Resume running the Yalnix kernel with the context returned by the switch.

    return 0;
}

int scheduler()
{
    //This function runs a loop that continuously schedules and utilizes processes.

    //Create the Queues of Processes!
    queue ready;
    queue blocked;
    initqueue(&ready);
    initqueue(&blocked);

    State processor_state = Processor(); //Keep the state of the processor, including what processes it is running.

    while(ComputerIsON && Proccessor.notSuspended(): //Lol
        if(Processor.checkBlocked()): //Otherwise keep going. This is a very simple scheduler... Realistically we would put time bounds and what not here as well, in addition to other queues based on I/O.
            processorState = Processor.switching();
            Process nextProcess = dequeue(&ready);

            Process blockedProcess = Processor.Running();
            enqueue(&blocked, blockedProcess);

            KernelContextSwitch(*KCSwitch, *blockedProcess, *nextProcess);
            processorState = Processor.running(*nextProcess);

        if(blocked.head.ready()):
            enqueue(&ready, blocked.head);
            dequeue(&blocked, blocked.head);



}
```

## Memory Management

```C

void SetKernelBrk(void * addr))
{
    //Invoked when the kernal needs to adjust the kernel heap.
    if(REG_VM_ENABLE == 1):
        brk(addr); //Act like regular brk() but for userland
    else:
        difference = addr - _kernel_orig_brk; //Keep track of the difference
        brk(addr); // on the kernel heap
        //doSomethingWithDifference();


}

void flushTLB(int writingValue){
    // The OS needs to flush the Translation Lookaside Buffer. This occurs most often when performing a context switch from one process to another.

    // Write writingValue to the REG_TLB_FLUSH register given to us by the hardware.
    WriteRegister(REG_TLB_FLUSH, writingValue);
}


```
