compare subtract an optionally-shifed register value. It updates the condition flags based on the result, and discards the result


![[Pasted image 20241020001036.png]]

## Negative Example
```assembly
.global _start
_start:
	
	mov r0, #5
	mov r1, #6
	
	/*
	if r0 > r1 -> result = +
	if r0 < r1 -> result = -
	if r0 == r1 -> result = 0
	*/
	
	cmp r0,r1
```

compare the bit result with the bit schema above

![[Pasted image 20241020001427.png]]


![[Pasted image 20241020001438.png]]


## Positive Example

```assembly
.global _start
_start:
	
	mov r0, #6
	mov r1, #5
	
	/*
	if r0 > r1 -> result = +
	if r0 < r1 -> result = -
	if r0 == r1 -> result = 0
	*/
	
	cmp r0,r1
```


![[Pasted image 20241020001551.png]]![[Pasted image 20241020001606.png]]

## Equals Example
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
```

![[Pasted image 20241020001746.png]]

![[Pasted image 20241020001802.png]]
