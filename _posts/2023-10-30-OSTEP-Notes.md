---
layout: post
title:  "OSTEP Notes"
date:   2023-10-30
categories: OS
---



#### Intro

##### 0-Preface

eBook: https://pages.cs.wisc.edu/~remzi/OSTEP/

Code: https://github.com/remzi-arpacidusseau/ostep-code

Project: https://github.com/remzi-arpacidusseau/ostep-projects



##### 1-Dialogue

Why 3 pieces?

- physics book 6 pieces, and OS is half difficult
- VIrtualization
- Concurrency
- Persistence



##### 2-Intro to OS

###### 2.1 virtualize CPU

`cpu.c` -> [here](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/intro/cpu.c)

`Spin()` : a function that repeatedly checks the time and returns once it has run for a second defined in `common.h`

```bash
gcc -o cpu cpu.c -Wall
./cpu A & ./cpu B & ./cpu C & ./cpu D &
# output
[1] 7353
[2] 7354
[3] 7355
[4] 7356 A
B
D
C
A
B
D
C
A
...
```

this is running many programs at once



###### 2.2 virtualize memory

`mem.c` -> [here](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/intro/mem.c)

- It is as if each running program has its own private memory, instead of sharing the same physical memory with other running programs
- an issue -> see readme

```bash
./mem & ./mem &
# output
[1] 24113
[2] 24114
(24113) address pointed to by p: 0x200000
(24114) address pointed to by p: 0x200000
(24113) p: 1
(24114) p: 1
(24114) p: 2
(24113) p: 2
(24113) p: 3
(24114) p: 3
(24113) p: 4
(24114) p: 4
```



###### 2.3 concurrency

`threads.c` -> [here](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/intro/threads.c)

```bash
gcc -o threads threads.c -Wall -pthread
./threads 1000
# output
Initial value : 0
Final value   : 2000

./threads 100000
# output
Initial value : 0
Final value   : 143012
```

the expected 2N is not there, why? 

- understand this program later
- `++` actually take 3 instructions that do not execute atomically



###### 2.4 persistence

`io.c` -> [here](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/intro/io.c)

4 useful IO APIs

- `open("/tmp/file", O_WRONLY | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);`
- `write(fd, buffer, strlen(buffer));`
- `fsync(fd);`
- `close(fd);`



###### 2.5 design goals

- trade-off between

  - minimize overheads (time/space)

  - virtualization/abstraction

- protection, key is isolation

  - between applications
  - between application and OS

- Reliability, run non-stop

- other goals



###### 2.6 history

refer back to [chapter](https://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf)

- library of commonly-used functions & human operator
- invented system call & kernel/user mode
- multiprogramming because IO is slow and we can switch to another job
- Unix->Linux->...



#### Virtualization

##### 3-Dialogue

Great Metaphor: 

- only one peach 
- try to make several people think that they are eating 1 peach
- without they knowing that there is only 1 peach



##### 4-Process

Many OS seperate these 2

- Low-level mechanism
  - e.g. context switch
- High-level policy
  - e.g. schuduling policy

###### 4.1 process

process is an instance of program

- memory (address space)
- registers
  - program counter(PC)
  - stack pointer
  - frame pointer
- IO



###### 4.2 process api

we need these kinds of APIs

- create
- destroy
- wait
- miscellaneous control
- get status info



###### 4.3 create

- Load code & static data 

  - from disk to memory

  - (modern OS do it lazily)

  - after learn machinery of paging & swapping

- stack

- heap

- IO

  - file descriptors

- go to `main`



###### 4.4 process states

3 states: running, ready, blocked 

- -> see relationship figure 4.2
- -> see example figure 4.3, 4.4



###### 4.5 data structures

Figure 4.5 -> 

- how OS store process info, using `process list`
- each node called process control block (PCB)

- many states
- many variables



###### homework

see [readme](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-intro) for a simulation of process schedule



##### 5-Process API

###### 5.1 `fork()`

`pid()` -> [p1.c](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/cpu-api/p1.c)

- PID = process identifier
- the order of 2 process is non-deterministic



###### 5.2 `wait()/waitpid()`

`wait()` won't return until its child has run and exited

- to make the order deterministic
- [p2.c](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/cpu-api/p2.c)



###### 5.3 `exec()`

usually used after `fork()` to make the copied process do something different

- [p3.c](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/cpu-api/p3.c)

```c
char *myargs[3];
myargs[0] = strdup("wc");   // program: "wc" (word count)
myargs[1] = strdup("p3.c"); // argument: file to count
myargs[2] = NULL;           // marks end of array
execvp(myargs[0], myargs);  // runs word count
```



###### 5.4 why api

making `fork()` and `exec()` seperated

- we can do something between `fork()` and `exec()`

- to change the env for the about-to-run program
- allow shell to do useful things, e.g. redirection, pipe, see [p4.c](https://github.com/remzi-arpacidusseau/ostep-code/blob/master/cpu-api/p4.c)



###### 5.5 process control & user

signal

- `kill()`
- Control-c sends `SIGINT`, normally terminate process
- Control-z sends `SIGTSTP`, pause process, can resume later using `fg` in shell
- different user have different permissions



###### 5.6 useful tools

```bash
ps 
top
kill
```



###### homework

Simulation & coding

- [Link](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-api)
- good place to practice api calls



##### 6-Mechanism: limited direct execution

###### 6.1 basic tech: lde

direct execution needs limitation

- make sure process running good
- need to stop/switch between process



###### 6.2 prob 1: restricted ops

system call

- User mode -> trap -> kernal mode -> return-from-trap
- see workflow in figure 6.2
- Use trap table at boot tume to set where to jump to for process trap
- each has a system-call number



###### 6.3 prob 2: switch process

reboot is good 233

use the timer interrupt handler to let OS take control from process



###### 6.4 worried about concurrency?

tricky situations

- what if during system call, a timer interrupt happens
- what if handling one interrupt, another happens
- will learn later -> see concurrency
- Solution: disable other interrupt when handling one



###### homework

some measure work, skipped



##### 7-Scheduling

###### 7.1 workload assumptions















