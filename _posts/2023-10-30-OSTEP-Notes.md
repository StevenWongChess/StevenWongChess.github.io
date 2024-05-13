---
layout: post
title:  "OSTEP Notes"
date:   2023-10-30
categories: OS
---



### I Intro

##### 0-Preface

eBook: https://pages.cs.wisc.edu/~remzi/OSTEP/

Code: https://github.com/remzi-arpacidusseau/ostep-code

Project: https://github.com/remzi-arpacidusseau/ostep-projects

Homework: https://github.com/remzi-arpacidusseau/ostep-homework



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



### II Virtualization

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

Assumption: fully-operational scheduling discipline

- Each job(process) same time duration
- all jobs arrive at same time
- Once started, each job runs to completion
- All jobs use CPU only
- runtime of each job known



###### 7.2 scheduling metrics

Turnaround for a job = completion - arrival (under our assumption, arrival = 0)

fairness



###### 7.3 FIFO

FIFO is not good when jobs have different duration

- Better way: shortest job first to solve the convoy effect



###### 7.4 SJF

SJF = shortest job first 

- optimal 
- Non-preempt (all modern OS scheduler are preempt)
- BUT, if we loose assumption 2 it is bad again



###### 7.5 STCF

STCF = Shortest Time-to-Completion First

- improve SJF
- preemptive



###### 7.6 new metric: response time

Response time = first run - arrive time

- STCF not good anymore



###### 7.7 RR

RR = round robin

- also called time-slicing
- the time slice length must be multiple of timer-interrupt period
- Tradeoff between
  - Response time
  - & too much context switching + worse turnaround 



In general: fair policy tend to have bad turnaround



###### 7.8 with I/O

Now relax assumption 4

- good example in figure 7.9



###### 7.9 unknow runtime

Now relax assumption 5

- solve this by learn from history & predict future 
- multi-level feedback queue, see in next chapter



###### homework

simulation

- [scheduler.py](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched)



##### 8-MLFQ

MLFQ = Multi-level feedback queue

- optimize turnaround 
- responsive to interavtive users (that is optimize response time)



###### 8.1 basic rules

There are distinct queues, each with priority

- Rule 1, if A > B, run A
- Rule 2, if A == B, run both using RR
- Caution, we need to be able to change priority

e.g. cpu intensive -> low priority, keep waiting for keyboard input -> high priority



###### 8.2 attempt1, how to change priority

Def: allotment = amount of time a job can spend at a given priority before redueces its priority

- Rule 3, when job enters, it has highest priority
- Rule 4a, if allotment is used up, `priority -= 1`
- Rule 4b, if a job gives up cpu(e.g. to do I/O), the allotment is reset

Flaw

- Starvation: what if too many interactive jobs, long-running jobs will never run
- a smart user can write the program to game the scheduler
- what if a long-running job becomes interactive later, it still can't run



###### 8.3 attempt 2, priority boost

Idea is: periodically boost the priority for all jobs

- Rule 5, after time period S, move all jobs to the topmost queue
- this solves the 1st and 3rd flaw



###### 8.4 attempt 3, better accounting

change rule 4 into 

- Rule 4, allotment at a certain level does not recover



###### 8.5 tuning MLFQ & other issues

Advanced and difficult



###### homework

simulation



##### 9-Lottery Scheduling

lottery scheduling is one of proportional-share scheduler



###### 9.1 ticket means stock share

advantage for randomness

- avoid strange corner case
- lightweight
- Fast



###### 9.2 ticket machanisms

concept

- ticket currency: user can allocate ticket to different jobs (total ticket for a user is given)

- ticket transfer
- ticket inflation: print money



###### 9.3 implementation

pick random number and traverse the list

- to increase efficiency, sort it (this will not affect fairness)



###### 9.4 example

easy, skipped



###### 9.5 how to assign tickets

open question



###### 9.6 stride scheduling

a deterministic fair-share scheduler

- description long, just see example in figure 9.3
- downside, need global state, what if a job come in midway
- Goodside is it is deterministic



###### 9.7 CFS 

###### CFS is hard and this book does not go into detail, need review

need to improve scheduler

- now it still use 5% of CPU time



CFS = Linux completely fair scheduler

- use virtual runtime
- Switch(fairness) vs performance, introduce parameters
  - `screed_latency`, this will be divided by # of jobs
  - `min_granularit`, this division can't be smaller than this limit
  - `niceness`, this is the priority (range -20 to 19, default 0), higher niceness, less priority

