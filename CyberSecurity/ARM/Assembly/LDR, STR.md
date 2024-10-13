# LDR
LDR is a command to load from the memory address and store in a register
```assembly
.global _start

.text
_start:	
	ldr r0, =var1

.data
var1: .word 5
var2: .word 4
```

![[Pasted image 20241013235325.png]]

Now to store the value of var1 using the ``

```assembly
.global _start

.text
_start:	
	ldr r0, =var1
	ldr r1, [r0] @store the var1 value

.data
var1: .word 5
var2: .word 4
```

![[Pasted image 20241013235840.png]]

# STR
Store Register (STR) calculate the value from a register and store than in memory
```assembly
.global _start

.text
_start:	
	mov r0, #2
	ldr r1, =var2 @load from the memory address of var2
	str r0, [r1] @take value from r0 and store than in r1 address memory of var2
	
	

.data
var1: .word 5
var2: .word 4
```

![[Pasted image 20241014001131.png]]![[Pasted image 20241014001510.png]]
![[Pasted image 20241014001534.png]]
