# C

TODO: description
Notas retiradas do livro `Extreme C`

## Indexing arrays with char

The index of an array is an integer. In this case c is a character so you cannot have c as index of the array: the ASCII value of a character is often much larger than the indices 0 till 9 in this array.

So to map the character '0' to index 0, the character '1' to index 1 and so on, you have to subtract '0' from the character. That means you'll get 0 for the character '0', 1 for the character '1' and so on, as intended.

```c
int main() {
    int ndigit[10];

    //initialize
    for (i = 0; i< 10; ++i)
        ndigit[i] = 0;

    while ((c = getchar()) != EOF) {
        if (c >= '0' && c <= '9' )
            ++ndigit[c-'0'];
    }

}
```

In ASCII, the character '0' is at position 48. The standard guarantees that in the character encoding the number must be sequential. That is, just like 1 comes after 0, '1' will come after '0'

Example:
    if the user enter the character '1' (number 49 in the ASCII table), the expression `ndigit[c-'0']` would be `ndigit[49-48]`


## Relocatable object files
Relocatable object files are the output of the assembly step in the C compilation pipeline. These files are considered to be temporary products of a C project, and the are the main ingredients to produce further and final products.

In a relocatable object file, we can find the following items regarding the compiled translation unit:
- The machine-level instructions produced for the functions found in the translation unit (code).
- The values of the initialized global variables declared in the translation unit (data).
- the *symbol table* containing all the defined and reference symbols found in the translation unit.

### generate an object file:
```c
int add(int a, int b){
    return a+b;
}
```
```sh
gcc -c add.c -o add.o
```

### Some tools to inspect object files:
- `nm` to list symbols
```sh
nm add.o
```
will output:
```
0000000000000000 T add
```
- `readelf` to display symbol table
```sh
readelf -s add.o
```
will output:
```
table '.symtab' contains 9 entries:
Num:    Value          Size Type    Bind   Vis      Ndx Name
0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS add.c
2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1
3: 0000000000000000     0 SECTION LOCAL  DEFAULT    2
4: 0000000000000000     0 SECTION LOCAL  DEFAULT    3
5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5
6: 0000000000000000     0 SECTION LOCAL  DEFAULT    6
7: 0000000000000000     0 SECTION LOCAL  DEFAULT    4
8: 0000000000000000    20 FUNC    GLOBAL DEFAULT    1 add
```
- `objdump` display assembler contents of executable section
```sh
objdump -d add.o
```
will output:
```
add.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <add>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	89 75 f8             	mov    %esi,-0x8(%rbp)
   a:	8b 55 fc             	mov    -0x4(%rbp),%edx
   d:	8b 45 f8             	mov    -0x8(%rbp),%eax
  10:	01 d0                	add    %edx,%eax
  12:	5d                   	pop    %rbp
  13:	c3                   	retq
```

## Static Libraries
In Unix systems, static libraries are usually named according to an accepted and widely used convention. The name starts with `lib`, and it end with the `.a` extension. This can be different for other operating systems; for instance, in Microsoft Windows, static libraries carry the `.lib` extension.

To create a static library file, we simply need to run the following command:
```sh
ar crs libexample.a aa.o bb.o cc.o ...
```
As a result, libexample.a is created, which contains all of the preceding relocatable object files as a single archive

Note:
>The ar command does not necessarily create a `compressed` archive file, It is only used to put files together to a form a single file that is an archive of all those files. The tool ar is general purpose, and you can use it  to put any kind of files together and create your won archive out of them.

Using the `ar` program and passing the option `t` we can see the content of the archive file:
```sh
ar t libexample.a
```
will out put:
```
aa.o
bb.o
cc.o
...
```

When using a C library, we need to have access to the declarations that are exposed by the library together with its library file. The declarations are considered as the public interface of the library, or more commonly, the API of the library.

We need declarations in the compile stage when the compiler needs to know about the existence of types, function signatures, and so on. Header files serve this purpose.