- Use red-black tree instead of list



###### homework

Simulation [lottery.py](https://github.com/remzi-arpacidusseau/ostep-homework/tree/master/cpu-sched-lottery)



##### 10-Multi-CPU schedule

advanced topic, go back later after reading the second piece



##### 11-Summary

funny dialogue

best way to learn is read and code



##### 12-Dialogue

Every address generated by a user program is a virtual address

- ease of use
- isolation & protection



##### 13-Address Space

###### 13.1 early systems

easy structure, see fugure 13.1



###### 13.2 muitiprogramming & time sharing

(history)



###### 13.3 address space

Code + heap + stack (top to bottom, see Figure 13.3)

Even C pointer address is virtual

- even every address programmer can see is virtuals



###### homework

play with `free` and `pmap` command



##### 14-Memory API

###### 14.1 type of memory

###### 14.2 `malloc()`

`sizeof()`

- Compile-time operator (not run time)



###### 14.3 `free()`

unlike malloc, size is not passed, must be tracked by the memory-allocation library itself



###### 14.4 common errors

list of errors

use `purufy` or `valgrind` to check	



###### 14.5 underlying OS support

`malloc` and `free` are library calls

- they call system call like `brk` and `sbrk`
- we should not use system calls for memory
- another way to obtain memory from OS: `mmap()`



###### homework

write bugs and use tools(gdb + valgrind) to debug



##### 15-Address Translation

###### 15.1 assumptions

- Address space are contiguously in physical memory
- size of address space less than size of physical memory
- Later we will assume equal size



###### 15.2 example

see figure15.2



###### 15.3 dynamic relocation

dynamic here = hardware-based = base and bounds

- physical address = base + virtual
- bounds for protection



###### 15.4 hardware summary

see figure 15.3



###### 15.5 OS issues

see figure 15.4, 15.5, 15.6



###### homework

`relocation.py`, see how address translation work



##### 16-Segmentation

First attempt to avoid internal fragmentation is segmentation, which is a slight generalization of base and bounds

###### 16.1 segmentation

better than b&b

can have code, stack, heap segments separated, each with base and size pair

enscence is we don't need to have those three together



###### 16.2 which segment

in virtual address, use first 2 prefix, e.g.

- 00 means code
- 01 means stack
- 10 means heap

there are other ways to avoid using prefix (because it waste 2 bits)



###### 16.3 stack

hardware need to know which direction segment grow, heap differ from stack

- Add a bit



###### 16.4 support for Sharing

need to know read/write/execute protection

- need protection bits



###### 16.5 Fine-grained vs. Coarse-grained Segmentation

just big/small segments



###### 16.6 OS supports

3 supports

- Context swtich, need to deal with registers
- when segments grow/shrink
- how to manage free space
  - one solution: compaction, but expensive & make segments harder to grow
  - many algorithms in this field



###### homework

Simulation



##### 17-Free-space Management

about free-space management

- fixed size units => easy
- variable-sized units => hard, causing external fragmentation



###### 17.1 assumptions

1. like heap, handle `malloc(size)` and `free(ptr)` 

- note that no size passed to free
- need free "list" to track

2. biggest concern is external fragmentation

- (for now) don't care internal fragmentation

3. memory can't be relocated

- thus no compaction allowed

4. Malloc get contiguous bytes



###### 17.2 low-level mechanisms

Splitting: if one node in free list is `x + y` and malloc ask for `x` space, we will split that into two and give `x` to malloc, `y` stay in free list

Coalescing: when `free`, look around and see if we can combine them together



###### 17.3 deal with growing size

see examples

- best fit
- worst fit
- first fit
- next fid



###### 17.4 other approaches

- segregated lists

- buddy allocation

- other ideas



###### homework

simulation



##### 18-Intro to Paging

Paging = chop space into fix-sized pieces (called page frames)



###### 18.1 simple example

- now we have page table
- use virtual page number(VPN) + offset to translate



###### 18.2 where is page tables stored

page tables are large

- store in OS memory, even OS virtual memory or disk



###### 18.3 what is in page table

linear page table for now, later chap will discuss other

a data structure has many bits 

- reference bit
- dirty bit
- present bit
- protection bits
- valid bit



###### 18.4 also too slow

a simple one-line operation can be slow

- see example



###### 18.5 memory trace

array example, next two chap will cover improvement for paging



##### 19-Translation Lookasside Buffers

we need to speed up 























