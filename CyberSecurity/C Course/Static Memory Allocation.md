Sometimes we want data to exist in the scope of the lifetime of our entire program. For example, what if we want to track the amount of employees that have been initialized in our database so far? We have two options: global and static variables.

# Static Variables

A static variable is a variable that exists in a static lifetime, but does not have global scope. By creating a static variable, we are able to create variables that out live the lifetime of a function call and track data over multiple calls to the same function.

For example
```C
int my_static_test() {
	static int i = 0;
	i++;
}
```

In the above function, i will be set to zero the first time the function is ran. When this happens, the variable's data is actually stored not on the stack, but somewhere in the data section of the program that is running. Later calls to my_static_test will increase the variable every time, so it will be `1`, then `2`, then `3`.

## Static Use Case

Let's say for example we wanted to track how many times a particular function was called, maybe the employee_initialize function for example.

```C
int employee_initialize (employee_t) {
	static int n_called = 0; 
	... 
	return n_called++;
}
```

The above code will increment the variable every time the `employee_initialize` function was called, and return the current value to the user.