In previous examples we showed how to allocate memory via an array or create memory space using structures. The problem with both of these solutions to allocating memory is that their sizes are static, and have to be known at compile time. Because of this, we end up allocating too much space and not being able to allocate any more if our program grows.

# The Heap

The heap is a data region in ELF (Linux) memory that is managed by a memory allocator. Instead of using statically defined arrays or structures, we can ask the memory allocator via a function called malloc to get us a certain amount of memory. For example: this code allocates 64 bytes of memory.

```C
malloc(64);
```

## Allocating Memory

To use this memory, we need to assign it to variable for use later. Also, its important that we check the return value from malloc to make sure that we got a value from the allocator. Sometimes, allocators fail.

```C
struct employee_t *my_employee = malloc(sizeof(struct employee_t)); 
if (my_employee == NULL) {
	printf("Something went wrong\n"); 
	return -1; 
}
```

## Freeing Memory

We MUST give the memory back to the system. If we fail to free memory that we allocate, over time we will leak memory, and could possibly use up all available system memory and eventually have our process be killed by the operating system kernel. To avoid this, free your memory like so:

```C
... 
free(my_employee); 
my_employee = NULL;
```
It's not required, but always a smart idea to set used pointers to NULL. By setting pointers to NULL, we avoid what is called a dangling pointer, or a pointer that points to memory that is no longer valid