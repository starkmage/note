## Bus

A bus is a common communication line for transmitting information between various functional components of a computer. It consists of a bundle of transmission lines made of [conductors](https://baike.baidu.com/item/导线/1413914). According to the types of information transmitted in computers, computer buses can be divided into [data bus](https://baike.baidu.com/item/数据总线/272650), [address bus](https://baike.baidu.com/item/地址总线/4307936), and [control bus](https://baike.baidu.com/item/控制总线/272568), used to transmit data, data addresses, and [control signals](https://baike.baidu.com/item/控制信号/10329713) respectively.

## Process States

* 5 States
  * Running State
    * Occupies CPU and is running on the CPU
  * Ready State
    * Has met running conditions but cannot run due to no available CPU
  * Blocked State
    * Cannot run temporarily due to waiting for an event or resource
  * Created State
  * Terminated State
* Only ready and running states can transition between each other; all other transitions are one-way.
* Ready state processes get CPU time through scheduling algorithms to transition to running state; running state processes transition to ready state when their CPU time slice is used up, waiting for next scheduling
* Blocked state occurs when a running process lacks needed resources, but this resource doesn't include CPU time - lacking CPU time causes transition from running to ready state
* Blocked state transitions to ready state when required resources become available

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ProcessState.png)

## Differences Between Processes and Threads

A process refers to an application program running in the operating system, while a thread refers to an independent unit executing a specific task within a process.

Process is the basic unit of resource allocation, used to manage resources; thread is the basic unit of independent scheduling, and multiple threads within a process share process resources.

1. Resource Ownership

   Process is the basic unit of resource allocation, but threads don't own resources. Threads can access resources belonging to their process

2. Scheduling

   Thread is the basic unit of independent scheduling. Within the same process, thread switching doesn't cause process switching. Switching between threads of different processes will cause process switching

3. System Overhead

   The overhead of creating or terminating processes is much larger than that of threads because the system needs to allocate or reclaim resources like memory space and I/O devices. Similarly, process switching involves saving the current process's CPU environment and setting up the new process's CPU environment, while thread switching only requires saving and setting up a small amount of register content

4. Communication

   Threads can communicate by directly reading and writing data within the same process, but process communication needs IPC

## Inter-Process Communication (IPC) Methods

* Shared Memory
  * Operating system allocates a shared space in memory
  * Processes access shared space mutually exclusively
  * Multiple processes can map the same file to their address spaces to achieve shared memory
  * Shared memory is the fastest IPC method
* Message Queue
  * Message queues can exist independently of reading and writing processes
  * Avoids synchronization blocking issues, no need for processes to provide synchronization methods
  * Reading processes can selectively receive messages based on message type
* Pipe Communication
  * Opens a fixed-size buffer in memory
  * Pipes can only use half-duplex communication; two pipes are needed for bidirectional communication
  * Processes access pipes mutually exclusively
* Semaphore
  * Signal is a more complex communication method used to notify receiving processes that certain events have occurred
* Socket Communication

## Unshared Parts in Threads of the Same Process

Stack Space

Thread functions can call other functions, and called functions can be nested layer by layer, so threads must have their own function call stack to ensure normal function execution without interference from other threads.

**Content Shared by Threads:**

1. Process code segment
2. Process data segment
3. File descriptors opened by the process
4. Signal handlers
5. Current directory of the process
6. Process user ID and process group ID

**Content Unique to Threads:**

1. Thread ID
2. Register set values
3. Thread stack
4. Error return code
5. Thread signal mask

## Conditions for Deadlock

Deadlock refers to a phenomenon where processes are mutually waiting for resources held by each other, causing all processes to block and unable to progress. Deadlock must occur in blocked state and involve at least two processes.

Necessary conditions for deadlock:

1. **Mutual Exclusion**: A resource can only be held by one process at a time and cannot be held by two or more processes simultaneously.
2. **No Preemption**: Resources cannot be forcibly taken from resource holders by other processes requesting resources before the holders complete using them; resources can only be released voluntarily.
3. **Hold and Wait**: A process has already held at least one resource but is requesting a new resource held by other processes, putting it in a waiting state.
4. **Circular Wait**: Several processes form a circular chain, each holding the next resource that others are requesting.

Deadlock handling strategies:

1. Ostrich Algorithm

   Most operating systems, including Unix, Linux, and Windows, handle deadlock problems simply by ignoring them

2. Deadlock Prevention — Prevent deadlock **before process execution**
   * Break mutual exclusion condition
   * Break no preemption condition
   * Break hold and wait condition
     * One implementation is to require all processes to request all needed resources before starting execution
   * Break circular wait
     * Give resources uniform numbering, processes can only request resources in numerical order

3. Deadlock Avoidance — Avoid deadlock **during process execution**, Banker's Algorithm

   Safe sequence: If the system allocates resources according to this sequence, each process can complete successfully. As long as a safe sequence can be found, the system is in a safe state

