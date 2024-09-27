A pointer is a variable who's value equals the address of another variable.

## Creating Pointers

To make a variable, first you need another variable that you want to point to. Lets say we have an integer that we'd like to reference.
```C
int x = 3;
```

To create a pointer that points to `x`, all we'd have to do is create a pointer and set it's value equal to the address of `x`. We can do that with the following syntax.

```C
int x = 3;
int *pX = &x;
```

Here, the syntax is as follows. The `*` character notes that the type is a pointer. The `&` character gets the "address of", which gets the address of `x` and puts that value in `pX`.

## Using Pointers

To use a pointer, all you have to do is "**dereference**" it. To dereference a pointer, simply add the `*` to the name of the pointer to get the value at that location.

```C
int x = 3;
int *pX = &x;

printf("%d\n", *pX);
```

## Next

Move on to the next lesson where we talk about pointers and how to use dynamic memory allocation via the heap.