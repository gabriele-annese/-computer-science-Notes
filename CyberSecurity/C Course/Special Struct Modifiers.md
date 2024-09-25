In a perfect world, structs look the same across all systems. However, when optimizing for spacing and alignment, this isn't always the case. To fix this, we can use struct attributes to make the structures more predictable.

# Structures again
Structures are areas of memory that organize variables of multiple types into a logical unit.

## Whats the problem?

If wanted to serialize this data and work on it between two different systems that might have different architectures, there's a chance that the structures may change between these systems. By adding special modifiers to the structures, we can ensure that the structure does not get modified in this way.

You can create a structure with the following syntax.

```C
struct mystruct {
	int i; 
	char c; 
};
```

Here, the structure should only be 5 bytes. But, there is a high chance that the structure is actually 8 bytes, or even 16 bytes due to alignment issues.

To fix this, we can add the `packed` attribute.
```C
struct __attribute__((__packed__)) mystruct {  
	int i; 
	char c;
};
```
This will create the same struct, but ensure that the compiler doesn't add any special sauce in between the elements so we can ensure it's the same size on multiple systems.