When it comes to using the static library, we need to write a new source file includes the library's API and make use of its functions.


## Shared Libraries
While the static library files have a `.a` extension in their names, the shared object files carry the `.so` extension im most Unix-like ystems. In macOS, the have the .dylib extension.

```sh
gcc -c trigon.c -fPIC -o trigon.o
gcc -c g2d.c -fPIC -o 2d.o
gcc -c g3d.c -fPIC -o 3d.o
```

Looking at the commands, you can see that we have passed an extra option, `-fPIC`, to gcc while compiling the sources. This option is *mandatory* if you are going to create a shared object file out of some relocatable object files. PIC stands for position independent code. As we explained before, if a relocatable object file is position independent, it simply means that the instructions within it don't have fixed addresses. Instead, they have a relative addresses; hence they can obtain different addresses in different processes. This is a requirement because of the way we use shared object files.

The following command shows how you should create a shared object file out of a number of relocatable object files that have been compiled using the `-fPIC` option:
```sh
gcc -shared 2d.o 3d.o trigon.o -o libgeometry.so
```

## Static Memory Layout
The memory layout of an ordinary process is divided into multiple parts. Each is called a *segment*. Each segment is a region of memory which has a definite task and it is supposed to store a specific type of data. You can see the following list of segments being part of the memory layout of a running process:
  - Uninitialized data segment or **Block Started by Symbol (BSS)** segment
  - Data segment
  - Text segment or Code segment
  - Stack segment
  - Heap segment


The `size` command can be used to print the static memory layout of an executable object file.

You can see the usage of the `size` command in order to see the various segments found as part of the static memory layout as follows:
```sh
size a.out
```
will output:
```
   text	   data	    bss	    dec	    hex	filename
   1282	    512	      8	   1802	    70a	main.out
```

### BSS
BSS stands for **Block Started by Symbol**. Historically, the name was used to denote reserved regions for uninitialized words. Basically, that's the purpose we use the BSS segment for; either uninitialized global variables or global variables set to zero.

### Data Segment
Data segment is used to store the initialized global variables set to a non-zero value. Other than global variables, we can have some static variables declared inside a function. These variables retain their values while calling the same function multiple times. These variables can be stored either in the Data segment or the BSS segment depending on the platform and whether they are initialized or not.

### Text Segment

## Dynamic memory layout
The dynamic memory layout is actually the runtime memory of a process, and it exists as long as the process is running. When you execute an executable object file, a program called loader takes car of  the execution. It spawns a new process and it creates the initial memory layout which is supposed to be dynamic. To form this layout, the segments found in the static layout will be copied from the executable object file. More than that, two new segments will also be added to it. Only then can the process proceed and become running.

In short, we expect to have five segments in the memory layout of a running process. Three of these segments are directly copied from the static layout found in the executable object file. The two newly added segments are called Stack and Heap segments. These segments are dynamic, and they exist only when the process is running. This means that you cannot find any trace of them as part of the executable object file.

### Stack
  - Stack memory has a limited size; therefore , it is not a good place to store objects.
  - The addresses in Stack segment grow backward, therefore reading forward in the Stack memory means reading already pushed bytes.
  - Stack has automatic memory management, both for allocation and deallocation.
  - Every Stack variable has a scope and it determines its lifetime. You should design your login based on this lifetime. You have no control over it.
  - Pointers should only point to those Stack variables that are still in a scope.
  - Memory deallocation of Stack variables is done automatically when the scope is about to finish, and you have no control over it.
  - Pointers to variables that exists in the current scope can be passed to other functions as arguments only when we are sure that the current scope will be still in place when the code in the called functions is about to use that pointer. This conditions might break in situations when we have concurrent logic.

