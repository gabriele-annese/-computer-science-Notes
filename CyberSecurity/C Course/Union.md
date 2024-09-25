Unions are a field that assign multiple labels of multiple types to the same memory location.

You can create a structure with the following syntax.
```C
union myunion { 
	int i; 
	char c;
};
```

This will create a type called `union myunion` that we can use later on in the code. This structure has two members `i` and `c`, but both of them will contain data at the same location. The union is only the size of the largest element, in this case `int i`, or 4 bytes.

## Unions in Practice

```C
union myunion { 
	int i; 
	char c;
}; 
int main() { 
	 union myunion u; 
	 u.i = 0x41424344;
	 printf("%c"\n, u.c)
}
```

Above, `u.c` will equal `0x44`, which is the value in the location of `i`.
