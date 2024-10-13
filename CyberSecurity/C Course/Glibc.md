Glibc, or the GNU C library, is the library that gets baked into every program you compile with gcc. This concept is important to understand because it will be the foundation of every program you write, and as a result you should understand GNU C library functionality
## Why

When we write code, we write userland code, or code that exists in the context of a userspace process. To the CPU, this code is unpriveleged, and as a result it can't really do anything fancy, like allocate memory, access the filesystem, or access the network.

To do privleged things, we ask the kernel (which runs as privleged code) to do it for us. This is done through what is called a "system call interface" where the syscall instruction is ran, and asks the kernel to perform a certain action.

The GNU C library cleanly wraps all of this functionality up into easy to use functions that wrap the otherwise hard to maintain system call functionality

## Malloc

For example, when we allocate memory from the kernel for our process, we use malloc. Malloc internally calls the sbrk or mmap system calls, which asks the kernel through a system call to give us more memory. All of this abstracted away from us as the developer, and all we have to do is managed the result