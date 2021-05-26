# Syscalls
A System call is made when an application program explicitly requests an OS service. 


#include <ykernel.h>

```C
sys_fork(void){
	// how new processes are created 
	
	int pid_child;

	// malloc memory for child process
	
	if(not enough memory):
		//  free memory allocated
	    return -1; 
	//intialize child process
	
	rc = KernelContextSwitch(KCSwitch,(void*)  &currentpcb , (void*)  &nextpcb);
	
	return rc;

}


int sys_exec(char*filename, char**argvec){

//Replace the currently running program in the calling process's memory with filename 

	 int return_status;
	 
    if(filename==NULL):
	//check to see if file name is NULL
        return -1;

    return_status = LoadProgram(filename,argvec);

    if(return_status==-1):
	// make sure loadprogram did not fail
        return -1;
 
    return 0;
}



void sys_exit(int status){
	//terminate process
	
	pcb *temp;
	
	// If the initial process exits, you should halt the system.
	if(pcb->pid==0||pcb->pid==1)
        Halt
		
	free(pcb->children);
}


int sys_wait(int*statusptr){
     
	int rc;
	
	if(pcb->children == 0)
        return -1;
		
	 processor.blocked();
	rc = pcb->statusptr;
	 free(temp_status);
    return rc;
}


int sys_getpid(void){
	// retruns the process ID of the calling process
	return pcb->pid; 
	
}


int sys_brk(void*addr){
// allocating or deallocating enough memory to cover only up to the specified address.

	if(addr == NULL):
		return -1; 
		
   UPTOPAGE(addr)>>PAGESHIFT;
   
   return 0; 
   

}


int sys_delay(int clock_ticks){
// calling process is blocked until at least clock_ticks clock interrupts have occured after the call.

	if(clock == 0):
		return 0;
	if(clock_ticks >0 ):
		processor.blocked();
		//tell the processor to let another processs run 
		for(clock_ticks):
			interruptVectorTable[TRAP_CLOCK]
	else:
		return -1; 
}

```