### Heap
  - Heap memory allocation is not free, and ir has its own costs. Not all memory allocation functions have the same cost and, usually, `malloc` is the cheapest one.
  - All memory blocks allocated from the Heap space must be freed either immediately when they are not needed anymore or just before ending the program.
  - Since Heap memory block have no scope, the program must be able to manage the memory in order to avoid any possible leakage.
  - Sticking to a chosen memory management strategy for each Heap memory block seems to be necessary.
  - The chosen strategy and its assumptions should be documented throughout the code wherever the block is accessed so that future programmers will know about it.
  - In certain programming languages like C++, we can use `RAII` objects to manage a resource, possibly a Heap memory block.
```c
malloc() // Specifies the number of bytes and then `malloc` will return a pointer to the beginning of the memory allocated.
memset() // useful to initialize an allocated memory
calloc() // Same as `malloc` but the allocated memory is zero initialized.
realloc() // Is used to reallocate an existing allocated memory . It expects two parameters  a pointer to a memory previously allocated and size of bytes. If a nullptr is passed the `realloc` acts sames as `malloc`.
free()
```
If you are going to write a cross-platform program, always be aligned with the C specification. The specification says malloc does not initialize te allocated memory block.
Even when you are writing you program only for Linux and not for other operating systems, be aware that future compilers may behave differently. Therefore, according to the C specification, we must always assume that the memory block allocated by the malloc is not initialized and it should be initialized manually if necessary.


The realloc function does not change the data in the old block and only expands an already allocated block to a new one. If it cannot expand the currently allocated block because of fragmentation, it will find another block that's large enough and copy the data from the old block to the new one. In this case, it will also free the old block. As you can see, reallocation is not cheap operation in some cases because it involves many steps, hence it should be used with care.

The free function deallocates an already allocated Heap memory block by passing the block's address as a pointer. Any allocated Heap block should be freed when it is not needed. Failing to do so leads to *memory leakage*.

## Composition and Aggregation

### Relation between classes
An object model is a set of related objects. The number of relation can be many, but there are a few relationship types that can exist between two objects. Generally, there are two categories of relationships found between objects (or their corresponding classes): `to-have` relationships and `to-be` relationships.

#### Composition
As the term "composition" implies, when an object contains or possesse another object- in other words, it is composed of another object - we say that there is a composition relationship between them.

As an example , a car has an engine; a car is an object that contains an engine object. Therefore, the car and engine objects have a composition relationship. There is an important condition that a composition relationship must have: *the lifetime of the contained object is bound to the lifetime of the container object*

#### Aggregation
Aggregation also involves a container that contains another object. The main difference is that in aggregation, the lifetime of the contained object is independent of the lifetime of the container object.

In aggregation, the contained object could be constructed even before the container object is constructed. This is opposite to composition, in which the contained object should have a lifetime shorter than or equal to the container object.


## Concurrency

### Concurrency Issues
Some concurrency issues only exist when no concurrency control mechanism is in place, and some are introduced by using a concurrency control technique.

Therefore, we now have two different groups of concurrency issues, these being:
- The issue that exist in a concurrent system while having no control (syncronization) mechanism in place. We call them *instrisic concurrency issues*.
- The issues that happen after the attempted resolution of an issue in the  first group. We call them *post-syncronization issues*

The reason behind calling the first group *instrinsic* is due to the fact that these issues are present intrissically in all concurrent systems. You cannot avoid them, and you have to deal with them via use of control mechanisms. In a way, they can be considered as a property of the concurrent systems rather than being
issues. Despite that, we treat them as issues because their non-deterministic nature interferes with our ability to develop the deterministic programs that we require.

The issues in the second group only happen when you use control mechanisms in the wrong way. Note that control mechanisms are not problematic at all and indeed are necessary to bring back the determinism to our programs. If they are used in the wrong fashion, however, they can lead to secondary concurrency problems. These secondary problems, or post-concurrency issues, can be considered as new bugs being introduced by a programmer, rather than being

