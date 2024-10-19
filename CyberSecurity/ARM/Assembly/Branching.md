Most ARM instructions, and most Thumb instructions from ARMv6T2 onwards, can be executed conditionally, based on the values of the APSR condition flags. Before ARMv6T2, the only conditional Thumb instruction was the 16-bit conditional branch instruction. [Table 8.1](https://developer.arm.com/documentation/ddi0406/c/Application-Level-Architecture/Instruction-Details/Conditional-execution?lang=en#Chdcgdjb "Table 8.1. Condition codes") lists the available conditions.

![[Pasted image 20241020010440.png]]


Code example
```assembly
.global _start
_start:
	
	mov r0, #6
	mov r1, #6
	
	/*
	if r0 > r1 -> result = +
	if r0 < r1 -> result = -
	if r0 == r1 -> result = 0
	*/
	
	cmp r0,r1
	beq cond1  //if the cmp return equals jump to cond1 else go in condition2
	b cond2
	
	
cond1:
	mov r0, #2
	b end       // skip cond2
	
cond2:
	mov r1, #4
```


![[Pasted image 20241020010935.png]]

example != input
![[Pasted image 20241020011726.png]]
