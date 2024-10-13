# errno

If a function that you call returns the error value for that function as described in the RETURN VALUE section of the man page, we can ask the function what specifically went wrong by asking for the errno. Errno is a global value in glibc that is used to give the developer information about what went wrong when a function fails.

## man pages again

At the bottom of the man page is the description for all of the errors that can occur in every function. Check out the following example.

```C
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

  

int main(int argc, char *argv[]) {
    int fd = open("./file-that-doesnt-exist", O_RDONLY);
    if (fd == -1) {
        perror("open"); // return error for open faction
        return -1;
    }
}
```

![[Pasted image 20241007233314.png]]
Here, we write some code that uses a glibc function open to try to open a file. This will ultimately fail. We can use the `perror` function to ask glibc to print out the error string for what specifically went wrong with that function call so that we can take corrective action. We can also check the manpage to read about what could have went wrong based on the `errno` value.

