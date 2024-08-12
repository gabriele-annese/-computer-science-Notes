# What is a Buffer OverFlows

Buffer OverFlows as a common **software error** that can be exploited by malicious actor.
Also know as a **Buffer Overrun**, buffer overflows occurs when the amount of data in the **buffer exceeds** its storage capacity. 
***That extra data overflows into adjacent memory locations and corrupts or overwrites the data in those location***

# Process Layout

**In computer all programs are runs as a processes**. The current computer architecture allow run multiple processes **concurrently**. While these processes may appear at the same time, the computer actually switches between the processes very quickly and makes this like the are running at the same time, this ability is called **switch context**.

The memory context layout:
![[Pasted image 20240812225930.png]]

- **User Stack:** This section contains information required to start a process. This information would include the current program, saved register and more information. The after section are unused memory and it used in case the stack grows.

- **Shared Library:** This section are used to stored statically/dynamically link libraries used by a program. 

- **Heap:** This section increase and decrease depending about the program dynamically assigns memory. The section above is unused and it used in case the size of heap increase.
- **Data:** The program code and data stores the program executable and initialized variables.
# x86-64 Procedures

To tracking all information in a program as example which function has been called and which data is passed from one function to another. For that is used the **Stack region**.

> NOTE
> The top of the stack is at the **lowest** memory address and the **stack grows towards lower memory addresses**.

The most common operations of the stack are 
- **push:** used to add data onto the stack
- **pop:** used to remove data from the stack
![[Pasted image 20240812232905.png]]

### PUSH
The `push var` are **assembly instruction** to push a value onto the stack. 
- Uses var or value stored in memory location of var 
![[Pasted image 20240812233251.png]]

- **Decrements** the stack pointer(know as `rsp`) by 8
- Writes above value to new location of `rsp`, which is now the top of the stack.

![[Pasted image 20240812233303.png]]

### POP
The `pop var` is an **assembly instruction** to read a value and pop it off the stack. It does the following:

- Reads the value at the address given by the stack pointer

![[Pasted image 20240812233708.png]]
Stack Top(memory location 0x0)(**rsp** points here)

- Store the value that was read from `rsp` into var
- Increment the stack pointer by 8

![[Pasted image 20240812233855.png]]

> NOTE
> It's important to note that the memory doesn't change when popping the values of the stack, it is only the value of the stack pointer that changes

### Stack Frame
Each compiled program may include multiple functions, where each function would need to stored local variables, arguments passed to the function and more. To make this east to manage **each function has it own separate stack frame**, where each new stack frame is allocated when a function is called and deallocated when the function is complete.

![[Pasted image 20240812234638.png]]

Code example for stack frame
```c
int add(int a, int b){
   int new = a + b;
   return new;
}


int calc(int a, int b){
   int final = add(a, b);
   return final;
}

calc(4, 5)
```
