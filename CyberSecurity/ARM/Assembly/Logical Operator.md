# AND
Boolean AND returns true when two value are true

```text
AND Truth Table
A B Result
0 0 0
0 1 0
1 0 0
1 1 1
```

![[Pasted image 20241014232012.png]]
![[Pasted image 20241014232026.png]]

## Binary explanation

```binary
0000000000000000000000001000010 0x42 value
0000000000000000000000000100110 0x26 value
0000000000000000000000000000010 0x2 result value
```


# ORR
Boolean OR returns true when at least one input is true
```text
ORR Truth Table
A B Result
0 0 0
0 1 1
1 0 1
1 1 1
```

```assembly 
.global _start
_start:
	
	mov r0, #0x42
	orr r1, r0, #0x26
```

![[Pasted image 20241014231255.png]]

![[Pasted image 20241014231309.png]]


### Binary explanation

```
0000000000000000000000001000010 0x42 value
0000000000000000000000000100110 0x26 value
0000000000000000000000001100110 0x66 result value
```



# EOR
Exclusive OR returns true if the one input is true and the other is false
```
EOR Truth Table
A B Result
0 0 0
0 1 1
1 0 1
1 1 0
```

![[Pasted image 20241014232637.png]]

### Binary explanation
```
0000000000000000000000001000010 0x42 value
0000000000000000000000000100110 0x26 value
0000000000000000000000001100100 0x64 result value
```


# MVN
Move negation "flip" the bit

```text
MVN Truth Table
A Result
0 1 
1 0  
```

![[Pasted image 20241014233416.png]]


## Binary explanation
```binary
00000000000000000000000001000010 0x42 value
11111111111111111111111110111101 0xFFFFFFBD result value
```
