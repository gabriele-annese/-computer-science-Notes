
## For Loops
A for loop is created with 3 clauses: the initialization clause, the check clause, and the iteration clause. The initialization clause is ran once when the loop starts, the check clause checks if the loop should keep going, and the iteration clause runs on every loop completion. Check it out here:
```C
int i = 0;

for (i = 0; i < 32; i++){
	//do something
	ids[i] = 0;
}
```

This loop will set i to zero at start, and run while is less than 32, and increase i every time. We can also use i inside that for loop to do some logic as a function of the iterator.
## While loops
A while loop is really simple, it just runs the program inside the loop while the condition is true.

```C
int i = 0;
while(i<32){
	//do something
	i++;
}
```

>NOTICE
 In the while loop you're required to increase the iterator yourself, otherwise you'll have an infinite loop.

## Do-while Loop

A do-while loop is the same as the while loop, but the code inside the loop will **ALWAYS** execute once, regardless of the condition.
```C
int i = 0;
do{
	//do something
	i++;
} while(i<32);
```

