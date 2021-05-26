# Traps
A `trap` is a synchronous interrupt caused by an 'exceptional' condition. 
The following pseudocode is a representation of `trap` processing. With the various `trap handlers` provided below. 

#### Generic Trap Process
```c
if trap_is_triggered:
	// Suspend the processor
	Processor.suspend();

	// Store the current state of the processor: program counter, 
	// values of registers, page table mappings, etc. 
	State processor_State = Processor.saveCurrentState(); // push to stack
	
	// Flip 'mode bit' to enter Kernel Mode, allowing execution in
	// the privileged mode.  
	Processor.flipModeBit(); // 1 to 0
	
	// Switch based on the interrupt code that the trap triggered
	// and call the appropriate handler. The following example traps
	// were taken from the YALNIX manual.
	// Interrupt Vector
	switch (interupt_code):
		case TRAP_KERNEL:
			trapKernelHandler(UserContext.code); 
		
		case TRAP_CLOCK:
			trapClockHandler();
		
		case TRAP_ILLEGAL:
			trapIllegalHandler();
			
		case TRAP_MEMORY:
			trapMemoryHandler(UserContext.addr);
		
		case TRAP_MATH:
			trapMathHandler();
			
		case TRAP_TTY_RECEIVE:
			trapTtyReceiveHandler(UserContext.code);
		
		case TRAP_TTY_TRANSMIT:
			trapTtyTransmitHandler(UserContext.code); 
			
	// Flip 'mode bit' to enter User Mode when finished.   
	Processor.flipModeBit(); // 0 to 1
```
#### Trap Handlers
```c
void trapKernelHandler(UserContext.code) {
    requested_syscall = UserContext.code;
	// Execute the requested syscall
	requested_syscall.execute();
}
```
```c
void trapClockHandler() {
	if (process_ready_queue != null) {
	   // if there are other runnable processes on the ready queue, perform
	   // a context switch to the next runnable process.
		Process.context = ready_queue.next();
	}
	else {
	    // dispatch idle
	    Process.context = context.idle;
	}
}
```
```c
void trapIllegalHandler() {
    // abort the currently running process 
    Process.current_context.abort();
}
```
```c
void trapMemoryHandler(UserContext.addr) {
    if (UserContext.addr exists in allowed region) {
        // if an implicit request, enlarge the process's stack to "cover" the 
        // address being referenced that caused the exception.
        Stack.push(UserContext.addr);
    }
    else {
        // abort the currently running process 
        Process.current_context.abort();
    }
}
```
```c
void trapMathHandler() {
    // abort the currently running process 
    Process.current_context.abort();
}
```
```c
void trapTtyReceiveHandler(UserContext.code) {
    // a new line of input is available from the terminal
    boolean isNecessary = TtyReceive(UserContext.code); // hardware operation
    
    // ttyReceive() is not actually a boolean (maybe bit switch?), but some 
    // information from that result is used to check if the next step is necessary. 
    if (isNecessary) {
        ttyRead(); // specific kernel call 
    }
}
```
```c
void trapTtyTransmitHandler(UserContext.code) {
    // The kernel should complete the blocked process that started this terminal output
    while (Process.isBlocked()) {
        TtyWrite(UserContext.code);
    }
}
```