#### intrinsic concurrency issues
Every concurrent system with more than one task can have a number of possible interleavings, which can be thought of as an intrinsic property of the system. We know that this property has a non-deterministic nature, which causes the instructions of different tasks to be executed in a chaotic order in each run, while still following the happens-before constraints.

##### Race Condition
A race condition is an issue that is caused by the intrinsic property of concurrent systems, or in other words, the interleavings. Whenever we get a race condition, the invariant constraints of the system are in danger of being missed. The consequences of failing to satisfy the invariant constraints can be observed

##### Data Race
Data races are very similar to race conditions, but to have a data race we need to have a shared state among different tasks and that shared state must be modifiable (writable) by at least one of the tasks. In other words, the shared state should not be read-only for all tasks, and there should be at least one task which may write to the shared state based on its logic.

#### Post-synchronization issues
Next, we're going to talk about three of the key issues that are expected to occur as a result of misusing the control mechanisms. You could experience one or even all of these issues together because they have different root causes:
- **New intrinsic issues**: Applying control mechanisms may result in having different race conditions or data races. Control mechanisms are to enforce a strict order between instructions, and this may cause newer intrinsic issues to occur. The fact that control mechanisms introduce new interleavings is the basis of experiencing new concurrency-related behaviors and issues. As a consequence of having new race conditions and new data races, new logical errors and crashes can occur. You'll have to go through the employed synchronization techniques and tune them based on your program's logic in order to fix these new issues.
- **Starvation**: When a task in a concurrent system doesn't have access to a shared resource for a long period of time, mainly because of employing a specific control mechanism, it is said that the task has become starved. A starved task cannot access the shared resource, and thus it cannot execute its purpose effectively. If other tasks rely on the cooperation of a starved task, they themselves may also get starved.
- **Deadlock**: When all the tasks in a concurrent system are waiting for each other and none of them is advancing, we say that a deadlock is reached. It happens mainly due to a control mechanism being applied in the wrong way, which in turn makes tasks enter an infinite loop waiting for the other task to release a shared resource or unlock a lock object, and so on. This is usually called a circular wait. While the tasks are waiting, none of them will be able to continue their execution, and as a result, the system will go into a coma-like situation. In a deadlock situation, all the tasks are stuck and waiting for each other. But there are often situations in which only a portion of tasks, only one or two of them, are stuck and the rest can continue. We call them semi-deadlock situations. We are going to see more of these semi-deadlock situations in the upcoming sections.
- **Priority inversion**: There are situations in which, after employing a synchronization technique, a task with a higher priority to use a shared resource is blocked behind a low priority task and this way, their priorities are reversed. This is another type of secondary issues that can happen because of a wrongly implemented synchronization technique.

## Thread
The most basic fact about thread is that it is limited to a single-process system. Every thread is initiated by a process. It will the belong to the process forever. Every process has at least  one thread that is its *main thread*. In C program, the main function is executed as a part of the main thread.

All the threads share the same **Process ID (PID)**. If you use utilities like `top` or `htop`, you can easily see the threads are sharing the same process ID, and are grouped under it. More than that, all the attributes of the owner process are inherited by all of its threads for example, group ID, user ID, current working directory, and signal handlers. As an example, the current working directory of a thread is the same as its owner process.

Every thread has a unique and dedicated **Thread ID (TID)**. This ID can be used to pass signals to that thread or track it while debugging. In addition, every thread has also a dedicated signal mask that can be used to filter out the signals it may receive.

Here, you can fund a list of methods that can be used by threads to share or transfer a state in a POSIX-compliant system:
- Owner process's memory (Data, Stack, and Heap segments). This method is *only* specific to threads and not process.
- Filesystem
- Memory-mapped files.
- Network (Using internet sockets).
- Signal passed between threads.
- Shared memory.
- POSIX pipes.
- Unix domain sockets.
- POSIX message queues.
- Environment variables.

