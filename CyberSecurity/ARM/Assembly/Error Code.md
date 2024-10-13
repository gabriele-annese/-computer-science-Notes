# Introduction

This code is an example of error code exception write in ARM assembly. To achive that we need to set the error number in **r0** register, set the value **1** in **r7** register to call the syscall **int error_code**, [reference here](https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md#arm-32_bit_EABI).

## Code
```arm

.global _start

.section .text

  

_start:
    mov r0, #60     @ Move number 60 in r0 register

    mov r7, #1      @ Move muber 1 in r7 register for trigger exception

    swi 0           @ Software interrupt (SWI) need to be call kernle exception
```
  

## Execution
![[Pasted image 20241013220319.png]]