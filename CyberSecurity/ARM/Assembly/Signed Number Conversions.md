
## Type

In arm assembly we have tree type of data
- `Byte`: 8 bit
- `Halfword`: 16 bit
- `Word`: 32 bit, the most used

## Processor's registers
One register have 32 bit (word type) in hexadecimal format. One digit in hexadecimal is called `nibble` (4 bit).


## Conversion in binary

If i have a negative number all left bit are represent like `1`

```assembly
.global _start
_start:
	
	mov r0, #55
	mov r1, #-44
	
```

![[Pasted image 20241016232036.png]]


![[Pasted image 20241016232059.png]]

### Convert positive number

Now i have the `0x00000037` hexadecimal number, to convert that i use this schema 

```
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
G = 16

2^n
n	5 4  3 2 1 0
result 32 16 8 4 2 1


Hex: 00000037
Binary: 0000 0000 0000 0000 0000 0000 0011 0111
```

![[Pasted image 20241016233032.png]]

### Convert negative number

Now i have the `0xffffffd4` hexadecimal number, to convert that i use this schema 

whit this schema i obtain a binary number

```
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
G = 16

2^n
n	5 4  3 2 1 0
result 32 16 8 4 2 1


Hex: ffffffd4
Decimal 15-15-15-15-15-15-13-4
Binary: 1111 1111 1111 1111 1111 1111 1101 0100
```
![[Pasted image 20241016234543.png]]
as mentioned previously the left bit are 1 because is a negative number.

To covert this number in a positive number i need to `flip all left bits from the last positive bit to tge right `.

![[Pasted image 20241016234633.png]]
 

```
A = 10
B = 11
C = 12
D = 13
E = 14
F = 15
G = 16

2^n
n	5 4  3 2 1 0
result 32 16 8 4 2 1


Hex: ffffffd4
Decimal 15-15-15-15-15-15-13-4
Binary: 1111 1111 1111 1111 1111 1111 1101 0100
Binary-Positive: 0000 0000 0000 0000 0000 0000 0010 1100
```

SBOM 💣💣
![[Pasted image 20241016234711.png]]