The lifetime of a thread is dependent on the lifetime of its owner process. When a process gets *killed* or *terminated*, all threads belonging to that process will also get terminated.

The process that creates a thread can be the kernel process. At the same time, it can also be a user process initiated in the user space. If the process is the kernel, the thread is called a *kernel-level thread* or simply a *kernel thread*, otherwise, the thread is called a *user-level thread*. Kernel threads typically execute important logic, and because of this they have higher priorities that that of user threads.

Regarding the memory layout of thread, every thread has its own Stack memory region that can be considered as a private memory region dedicated to that thread. In practice, however, it can be accessed by other threads (within the same process) when haveing a pointer addressing it.

### Thread Stack memory
Each thread has its own Stack region that is supposed to be private to that thread only. A thread's Stack region is part of the owner process's Stack segment and all threads, by default, should have their Stack region allocated from the Stack segment. It is also possible that a thread has a Stack region that is allocated from the Heap Segment.

Since all threads within the same process can read and modify the process's Stack segment, they can effectively read and modify each other's Stack regions, but the *should not*. Note that working with other threads' Stack regions is considered dangerous behavior because the variables defined on top of the various Stack regions are subject to deallocation at any time, especially when a thread exits or a function returns.

### Thread Heap memory
The Heap segment and the Data segment are accessible by all threads. Unlike the Data segment, which is generated at compile time, the Heap segment is dynamic, and it is shaped at runtime. Threads can both read and modify the contents of the Heap. In addition, the contents of the Heap can stay as long as the process lives, and stay independent of the lifetime of the individual threads. Also, big objects can be put inside the Heap. All these factors together have caused the Heap to be a great place for storing states that are going to be shared among some threads.

### Memory visibility
As you know, a cache coherency protocol among CPU cores ensures that all cached versions of a single memory address in all CPU cores remain synchronized and updated regarding the latest changes made in one of the CPU cores. But this protocol should be triggered somehow.

There are APIs in the system call interface to trigger the cache coherency protocol and make the memory visible to all CPU cores. In pthread also, there are a number of functions that guarantee the memory visibility before their execution.

You may have encountered some of these functions before. A list of them is presented below:
- pthread_barrier_wait
- pthread_cond_broadcast
- pthread_cond_signal
- pthread_cond_timedwait
- pthread_cond_wait
- pthread_create
- pthread_join
- pthread_mutex_lock
- pthread_mutex_timedlock
- pthread_mutex_trylock
- pthread_mutex_unlock
- pthread_spin_lock
- pthread_spin_trylock
- pthread_spin_unlock
- pthread_rwlock_rdlock
- pthread_rwlock_timedrdlock
- pthread_rwlock_timedwrlock
- pthread_rwlock_tryrdlock
- pthread_rwlock_trywrlock
- pthread_rwlock_unlock
- pthread_rwlock_wrlock
- sem_post
- sem_timedwait
- sem_trywait
- sem_wait
- semctl
- semop

Other than local caches in CPU cores, the compilers can also introduce caching mechanisms for the frequently used variable. For this to happen, the compiler needs to analyze the code and optimize it in a way that means frequently used variables are written to and read from the compiler caches. These are software caches that are put in the final binary by the compiler in order to optimize and boost the execution of the program.

While these caches can be beneficial, they potentially add another headache while writing multithreaded code and raise some memory visibility issues. Therefore, sometimes these caches must be disabled for specific variables. The variables that are not supposed to be optimized by the compiler via caching can be declared as volatile. Note that a volatile variable still can be cached at the CPU level, but the compiler won't optimize it by keeping it in compiler caches. A variable can be declared as volatile using the keyword volatile. Following is a declaration of an integer that is volatile:

```c
volatile int number;
```
The important thing about volatile variables is that they don't solve the memory visibility problems in multi-threaded systems. In order to solve this issue, you need to use the preceding POSIX functions in their proper places in order to ensure memory visibility.