4. Deadlock Detection and Resolution
   - Resource preemption method
   - Process termination method
   - Process rollback method

## Banker's Algorithm

A small-town banker who has promised certain credit limits to a group of clients. The algorithm determines whether satisfying requests would enter an unsafe state; if so, the request is denied; if not, resources are allocated.

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/d160ec2e-cfe2-4640-bda7-62f53e58b8c0.png)

In figure a, the second column 'Has' shows resources already owned, the third column 'Max' shows total resources needed, and 'Free' shows available resources.

Figure c shows an unsafe state, so the algorithm would reject the previous request to avoid entering the state shown in figure c.

## Concurrency and Parallelism

Concurrency: In operating systems, concurrency refers to several programs being in states between started and completed during a time period, and all these programs running on the same processor.

Parallelism: When a system has more than one CPU, while one CPU executes one process, another CPU can execute another process. The two processes don't compete for CPU resources and can proceed simultaneously. We call this parallelism.

Concurrency is like holding chopsticks in one hand and a phone in the other, saying a sentence, then taking a bite of food.
Parallelism is like swallowing food while speaking a sentence, which is impossible with just one mouth - you need at least two mouths.

The key to concurrency is having the ability to handle multiple tasks, not necessarily simultaneously.
The key to parallelism is having the ability to handle multiple tasks simultaneously.

So I think their most crucial difference is whether it's "simultaneous".

## Multiple Threads Accessing the Same Memory

Implemented through **locking and unlocking** files. When a file is being operated by one user, that file is locked.

Multiple threads can read from the same memory simultaneously but cannot write simultaneously.

## Static vs Dynamic Languages, Strong vs Weak Typing

Static typing: All variable types are determined at compile time

```c++
class C {
    public;
        int x;
        int y;
}
```

Dynamic typing: All variable types are determined at execution time

```js
class C {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
}

var c1 = new C(10, 20);
var c2 = new C('hello', 'world');
```

Strong typing: Does not allow changing variable data types unless explicit type conversion is performed

Weak typing: Variables can be assigned values of different types

## Differences Between 64-bit and 32-bit Operating Systems

1. Different processing capabilities. 64-bit can process 8 bytes of data at once, while 32-bit can only process 4 bytes at once, thus 64-bit improves processing capability by one fold.

2. Different memory addressing. 64-bit maximum addressing space is 2^64, reaching a theoretical value of 16TB, while 32-bit maximum addressing space is 2^32, which is 4GB. In other words, 32-bit system processors support maximum 4GB memory, while 64-bit systems support memory up to billions.

3. Different software compatibility. Due to different instruction sets between 32-bit and 64-bit CPUs, software needs to be distinguished between 32-bit and 64-bit versions.

## Why Memory is Divided into Stack and Heap

For convenient memory/data management

Stack memory has a very clear lifecycle, and stack memory management is quite straightforward, usually handled directly by the operating system.

Advantages of stack memory:

1. No need for dynamic addressing through pointers or references, fast access
2. Very clear lifecycle, timely release, high memory space efficiency
3. Fixed size and offset, simple memory allocation algorithm, short memory allocation delay
4. No need for complex garbage collection mechanism, stable algorithm time overhead

But stack memory also has its "defects" (strictly speaking, it wasn't designed for such use):

1. Once a function call returns, data on its corresponding stack frame can't be used anymore. In other words, data on stack memory can't survive beyond a function call's lifecycle.
2. Stack memory cannot be shared between multiple threads
3. Stack memory size is usually quite small (a few MB). If too many stack frames are pushed (=function call depth too deep) or structures in stack frames are too large, it will cause stack overflow.

Heap memory is a dynamic memory management mechanism allocated as needed (here "dynamic" means you get it when you request it, and you get as much as you request, unlike stack memory where all memory possibly needed for this call is given directly when a function is called)

Only programmers know the lifecycle of data in heap memory (if they really know), so no operating system or language runtime can accurately know when to free a piece of heap memory space. There are currently three common approaches to heap memory management:

1. Manual management by programmers (C/C++)
2. Introduce garbage collection mechanism, clean up during program idle time or when memory is insufficient (most common in languages)

https://www.zhihu.com/question/447017261

## Why Reference Values are Stored in Heap while Primitive Values in Stack

Energy is conserved, it's just a matter of trading time for space or space for time

Heap is larger than stack, stack operates faster than heap. Objects are complex structures that can be freely expanded, e.g., arrays can be infinitely expanded, objects can freely add properties. Putting them in heap is to avoid affecting stack efficiency. Instead, operations are performed by finding actual objects in heap through references.
Compared to simple data types, simple data types are more stable and only occupy small memory.

Simple data types are not put in heap because the combined cost of finding actual objects in heap through references takes time, and this total cost far exceeds the cost of getting actual values directly from stack.
Therefore, values of simple data types are stored directly in stack.