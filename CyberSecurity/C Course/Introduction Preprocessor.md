## What is Preprocessor?
The C **preprocessor is the VERY first** step in converting C to code code that can be ran by the computer

## Compilation Process
The process of converting source code to machine code is the following steps:
- Preprocessor
- Compilation
- Assembly
- Linking
## Preprocessor
In the preprocessor step the compiler looks over all of the code and runs macros on lines in the code. Some of these macros are the following:

- `#define`: Defines a macro.
    - Example: `#define PI 3.14`
- `#undef`: Undefines a previously defined macro.
    - Example: `#undef PI`
- `#include`: Includes a file.
    - Example: `#include <stdio.h>`
- `#if`: Conditional compilation directive.
    - Example: `#if DEBUG`
- `#ifdef`: Checks if a macro is defined.
    - Example: `#ifdef DEBUG`
- `#ifndef`: Checks if a macro is not defined.
    - Example: `#ifndef DEBUG`
- `#elif`: Else if condition for `#if`.
    - Example: `#elif DEBUG == 2`
- `#else`: Else condition for `#if`.
    - Example: `#else`
- `#endif`: Ends a `#if`, `#ifdef`, or `#ifndef` block.
    - Example: `#endif`
- `#error`: Generates a compiler error with a specified message.
    - Example: `#error "Error message"`
- `#pragma`: Specifies diverse behavior depending on compiler.
    - Example: `#pragma once`
- `#line`: Specifies the original line number and filename.
    - Example: `#line 20 "myfile.cpp"`
- `#warning` (GCC): Generates a compiler warning with a specified message (GCC).
    - Example: `#warning "Warning message"`

## Example 
```c
#include <stdio.h>
// you know what to do :)

int main() {

    printf("Hello World! \n");

}
```

- `#include <stdio.h>` a line of preprocessor that includes a header file to give us access to other code

- `int main() {` a function definition that is required for our code to run

- `printf("Hello, World!\n");` a function call that prints something to the screen

- `return 0;` a line of code that ends our programs currently running function

- `}` the end of the function