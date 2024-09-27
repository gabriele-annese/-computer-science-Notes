In our previous lesson, we talked about how memory can be taken from the system using calls to an allocator, like malloc for example. But, we also discussed how bad things can happen if you don't free the memory after.

In this section we'll discuss how to check your code for memory leaks using a tool called Valgrind.

# Valgrind

Valgrind is a program that wraps the calls to malloc and free, tags them, and checks to see where memory is leaked or potentially lost.

## Setup your program

To make your code compatible with valgrind, you'll need to compile with debug symbols in your code. Do this the following way.

```bash
gcc -o program code.c -g
```

The -g flag will add the debug symbols to your code. To test your code, run

```bash
valgrind --leak-check=full ./program
```
Valgrind will run your code and provide a report of what memory did not get freed, and where the allocation happened.

## Code Coverage

Please note that you need to write tests to make your code execute lines that allocate from the heap to use valgrind correctly. If your code never allocates from the heap in your test case, it'll never see the memory leak.