## Introduction

This lesson will focus on understanding double pointers through an example of updating a pointer via `realloc` in a function called `foo`.

## Understanding Double Pointers

A double pointer is a pointer that points to another pointer. In memory management, double pointers are particularly useful for functions that need to modify the original pointer, such as allocating or resizing dynamic memory.

### Basic Concept

- **Pointer**: A variable that holds the address of another variable.
- **Double Pointer**: A variable that holds the address of a pointer.

### Example
```C
int main() {
    int *ptr;
    int **dptr;

    ptr = (int *)malloc(sizeof(int));
    dptr = &ptr;

}
```

In this example:

- `ptr` is a pointer to an `int`.
- `dptr` is a double pointer to `ptr`.

## Using Double Pointers with `realloc`

### Problem Statement

We want to create a function `foo` that resizes an array using `realloc`. The function should accept a double pointer to the array so that it can modify the original pointer.

### Example C Code

```C
#include <stdio.h>
#include <stdlib.h>
  

typedef enum {
    STATUS_GOOD,
    STATUS_BAD,
} status_t;

  

status_t foo(int **arr, size_t new_size) {
    int *temp = realloc(*arr, new_size * sizeof(int));

    if (temp == NULL) {
        // Handle realloc failure
        printf("Memory allocation failed\n");
        return STATUS_BAD;
    }

    *arr = temp;
    return STATUS_GOOD;
}

  

int main() {
    size_t initial_size = 5;
    size_t new_size = 10;

    int *arr = malloc(initial_size * sizeof(int));
    if (arr == NULL) {
        printf("Memory allocation failed\n");
        return 1;
    }
    
    // Initialize the array
    for (size_t i = 0; i < initial_size; i++) {
        arr[i] = i;
    }

    // Print the initial array
    printf("Initial array:\n");
    for (size_t i = 0; i < initial_size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    // Call foo to resize the array

    if (STATUS_BAD == foo(&arr, new_size)) {
        printf("uh oh");
        return -1;
    }

    // Initialize the new part of the array
    for (size_t i = initial_size; i < new_size; i++) {
        arr[i] = i;
    }

    // Print the resized array
    printf("Resized array:\n");
    for (size_t i = 0; i < new_size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    free(arr);
    return 0;

}
```