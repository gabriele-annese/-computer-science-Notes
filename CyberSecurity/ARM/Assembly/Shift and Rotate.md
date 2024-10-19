# Logical Shift Left
the `lsl` operate shift left the bits 

```
lsl <registerDes>, <registerSource>, <ValueShift>
```

Example
```assembly
.global _start
_start:
	
	mov r0, #40
	
	lsl r0, r0, #1
    lsl r0, r0, #1
	lsl r0, r0, #1
	lsl r0, r0, #1
    lsl r0, r0, #1
	lsl r0, r0, #1	
	lsl r0, r0, #1
    lsl r0, r0, #1
	lsl r0, r0, #1	
```

![[Pasted image 20241019230150.png]]
Hex to binary for `0x28` value
```
0x28 = 00000000 00000000 00000000 00101000
```

Step over and the in r0 now is `0x50`
![[Pasted image 20241019230530.png]]
Hex to binary for `0x50` value

```
0x28 = 00000000 00000000 00000000 00101000
0x50 = 00000000 00000000 00000000 01010000
```

the bits shift left to one position.

Ok cool but in practice? So in practice when shift left the bit we are multiplicate by 2 the number.
Process in decimal 
![[Pasted image 20241019230930.png]]

![[Pasted image 20241019230956.png]]

![[Pasted image 20241019231010.png]]




# Logical Shift Right
its the same of the `lsl` but now we divide the number.


Example
```assembly
.global _start
_start:
	
	mov r0, #40
	
	lsr r0, r0, #1
    lsr r0, r0, #1
	lsr r0, r0, #1
	lsr r0, r0, #1
    lsr r0, r0, #1
	lsr r0, r0, #1	
	lsr r0, r0, #1
    lsr r0, r0, #1
	lsr r0, r0, #1	
```

![[Pasted image 20241019231635.png]]

 
```
0x28 = 00000000 00000000 00000000 00101000
```



![[Pasted image 20241019231729.png]]


```
0x28 = 00000000 00000000 00000000 00101000
0x14 = 00000000 00000000 00000000 00010100
```

in decimal
![[Pasted image 20241019231828.png]]
![[Pasted image 20241019231836.png]]


# Arithmetic Shift Right
Its the same but for negative number


Example
```assembly
.global _start
_start:
	
	mov r0, #40
	
	asr r0, r0, #1
    asr r0, r0, #1
	asr r0, r0, #1
	asr r0, r0, #1
    asr r0, r0, #1
	asr r0, r0, #1	
	asr r0, r0, #1
    asr r0, r0, #1
	asr r0, r0, #1	
```

![[Pasted image 20241019232515.png]]

```
0xffffffd8 = 11111111111111111111111111011000
```

![[Pasted image 20241019232456.png]]
```
0xffffffd8 = 11111111111111111111111111011000
0xffffffec = 11111111111111111111111111101100
```

in decimal
![[Pasted image 20241019232631.png]]
![[Pasted image 20241019232638.png]]


# Rotate Instruction
rotate instruction `ror` take the least bit and but on the start of sequence


```assembly
.global _start
_start:
	
	mov r0, #-40
	
	ror r0, r0, #1
    ror r0, r0, #1
	ror r0, r0, #1
	ror r0, r0, #1
    ror r0, r0, #1
	ror r0, r0, #1	
	ror r0, r0, #1
    ror r0, r0, #1
	ror r0, r0, #1	
```

![[Pasted image 20241019233231.png]]
