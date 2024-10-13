## Strings
In a C structure string can gets people in trouble a lot. Tons of security vulnerabilities in modern software are caused by ***buffer overflows*** and most of those buffer overflows have to do with strings.

## What is a string
A string is just a array of characters. Example of that it possible to see in this code

```C
char my_str[] = {'h', 'e', 'l', 'l', 'o'}; 
printf("%s\n", my_str);
```


## String are special

A string in C has a very special characteristic that makes it amazing, but also dangerous. ***All string in C ended with a null byte***. If the don't end in a null byte the operation you'll perform still going.

## Why does that matter?
If you run operation like strcpy, it will ONLY STOP COPYING OR PRINTING IF IT ENCOUNTERS THAT ZERO BYTE. So the code above actually has a pretty major security vulnerability. You have to make sure your strings end with a zero otherwise thing will go sideways.

```C
char my_str[] = {'h', 'e', 'l', 'l', 'o', 0}; 
printf("%s\n", my_str);

// if u use the syntax below the null byte is set by default 
char *myotherstr = "hello";

```

# Exercise
### Objective

In this exercise, we'll learn about two of the syntax for strings in C. In the C programming language, we can use various forms of syntax to decide where in the program our string memory is allocated.

### Task

Create a copy of the string "hehe" using hex syntax, and ensure that they are the same strings!

To do this, we have a few options. We can use the `char *<name>="<string>";` syntax, **which puts the value in the .rodata** section of the ELF.

We can also use the `char <name>[] = "string";`, which puts the value on the stack. Also, we can use `char <name>[] = {hexvalues};` to do this.

```C
#include <stdio.h>

int main(int argc, char **argv) {
    // this is a string
    char *str = "hehe";
    
    // create the same string
    // as a character array
    char otherstr[] = {'h','e','h','e',0};
    
    if (!strcmp(str, otherstr)) {
        printf("Yay!\n");
    } else {
        printf("Nay!\n");
    }

    return 0;

}
```