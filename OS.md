# Operating System Concepts - Comprehensive Guide

A detailed reference guide covering fundamental operating system concepts, process management, memory management, concurrency, and system programming.

---

## Table of Contents

- [Processes and Threads](#processes-and-threads)
  - [What is a Process?](#what-is-a-process)
  - [What is a Thread?](#what-is-a-thread)
  - [Process vs Thread](#process-vs-thread)
  - [Lightweight Processes](#lightweight-processes)
- [Context Switching](#context-switching)
  - [Process Context Switching](#process-context-switching)
  - [Thread Context Switching](#thread-context-switching)
  - [Single vs Multiple CPU Cores](#single-vs-multiple-cpu-cores)
- [Multi-processing and Multi-threading](#multi-processing-and-multi-threading)
- [Process States and Scheduling](#process-states-and-scheduling)
  - [Process States](#process-states)
  - [State Transitions](#state-transitions)
  - [Scheduling Algorithms](#scheduling-algorithms)
  - [Thread Sleep and Process Wait](#thread-sleep-and-process-wait)
- [Thread Pools](#thread-pools)
- [Inter-Process Communication](#inter-process-communication)
  - [Process Address Spaces](#process-address-spaces)
  - [Shared Libraries](#shared-libraries)
  - [IPC Mechanisms](#ipc-mechanisms)
  - [Parent-Child Processes](#parent-child-processes)
  - [Copy-on-Write (COW)](#copy-on-write-cow)
  - [File Descriptors and Sockets](#file-descriptors-and-sockets)
- [Concurrency and Parallelism](#concurrency-and-parallelism)
- [Synchronization Mechanisms](#synchronization-mechanisms)
  - [Critical Zones](#critical-zones)
  - [Race Conditions](#race-conditions)
  - [Locking Mechanisms](#locking-mechanisms)
  - [Deadlocks](#deadlocks)
- [Memory Management](#memory-management)
  - [Virtual Memory](#virtual-memory)
  - [Paging](#paging)
  - [Memory Mapping](#memory-mapping)
  - [Heap vs Stack](#heap-vs-stack)
  - [Memory Allocation](#memory-allocation)
  - [Garbage Collection](#garbage-collection)
- [Pointers and Memory](#pointers-and-memory)
- [System Architecture](#system-architecture)
  - [Linux File Philosophy](#linux-file-philosophy)
  - [File Descriptors](#file-descriptors)
  - [Buffers](#buffers)
  - [System Calls](#system-calls)
  - [User Space vs Kernel Space](#user-space-vs-kernel-space)
- [Caching](#caching)
  - [In-Memory Caching](#in-memory-caching)
  - [LRU Cache](#lru-cache)
  - [Cache Stampede](#cache-stampede)
- [Algorithms](#algorithms)

---

## Processes and Threads

### What is a Process?

A **process** is an instance of a program in execution. When you run an application, the operating system creates a process for it. A process is more than just the program code (often called the "text section"):

**Components of a Process:**
- **Text Section**: The executable code of the program
- **Data Section**: Global and static variables
- **Heap**: Dynamically allocated memory during runtime
- **Stack**: Temporary data (function parameters, return addresses, local variables)
- **Program Counter**: Current position in the program
- **CPU Registers**: Values stored in CPU registers
- **List of Open Files**: File descriptors the process is using
- **Process ID (PID)**: Unique identifier assigned by the OS

**Key Characteristics:**
- Each process has its own isolated memory space
- Processes cannot directly access each other's memory
- Creating a process is resource-intensive
- Processes provide strong isolation and security
- If one process crashes, it typically doesn't affect other processes

### What is a Thread?

A **thread** (sometimes called a "lightweight process") is the smallest sequence of programmed instructions that can be managed independently by the operating system's scheduler. A thread exists within a process and shares the process's resources.

**Components of a Thread:**
- **Thread ID**: Unique identifier within the process
- **Program Counter**: Points to the next instruction to execute
- **Register Set**: CPU register values specific to this thread
- **Stack**: Local variables and function call history for this thread
- **Thread State**: Running, ready, blocked, etc.

**Key Characteristics:**
- Multiple threads can exist within a single process
- Threads share the process's memory space (heap, data section, code)
- Each thread has its own stack and registers
- Communication between threads is fast and easy (shared memory)
- Creating threads is less resource-intensive than creating processes
- Threads are more susceptible to bugs (race conditions, deadlocks)

### Process vs Thread

| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory** | Separate memory space | Shares memory with other threads in the same process |
| **Communication** | Complex (IPC mechanisms required) | Simple (shared variables) |
| **Creation Time** | Slow (more overhead) | Fast (less overhead) |
| **Context Switching** | Expensive (must switch address spaces) | Cheaper (same address space) |
| **Resource Consumption** | High (separate resources) | Low (shares resources) |
| **Isolation** | Strong (crash in one doesn't affect others) | Weak (crash in one thread can crash the process) |
| **Security** | More secure (isolated) | Less secure (shared memory) |

### Lightweight Processes

Threads are often called "lightweight processes" because:

1. **Lower Creation Overhead**: Creating a thread doesn't require allocating a new address space, copying the parent's memory map, or setting up new page tables. The OS simply allocates a new stack and thread control block.

2. **Faster Context Switching**: When switching between threads of the same process:
   - No need to switch address spaces (no TLB flush required)
   - Only register state and stack pointer need to be saved/restored
   - Memory management unit (MMU) doesn't need reconfiguration
   - Cache remains valid (no cache invalidation)

3. **Efficient Resource Sharing**: Threads share:
   - Code section (text)
   - Data section (global variables)
   - Heap memory
   - File descriptors
   - Signal handlers
   - Process ID and working directory

**Practical Impact:**
- Thread context switch: ~1-2 microseconds
- Process context switch: ~10-100 microseconds
- This 10-100x difference makes threads ideal for handling many concurrent tasks

---

## Context Switching

**Context switching** is the mechanism by which the CPU switches from executing one process/thread to another. This is fundamental to multitasking operating systems.

### Process Context Switching

**What Happens During a Process Context Switch:**

1. **Save Current Process State**:
   ```
   - Save all CPU register values
   - Save program counter (next instruction address)
   - Save stack pointer
   - Save process state (running, ready, waiting)
   - Save memory management information (page tables)
   - Save I/O status information
   - All this is stored in the Process Control Block (PCB)
   ```

2. **Update Process State**:
   - Mark current process as "Ready" (if preempted) or "Waiting" (if blocked)
   - Update process statistics and accounting information

3. **Select Next Process**:
   - OS scheduler selects the next process to run
   - Decision based on scheduling algorithm (Round Robin, Priority, etc.)

4. **Restore New Process State**:
   - Load the selected process's PCB
   - Restore all CPU register values
   - Restore program counter
   - Switch to the process's address space (update page tables in MMU)
   - Flush TLB (Translation Lookaside Buffer)
   - Potentially invalidate CPU caches

5. **Resume Execution**:
   - CPU begins executing the new process from where it left off

**Performance Implications:**
- The address space switch is expensive (involves hardware MMU reconfiguration)
- TLB flush means initial memory accesses will be slower (TLB misses)
- Cache pollution: new process may evict useful data from CPU caches

### Thread Context Switching

**What Happens During a Thread Context Switch (Same Process):**

1. **Save Current Thread State**:
   ```
   - Save CPU register values
   - Save program counter
   - Save stack pointer
   - Save thread state
   - All this is stored in the Thread Control Block (TCB)
   ```

2. **Select Next Thread**:
   - Scheduler picks the next thread to run
   - Could be from the same process or different process

3. **Restore New Thread State**:
   - Load the selected thread's TCB
   - Restore CPU registers
   - Restore program counter and stack pointer
   - **If same process**: No address space switch needed!
   - **If different process**: Must switch address space (like process switch)

**Key Difference:**
- **Same process threads**: Address space remains the same
- No page table switch, no TLB flush, no cache invalidation
- Much faster than process context switching

### Single vs Multiple CPU Cores

#### Single CPU Core:

**Concurrency Through Time-Slicing:**
```
Time: --->
Core 1: [Process A][Process B][Process A][Process C][Process B]
        <-------- Context Switches ---------->
```

- Only one process/thread runs at any instant
- OS rapidly switches between processes (time-slicing)
- Creates the illusion of parallelism
- Scheduling algorithm determines which process runs next
- Context switches happen frequently (every 10-100ms typically)

**Data Safety:**
- Only one process executing at any time
- Race conditions can still occur (interrupts, signals)
- Need synchronization primitives even on single core
- Preemptive scheduling can interrupt a process mid-operation

#### Multiple CPU Cores:

**True Parallelism:**
```
Time: --->
Core 1: [Process A ------ continues ------]
Core 2: [Process B ------ continues ------]
Core 3: [Process C ------ continues ------]
Core 4: [Process D ------ continues ------]
```

- Multiple processes/threads run simultaneously
- Each core can execute a different process/thread
- True parallel execution of code

**Data Safety Challenges:**
- Multiple cores can access shared memory simultaneously
- Cache coherence protocols (MESI, MOESI) ensure consistency
- Hardware memory barriers and atomic operations
- More complex synchronization required:
  - Atomic operations (CAS - Compare-And-Swap)
  - Memory fences/barriers
  - Lock-free data structures
  - Thread-safe primitives

**Example Data Safety Issue:**
```c
// Thread 1 (Core 1)          // Thread 2 (Core 2)
int balance = 100;             int balance = 100;
balance = balance + 50;        balance = balance - 30;
// Result could be 150, 70, or other values depending on timing
```

**Solutions:**
```c
// Using mutex
pthread_mutex_lock(&lock);
balance = balance + 50;
pthread_mutex_unlock(&lock);

// Using atomic operations
atomic_fetch_add(&balance, 50);
```

---

## Multi-processing and Multi-threading

### Multi-processing

**Multi-processing** refers to the ability of a system to support more than one process running at the same time.

**Types:**
1. **Symmetric Multi-Processing (SMP)**:
   - Multiple processors share the same memory
   - Each processor can execute any process
   - Modern multi-core CPUs use this approach

2. **Asymmetric Multi-Processing**:
   - Each processor assigned specific tasks
   - One "master" processor controls the system
   - Other processors do designated work

**When to Use Multi-processing:**
- Need strong isolation between tasks
- Tasks are CPU-intensive and independent
- Security is critical (separate address spaces)
- Want to leverage multiple CPU cores for different applications
- Example: Web browser (separate processes for each tab)

**Advantages:**
- Strong isolation (crash in one process doesn't affect others)
- Can distribute across multiple physical machines
- Better security boundary
- Can use more than 4GB memory per task (in 32-bit systems)

**Disadvantages:**
- Higher overhead (memory, creation time)
- Slower inter-process communication
- More complex data sharing

### Multi-threading

**Multi-threading** is when a single process contains multiple threads running concurrently.

**Threading Models:**
1. **User-Level Threads**: Managed by user-space library
2. **Kernel-Level Threads**: Managed by OS kernel
3. **Hybrid**: Combination of both (like M:N model)

**When to Use Multi-threading:**
- Tasks share a lot of data
- Need fast communication between tasks
- I/O-bound operations (network requests, file I/O)
- Want to maintain responsive UI while processing
- Example: Web server handling concurrent requests

**Advantages:**
- Fast creation and context switching
- Easy data sharing (shared memory)
- Lower memory overhead
- Better cache utilization

**Disadvantages:**
- No isolation (bug in one thread affects all)
- Synchronization complexity (race conditions, deadlocks)
- Global state can be modified by any thread
- Debugging is harder

**Practical Examples:**

```python
# Multi-processing example
from multiprocessing import Process

def worker(task_id):
    print(f"Process {task_id} working")
    # CPU-intensive work
    result = compute_heavy_task(task_id)
    return result

processes = []
for i in range(4):
    p = Process(target=worker, args=(i,))
    processes.append(p)
    p.start()

for p in processes:
    p.join()
```

```python
# Multi-threading example
from threading import Thread

def handle_request(request):
    print(f"Thread handling {request}")
    # I/O-bound work
    data = fetch_from_database(request)
    process_data(data)

threads = []
for request in requests:
    t = Thread(target=handle_request, args=(request,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

---

## Process States and Scheduling

### Process States

A process goes through different states during its lifetime. The typical process state model includes:

#### 1. **New**
- Process is being created
- OS is allocating resources
- PCB is being initialized
- Not yet ready to execute

#### 2. **Ready**
- Process has all resources it needs to run
- Waiting for CPU assignment
- In the ready queue
- Can be selected by scheduler at any time

#### 3. **Running**
- Process is currently being executed by CPU
- Instructions are being processed
- Only one process per CPU core can be in this state

#### 4. **Waiting (Blocked)**
- Process is waiting for an event to occur
- Cannot proceed until event happens
- Examples:
  - Waiting for I/O completion
  - Waiting for user input
  - Waiting for a resource to become available
  - Waiting for a signal

#### 5. **Terminated**
- Process has finished execution
- Resources are being deallocated
- PCB is being cleaned up
- Exit status is available to parent process

### State Transitions

```
    NEW
     |
     v
   READY <----------+
     |              |
     | (scheduled)  |
     v              |
  RUNNING           |
     |              |
     +---(I/O wait)--> WAITING
     |                    |
     | (time slice        | (I/O complete)
     |  expired)          |
     +--------------------+
     |
     | (process completes)
     v
 TERMINATED
```

**Transition Details:**

1. **New → Ready**: 
   - Process creation complete
   - Admitted to ready queue

2. **Ready → Running**:
   - Scheduler dispatches process
   - CPU assigned to process
   - Called "dispatching"

3. **Running → Ready**:
   - Time quantum expired (in time-sharing)
   - Higher priority process arrives
   - Called "preemption"

4. **Running → Waiting**:
   - Process requests I/O
   - Process waits for resource
   - Process sleeps voluntarily
   - Called "blocking"

5. **Waiting → Ready**:
   - I/O completes
   - Resource becomes available
   - Signal received
   - Called "wake up"

6. **Running → Terminated**:
   - Process completes normally
   - Process encounters error
   - Process is killed

### Scheduling Algorithms

The OS scheduler determines which process gets CPU time. Different algorithms optimize for different goals.

#### 1. **First-Come, First-Served (FCFS)**

- Simplest scheduling algorithm
- Processes executed in order of arrival
- Non-preemptive

**Characteristics:**
- Easy to implement
- Fair in terms of order
- Poor average waiting time
- Convoy effect: short processes wait for long ones

**Example:**
```
Process  Arrival  Burst Time
P1       0        24
P2       1        3
P3       2        3

Execution: [P1 (24)][P2 (3)][P3 (3)]
Waiting: P1=0, P2=23, P3=25
Average: 16
```

#### 2. **Shortest Job First (SJF)**

- Select process with smallest burst time
- Can be preemptive or non-preemptive
- Optimal average waiting time

**Characteristics:**
- Minimizes average waiting time
- Requires knowing burst time in advance
- Can cause starvation of long processes

#### 3. **Shortest Remaining Time First (SRTF)**

- Preemptive version of SJF
- If new process arrives with shorter burst time, preempt current

#### 4. **Round Robin (RR)**

- Each process gets small time quantum (time slice)
- After quantum expires, process goes to end of ready queue
- Preemptive

**Characteristics:**
- Fair allocation of CPU
- Good for time-sharing systems
- Response time depends on quantum size
- Quantum too small → too many context switches
- Quantum too large → degenerates to FCFS

**Example with quantum = 4:**
```
Process  Burst Time
P1       24
P2       3
P3       3

Execution: [P1(4)][P2(3)][P3(3)][P1(4)][P1(4)]...
```

#### 5. **Priority Scheduling**

- Each process assigned a priority number
- CPU allocated to highest priority process
- Can be preemptive or non-preemptive

**Characteristics:**
- Can cause starvation (low priority never executes)
- Solution: Aging (gradually increase priority of waiting processes)

#### 6. **Multilevel Queue Scheduling**

- Ready queue divided into separate queues
- Each queue has its own scheduling algorithm
- Processes permanently assigned to queues

**Example:**
- System processes (highest priority, FCFS)
- Interactive processes (high priority, RR)
- Batch processes (low priority, FCFS)

#### 7. **Multilevel Feedback Queue**

- Like multilevel queue, but processes can move between queues
- Adapt to process behavior
- Separate I/O-bound and CPU-bound processes

### Thread Sleep and Process Wait

#### Process Waiting

**What Happens:**
1. Process requests I/O or resource
2. Process state changes from Running → Waiting
3. Process removed from ready queue
4. OS selects another process to run
5. When event occurs, process moves to Ready state

**Example Scenarios:**
- Waiting for disk I/O to complete
- Waiting for network packet
- Waiting for user input
- Waiting for child process to terminate

**System Behavior:**
```c
// Process makes blocking system call
read(fd, buffer, size);  // Process enters Waiting state
// OS runs other processes
// When data available, process moves to Ready
// Eventually scheduled and continues here
```

#### Thread Sleeping

**What Happens:**
1. Thread calls sleep function
2. Thread marked as sleeping for specific duration
3. Thread removed from runnable threads
4. Other threads in same/different processes can run
5. After duration, thread moves to ready state

**Key Differences:**
- Sleep is time-based, wait is event-based
- Sleeping thread doesn't consume CPU
- Other threads in same process continue executing

**How CPU Detects Sleep/Wake:**

The OS maintains timing information for each thread:
```c
// Thread structure (simplified)
struct thread {
    int state;              // RUNNING, READY, SLEEPING, WAITING
    uint64_t wake_time;     // Time to wake up (if sleeping)
    event_t* wait_event;    // Event to wait for (if waiting)
    // ... other fields
};
```

**Scheduler Logic:**
```
For each scheduling cycle:
    1. Check current time
    2. For each sleeping thread:
        if (current_time >= thread.wake_time):
            thread.state = READY
            add_to_ready_queue(thread)
    3. For each waiting thread:
        if (thread.wait_event.occurred):
            thread.state = READY
            add_to_ready_queue(thread)
    4. Select next thread from ready queue
    5. Context switch to selected thread
```

**Example:**
```python
import threading
import time

def worker():
    print("Thread starting")
    time.sleep(5)  # Thread gives up CPU for 5 seconds
    print("Thread resuming")  # OS wakes thread after 5 seconds

t = threading.Thread(target=worker)
t.start()
# Other threads can run during the 5-second sleep
t.join()
```

---

## Thread Pools

A **thread pool** is a software design pattern for achieving concurrency of execution in a computer program.

### Concept

Instead of creating a new thread for each task:
1. Create a fixed number of threads at startup (the "pool")
2. Threads wait for tasks in a queue
3. When task arrives, idle thread picks it up
4. After completing task, thread returns to pool
5. Reuse threads for multiple tasks

### Architecture

```
                    Task Queue
                  [Task 1][Task 2][Task 3][Task 4]
                         |
                         v
           +-------------+-------------+
           |             |             |
      [Thread 1]    [Thread 2]    [Thread 3]
           |             |             |
        [Task]        [Idle]        [Task]
```

### Advantages

1. **Performance**:
   - Thread creation/destruction is expensive
   - Reusing threads eliminates this overhead
   - Faster response time for tasks

2. **Resource Management**:
   - Limits number of concurrent threads
   - Prevents resource exhaustion
   - Predictable memory usage

3. **Load Balancing**:
   - Tasks distributed across available threads
   - No thread sits idle if work available

4. **Simplicity**:
   - Hide complexity of thread management
   - Easy to submit tasks and get results

### Implementation Examples

#### Python (using concurrent.futures)

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    print(f"Processing task {n}")
    time.sleep(1)
    return n * n

# Create thread pool with 4 threads
with ThreadPoolExecutor(max_workers=4) as executor:
    # Submit tasks
    futures = [executor.submit(task, i) for i in range(10)]
    
    # Get results as they complete
    for future in futures:
        result = future.result()
        print(f"Result: {result}")
```

#### Java

```java
import java.util.concurrent.*;

public class ThreadPoolExample {
    public static void main(String[] args) {
        // Create thread pool with 4 threads
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        // Submit tasks
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " running");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return taskId * taskId;
            });
        }
        
        // Shutdown pool
        executor.shutdown();
    }
}
```

#### C++ (Manual Implementation)

```cpp
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>
#include <vector>

class ThreadPool {
private:
    std::vector<std::thread> threads;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;

public:
    ThreadPool(size_t num_threads) : stop(false) {
        // Create worker threads
        for (size_t i = 0; i < num_threads; ++i) {
            threads.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(queue_mutex);
                        // Wait for task or stop signal
                        condition.wait(lock, [this] {
                            return stop || !tasks.empty();
                        });
                        
                        if (stop && tasks.empty())
                            return;
                        
                        // Get task from queue
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    // Execute task
                    task();
                }
            });
        }
    }

    // Add task to queue
    template<class F>
    void enqueue(F&& f) {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            tasks.emplace(std::forward<F>(f));
        }
        condition.notify_one();
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for (std::thread& thread : threads)
            thread.join();
    }
};

// Usage
int main() {
    ThreadPool pool(4);
    
    for (int i = 0; i < 10; ++i) {
        pool.enqueue([i] {
            std::cout << "Task " << i << " executing\n";
            std::this_thread::sleep_for(std::chrono::seconds(1));
        });
    }
    
    return 0;
}
```

### Best Practices

1. **Pool Size**:
   - CPU-bound tasks: pool size = number of cores
   - I/O-bound tasks: pool size > number of cores (2-4x typical)
   - Too many threads → excessive context switching
   - Too few threads → underutilized resources

2. **Task Granularity**:
   - Tasks should be independent
   - Avoid very short tasks (overhead exceeds benefit)
   - Avoid very long tasks (blocks threads)

3. **Error Handling**:
   - Catch exceptions in tasks
   - Don't let exceptions kill pool threads
   - Log errors for debugging

4. **Shutdown**:
   - Graceful shutdown: finish pending tasks
   - Forced shutdown: cancel pending tasks
   - Wait for completion before program exit

---

## Inter-Process Communication

Processes are isolated by design, but often need to communicate. The OS provides various mechanisms for safe inter-process communication (IPC).

### Process Address Spaces

**Virtual Memory Isolation:**

Each process sees its own virtual address space:
```
Process A Virtual Memory     Process B Virtual Memory
+-------------------+        +-------------------+
| Stack             |        | Stack             |
+-------------------+        +-------------------+
| ↓ grows down      |        | ↓ grows down      |
|                   |        |                   |
| ↑ grows up        |        | ↑ grows up        |
+-------------------+        +-------------------+
| Heap              |        | Heap              |
+-------------------+        +-------------------+
| BSS (uninit data) |        | BSS (uninit data) |
+-------------------+        +-------------------+
| Data (init data)  |        | Data (init data)  |
+-------------------+        +-------------------+
| Text (code)       |        | Text (code)       |
+-------------------+        +-------------------+
0x00000000                   0x00000000
```

**Can two processes access each other's address space?**

**Short answer**: No, not directly.

**Long answer**: 
- Each process has its own page table
- Virtual addresses in Process A map to different physical addresses than Process B
- MMU (Memory Management Unit) enforces this isolation
- Attempting to access another process's address causes segmentation fault

**However:**
- Same virtual address (e.g., 0x1000) in different processes maps to different physical memory
- Over time, physical memory can be reused
  - Process A terminates, frees physical pages
  - OS allocates same physical pages to Process B later
  - But at any given moment, each page belongs to only one process
- Special cases where processes CAN share memory (see below)

### Shared Libraries

**Problem**: If every process has its own copy of libc, system would waste memory.

**Solution**: Shared libraries (dynamic linking)

**How It Works:**

1. **Single Physical Copy**:
   - Shared library loaded once in physical memory
   - Example: libc.so loaded at physical address 0x12345000

2. **Multiple Virtual Mappings**:
   ```
   Process A:           Process B:           Process C:
   Virtual 0x7f123000   Virtual 0x7f456000   Virtual 0x7f789000
         ↓                    ↓                    ↓
         +--------------------+--------------------+
                              ↓
                  Physical 0x12345000 (libc.so)
   ```

3. **Each Process Has Different Virtual Address**:
   - Process A sees libc at 0x7f123000
   - Process B sees libc at 0x7f456000
   - Both point to same physical pages

4. **Memory Protection**:
   - Code pages: Read + Execute (shared)
   - Data pages: Copy-on-Write (COW)
   - If process modifies library data, gets private copy

**Benefits:**
- Huge memory savings (one copy instead of hundreds)
- Faster process creation (no need to copy library code)
- Updates to shared library benefit all processes

**Example:**
```bash
# See shared libraries used by a process
ldd /bin/ls
    linux-vdso.so.1 =>  (0x00007ffd3e9fe000)
    libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f8e6c1e5000)
    libc.so.6 => /lib64/libc.so.6 (0x00007f8e6be17000)
    ...
```

### IPC Mechanisms

#### 1. **Pipes**

Unidirectional data channel.

```c
int pipefd[2];
pipe(pipefd);  // pipefd[0] = read end, pipefd[1] = write end

if (fork() == 0) {
    // Child process
    close(pipefd[1]);  // Close write end
    char buffer[100];
    read(pipefd[0], buffer, sizeof(buffer));
    printf("Child received: %s\n", buffer);
} else {
    // Parent process
    close(pipefd[0]);  // Close read end
    write(pipefd[1], "Hello from parent", 18);
}
```

#### 2. **Named Pipes (FIFOs)**

Like pipes, but have a filesystem name.

```c
// Process A (writer)
mkfifo("/tmp/myfifo", 0666);
int fd = open("/tmp/myfifo", O_WRONLY);
write(fd, "Hello", 5);

// Process B (reader)
int fd = open("/tmp/myfifo", O_RDONLY);
char buffer[100];
read(fd, buffer, sizeof(buffer));
```

#### 3. **Message Queues**

Structured messages with priorities.

```c
// Create message queue
int msgid = msgget(IPC_PRIVATE, 0666 | IPC_CREAT);

// Send message
struct msgbuf {
    long mtype;
    char mtext[100];
} message;
message.mtype = 1;
strcpy(message.mtext, "Hello");
msgsnd(msgid, &message, sizeof(message.mtext), 0);

// Receive message
msgrcv(msgid, &message, sizeof(message.mtext), 1, 0);
```

#### 4. **Shared Memory**

Fastest IPC method - direct memory access.

```c
// Process A
int shmid = shmget(IPC_PRIVATE, 1024, IPC_CREAT | 0666);
char* shared = (char*)shmat(shmid, NULL, 0);
strcpy(shared, "Shared data");
shmdt(shared);

// Process B
char* shared = (char*)shmat(shmid, NULL, 0);
printf("Read: %s\n", shared);
shmdt(shared);
```

**Note**: Requires synchronization (semaphores/mutexes) to avoid race conditions!

#### 5. **Sockets**

Communication between processes on same or different machines.

```python
# Server process
import socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 8080))
server.listen(1)
conn, addr = server.accept()
data = conn.recv(1024)

# Client process
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 8080))
client.send(b'Hello')
```

#### 6. **Signals**

Asynchronous notifications.

```c
// Process A
signal(SIGUSR1, signal_handler);
pause();  // Wait for signal

// Process B
kill(process_a_pid, SIGUSR1);  // Send signal
```

### Parent-Child Processes

#### Creating a Child Process

**In Unix/Linux (using fork):**

```c
pid_t pid = fork();

if (pid == 0) {
    // Child process code
    printf("I am child, my PID is %d\n", getpid());
    printf("My parent's PID is %d\n", getppid());
} else if (pid > 0) {
    // Parent process code
    printf("I am parent, my PID is %d\n", getpid());
    printf("My child's PID is %d\n", pid);
} else {
    // fork failed
    perror("fork");
}
```

#### What Child Process Inherits

**Inherited:**
- Code (text segment) - shared, not copied
- Data segment (globals, static variables) - copied
- Heap - copied
- Stack - copied
- Environment variables
- Working directory
- File descriptors (open files)
- Signal handlers
- Process group ID
- Session ID
- Real and effective user/group IDs
- Resource limits

**Not Inherited:**
- Process ID (PID) - child gets new PID
- Parent PID (PPID) - child's PPID is parent's PID
- Locks held by parent
- Pending signals
- CPU time counters
- Timers
- Asynchronous I/O operations

#### Can Child Read/Write Parent Data?

**No, child cannot directly access parent's memory.**

After `fork()`:
1. Child gets copy of parent's data (separate address space)
2. Modifications in child don't affect parent
3. Modifications in parent don't affect child

**Example:**
```c
int shared_var = 100;

if (fork() == 0) {
    // Child
    shared_var = 200;
    printf("Child: shared_var = %d\n", shared_var);  // 200
} else {
    // Parent
    sleep(1);
    printf("Parent: shared_var = %d\n", shared_var);  // Still 100!
}
```

**To Share Data:**
- Use shared memory segment
- Use pipes/sockets
- Use memory-mapped files

### Copy-on-Write (COW)

**Problem**: Copying entire parent address space for child is expensive.

**Solution**: Copy-on-Write

**How It Works:**

1. **After fork()**:
   - Child doesn't get actual copy of memory
   - Both parent and child point to same physical pages
   - Pages marked as read-only

2. **On First Write**:
   - Either process tries to modify page
   - Page fault occurs (trying to write to read-only page)
   - OS creates actual copy of that page
   - Modifying process gets private copy
   - Other process keeps original

3. **Only Modified Pages Copied**:
   - If child never modifies data, no copy made
   - Huge memory savings
   - Faster fork() operation

**Illustration:**
```
Before any modifications:
Parent Virtual → Physical Page A (read-only)
Child Virtual  → Physical Page A (read-only)

After child modifies:
Parent Virtual → Physical Page A (read-only)
Child Virtual  → Physical Page B (read-write) [new copy]
```

**Example:**
```c
int data[1000000];  // Large array

pid_t pid = fork();

if (pid == 0) {
    // Child - if it just reads data, no copy made!
    printf("Sum: %d\n", sum(data));  // No copy
    
    // But if child modifies...
    data[0] = 42;  // NOW a copy is made (just for modified page)
}
```

**Dirty COW (CVE-2016-5195)**:

A critical Linux kernel security vulnerability discovered in 2016.

**The Bug:**
- Race condition in COW mechanism
- Allowed unprivileged user to modify read-only memory
- Could write to files owned by other users
- Could escalate privileges

**Impact:**
- Affected Linux kernels since 2007
- One of the most serious Linux vulnerabilities
- Allowed local privilege escalation

**How It Worked:**
1. Process maps a read-only file
2. Exploit race condition between:
   - COW page fault handler
   - Memory write operation
3. Wrote to physical page before COW copy completed
4. Modified original file instead of copy

### File Descriptors and Sockets

**File Descriptors are Inherited:**

When child process is created:
- Gets copy of parent's file descriptor table
- Same file descriptors point to same open files
- Shared file offset

**Example:**
```c
int fd = open("file.txt", O_RDWR);

if (fork() == 0) {
    // Child
    write(fd, "Child\n", 6);
} else {
    // Parent
    write(fd, "Parent\n", 7);
}
```

Both processes write to same file!

**Shared File Offset:**
```c
int fd = open("file.txt", O_RDONLY);
lseek(fd, 100, SEEK_SET);  // Move to position 100

if (fork() == 0) {
    // Child
    char buf[10];
    read(fd, buf, 10);  // Reads from position 100
    // After read, offset is at 110
}
// Parent sees offset at 110 too!
```

**Socket Example:**

```c
int server_sock = socket(AF_INET, SOCK_STREAM, 0);
bind(server_sock, ...);
listen(server_sock, 5);

int client_sock = accept(server_sock, ...);

if (fork() == 0) {
    // Child handles client
    close(server_sock);  // Child doesn't need server socket
    handle_client(client_sock);
    close(client_sock);
    exit(0);
} else {
    // Parent continues accepting
    close(client_sock);  // Parent doesn't need this client
}
```

**Preventing Interference:**

If you want to close a socket but other processes might still use it:

```c
// Bad: close() makes fd unavailable to all processes
close(sockfd);

// Good: shutdown() stops further I/O but fd still valid
shutdown(sockfd, SHUT_RDWR);
// Other processes can finish their operations
// Then close when done
close(sockfd);
```

**Shutdown options:**
- `SHUT_RD`: No more receives
- `SHUT_WR`: No more sends
- `SHUT_RDWR`: No more sends or receives

---

## Concurrency and Parallelism

Two related but distinct concepts in computing.

### Concurrency

**Definition**: Multiple tasks making progress in overlapping time periods.

**Key Points:**
- Tasks don't necessarily execute at the same instant
- Can be achieved with single CPU core through time-slicing
- About dealing with multiple things at once
- Structure: How you organize code to handle multiple tasks

**Example**: Single-core CPU running multiple programs
```
Time:    1    2    3    4    5    6    7    8
Task A:  [====]         [====]         [====]
Task B:       [====]         [====]
Task C:            [====]         [====]

Only one task runs at any instant, but all make progress
```

### Parallelism

**Definition**: Multiple tasks executing simultaneously.

**Key Points:**
- Requires multiple CPU cores
- Tasks actually execute at the same instant
- About doing multiple things at once
- Execution: How you run code on multiple processors

**Example**: Multi-core CPU
```
Time:    1    2    3    4    5    6    7    8
Core 1:  [=========Task A=========]
Core 2:  [=========Task B=========]
Core 3:  [=========Task C=========]
Core 4:  [=========Task D=========]

All tasks run simultaneously
```

### Single CPU Core

**Only Concurrency Possible:**
- OS uses time-slicing
- Rapid context switching creates illusion of parallelism
- Threads/processes take turns on CPU
- Good for I/O-bound tasks (waiting for disk, network)
- CPU-bound tasks don't benefit (still only one CPU doing work)

**Example:**
```python
# Two CPU-intensive tasks on single core
def task1():
    result = 0
    for i in range(1000000):
        result += i
    return result

def task2():
    result = 0
    for i in range(1000000):
        result += i * 2
    return result

# Sequential execution: T seconds
start = time.time()
task1()
task2()
print(f"Sequential: {time.time() - start}")

# Concurrent execution on single core: Still ~T seconds
import threading
start = time.time()
t1 = threading.Thread(target=task1)
t2 = threading.Thread(target=task2)
t1.start()
t2.start()
t1.join()
t2.join()
print(f"Concurrent: {time.time() - start}")
```

### Multiple CPU Cores

**Both Concurrency and Parallelism:**
- True parallel execution
- Each core runs different task
- CPU-bound tasks benefit significantly
- I/O-bound tasks also benefit (more cores waiting for I/O)

**Example:**
```python
# Same CPU-intensive tasks on multi-core
from multiprocessing import Process

# Parallel execution: ~T/2 seconds (if 2+ cores)
start = time.time()
p1 = Process(target=task1)
p2 = Process(target=task2)
p1.start()
p2.start()
p1.join()
p2.join()
print(f"Parallel: {time.time() - start}")
```

### Comparison Table

| Aspect | Concurrency | Parallelism |
|--------|-------------|-------------|
| **Definition** | Multiple tasks in progress | Multiple tasks executing simultaneously |
| **CPU Requirement** | Can work with single core | Requires multiple cores |
| **Execution** | Time-sliced interleaving | Simultaneous execution |
| **Purpose** | Handle multiple tasks | Speed up execution |
| **Best For** | I/O-bound tasks | CPU-bound tasks |
| **Example** | Web server handling requests | Video rendering on multiple cores |

### Real-World Example

**Web Server:**

**Concurrency** (handling multiple requests):
```python
def handle_request(request):
    # Parse request
    # Query database (I/O - wait)
    # Generate response
    # Send response (I/O - wait)

# One thread per request
# While one waits for I/O, others can process
```

**Parallelism** (using multiple cores):
```python
# Multiple worker processes
# Each process handles requests on separate core
# True parallel processing of requests
```

**Video Encoding:**

**Concurrency** (managing tasks):
- Organize work into chunks
- Coordinate between chunks
- Handle progress updates

**Parallelism** (actual speedup):
- Each core encodes different chunk
- 8 cores → ~8x faster encoding

### Visual Summary

```
Concurrency (Single Core):
Time: →
Core: [A][B][C][A][B][C][A][B]
      Tasks interleaved

Parallelism (Multi-Core):
Core 1: [========A========]
Core 2: [========B========]
Core 3: [========C========]
        Tasks simultaneous

Concurrent + Parallel (Multi-Core with many tasks):
Core 1: [A1][B1][C1][A1][B1]
Core 2: [A2][B2][C2][A2][B2]
Core 3: [A3][B3][C3][A3][B3]
        Many tasks, cores executing different ones
```

---

## Synchronization Mechanisms

When multiple processes/threads access shared resources, we need synchronization to maintain data consistency.

### Critical Zones

**Critical Section** (or Critical Region): A segment of code where a process accesses shared resources.

**Properties:**
1. **Mutual Exclusion**: Only one process in critical section at a time
2. **Progress**: If no process is in critical section, one waiting process must be allowed to enter
3. **Bounded Waiting**: There exists a limit on how many times other processes can enter critical section before a waiting process is granted access

**Structure:**
```c
do {
    // Entry Section
    acquire_lock();
    
    // Critical Section
    // Access shared resource
    balance = balance + 100;
    
    // Exit Section
    release_lock();
    
    // Remainder Section
    // Other work
} while (1);
```

**Example Critical Section:**
```c
// Shared variables
int balance = 0;
pthread_mutex_t lock;

void* deposit(void* arg) {
    // Non-critical work
    int amount = *(int*)arg;
    
    // CRITICAL SECTION START
    pthread_mutex_lock(&lock);
    int temp = balance;
    temp = temp + amount;
    balance = temp;
    pthread_mutex_unlock(&lock);
    // CRITICAL SECTION END
    
    return NULL;
}
```

### Race Conditions

**Definition**: When two or more processes/threads access shared data concurrently, and the final outcome depends on the particular order of execution.

**Classic Example - Bank Account:**
```c
int balance = 100;

// Thread 1                      // Thread 2
int temp = balance; // 100       int temp = balance; // 100
temp = temp + 50;   // 150       temp = temp + 30;   // 130
balance = temp;     // balance=150  balance = temp;     // balance=130

// Expected: 180, Actual: 130 or 150 (last write wins!)
```

**Why It Happens:**
1. Operations are not atomic
2. Multiple steps at assembly level
3. Context switch can occur between steps

**Assembly Level View:**
```assembly
; deposit(50)
LOAD R1, balance    ; R1 = 100
ADD R1, 50          ; R1 = 150
; ← Context switch here!
STORE R1, balance   ; balance = 150

; deposit(30) on another core
LOAD R2, balance    ; R2 = 100 (old value!)
ADD R2, 30          ; R2 = 130
STORE R2, balance   ; balance = 130 (overwrites 150!)
```

**Real-World Example:**
```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # NOT atomic! Load, add, store

threads = []
for _ in range(10):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(counter)  # Expected: 1000000, Actual: less (race condition!)
```

**Solutions to Race Conditions:**

1. **Mutual Exclusion (Locks)**
2. **Atomic Operations**
3. **Synchronization Primitives**

### Locking Mechanisms

#### 1. **Mutex (Mutual Exclusion)**

**Definition**: A lock that ensures only one thread can access a resource at a time.

**Properties:**
- Binary state: locked or unlocked
- Only one thread can hold mutex
- Thread that locks must unlock

**Usage:**
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* thread_function(void* arg) {
    pthread_mutex_lock(&mutex);
    // Critical section
    shared_resource++;
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

```python
import threading

mutex = threading.Lock()

def thread_function():
    mutex.acquire()
    try:
        # Critical section
        global shared_resource
        shared_resource += 1
    finally:
        mutex.release()

# Or using context manager
def thread_function():
    with mutex:
        # Critical section
        global shared_resource
        shared_resource += 1
```

**Recursive Mutex:**
- Allows same thread to lock multiple times
- Must unlock same number of times

```c
pthread_mutex_t recursive_mutex;
pthread_mutexattr_t attr;
pthread_mutexattr_init(&attr);
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
pthread_mutex_init(&recursive_mutex, &attr);

void recursive_function(int n) {
    pthread_mutex_lock(&recursive_mutex);
    if (n > 0) {
        recursive_function(n - 1);  // Can lock again
    }
    pthread_mutex_unlock(&recursive_mutex);
}
```

#### 2. **Semaphore**

**Definition**: A generalized mutex that allows N threads to access a resource simultaneously.

**Types:**
1. **Binary Semaphore**: Like mutex (0 or 1)
2. **Counting Semaphore**: Can have value > 1

**Operations:**
- `wait()` (P, down): Decrement, block if zero
- `signal()` (V, up): Increment

**Example - Limiting Connections:**
```c
#include <semaphore.h>

sem_t connection_limit;
sem_init(&connection_limit, 0, 5);  // Max 5 concurrent connections

void* handle_connection(void* arg) {
    sem_wait(&connection_limit);  // Acquire connection slot
    
    // Handle connection
    process_client();
    
    sem_post(&connection_limit);  // Release connection slot
    return NULL;
}
```

```python
from threading import Semaphore

# Limit to 5 concurrent downloads
download_semaphore = Semaphore(5)

def download_file(url):
    with download_semaphore:
        # Only 5 threads can be here simultaneously
        data = fetch(url)
        save(data)
```

**Producer-Consumer Problem:**
```c
sem_t empty_slots;  // Number of empty buffer slots
sem_t full_slots;   // Number of filled buffer slots
sem_t mutex;        // For buffer access

sem_init(&empty_slots, 0, BUFFER_SIZE);
sem_init(&full_slots, 0, 0);
sem_init(&mutex, 0, 1);

void* producer(void* arg) {
    while (1) {
        int item = produce_item();
        
        sem_wait(&empty_slots);  // Wait for empty slot
        sem_wait(&mutex);        // Lock buffer
        
        add_to_buffer(item);
        
        sem_post(&mutex);        // Unlock buffer
        sem_post(&full_slots);   // Signal full slot
    }
}

void* consumer(void* arg) {
    while (1) {
        sem_wait(&full_slots);   // Wait for full slot
        sem_wait(&mutex);        // Lock buffer
        
        int item = remove_from_buffer();
        
        sem_post(&mutex);        // Unlock buffer
        sem_post(&empty_slots);  // Signal empty slot
        
        consume_item(item);
    }
}
```

#### 3. **Spinlock**

**Definition**: A lock where thread continuously checks (spins) if the lock is available, instead of sleeping.

**Characteristics:**
- Busy-waiting (wastes CPU)
- No context switch overhead
- Good for very short critical sections
- Bad for long wait times

**Implementation:**
```c
typedef struct {
    int locked;
} spinlock_t;

void spin_lock(spinlock_t* lock) {
    while (__sync_lock_test_and_set(&lock->locked, 1)) {
        // Spin (busy-wait)
        while (lock->locked) {
            // CPU pause instruction (reduce power consumption)
            __asm__ __volatile__("pause");
        }
    }
}

void spin_unlock(spinlock_t* lock) {
    __sync_lock_release(&lock->locked);
}
```

**When to Use:**
- Critical section is very short (few instructions)
- Thread count ≤ CPU cores
- Running in kernel mode

**When NOT to Use:**
- Long critical sections
- Many threads competing
- User-space applications (waste CPU, hurt other processes)

**Comparison: Mutex vs Spinlock:**
```
Mutex:
Thread blocked → Scheduler → Context switch → Other thread runs
Cost: ~1-10 microseconds

Spinlock:
Thread spins → Checks lock → Checks lock → ... → Gets lock
Cost: Continuous CPU usage
```

#### 4. **Read-Write Lock**

**Definition**: Allows multiple readers OR one writer.

**Rules:**
- Multiple readers can hold lock simultaneously
- Only one writer can hold lock
- Writer has exclusive access (no readers, no other writers)

**Use Case:** Data read often, written rarely

**Example:**
```c
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

// Reader thread
void* reader(void* arg) {
    pthread_rwlock_rdlock(&rwlock);  // Shared lock
    
    // Read shared data
    int value = shared_data;
    
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}

// Writer thread
void* writer(void* arg) {
    pthread_rwlock_wrlock(&rwlock);  // Exclusive lock
    
    // Modify shared data
    shared_data = new_value;
    
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}
```

```python
import threading

class ReadWriteLock:
    def __init__(self):
        self.readers = 0
        self.lock = threading.Lock()
        self.write_lock = threading.Lock()
    
    def acquire_read(self):
        with self.lock:
            self.readers += 1
            if self.readers == 1:
                self.write_lock.acquire()
    
    def release_read(self):
        with self.lock:
            self.readers -= 1
            if self.readers == 0:
                self.write_lock.release()
    
    def acquire_write(self):
        self.write_lock.acquire()
    
    def release_write(self):
        self.write_lock.release()

# Usage
rwlock = ReadWriteLock()

def reader():
    rwlock.acquire_read()
    # Read data
    value = shared_data
    rwlock.release_read()

def writer():
    rwlock.acquire_write()
    # Write data
    shared_data = new_value
    rwlock.release_write()
```

**Performance Benefits:**
```
Without RW lock:
Reader 1: [wait][read][unlock]
Reader 2:      [wait][read][unlock]
Reader 3:           [wait][read][unlock]

With RW lock:
Reader 1: [read]
Reader 2: [read]  // Concurrent!
Reader 3: [read]  // Concurrent!
Writer:   [wait for all readers][write]
```

### Deadlocks

**Definition**: A situation where two or more processes are unable to proceed because each is waiting for the other to release a resource.

**Example:**
```
Process A:                  Process B:
Lock(Resource 1)            Lock(Resource 2)
  ...                         ...
Lock(Resource 2) ← WAIT     Lock(Resource 1) ← WAIT
  ...                         ...
Unlock(Resource 2)          Unlock(Resource 1)
Unlock(Resource 1)          Unlock(Resource 2)

Both processes wait forever!
```

**Code Example:**
```c
pthread_mutex_t lock1, lock2;

// Thread 1
void* thread1(void* arg) {
    pthread_mutex_lock(&lock1);
    sleep(1);  // Simulate work
    pthread_mutex_lock(&lock2);  // ← Waits for Thread 2
    
    // Critical section
    
    pthread_mutex_unlock(&lock2);
    pthread_mutex_unlock(&lock1);
    return NULL;
}

// Thread 2
void* thread2(void* arg) {
    pthread_mutex_lock(&lock2);
    sleep(1);  // Simulate work
    pthread_mutex_lock(&lock1);  // ← Waits for Thread 1
    
    // Critical section
    
    pthread_mutex_unlock(&lock1);
    pthread_mutex_unlock(&lock2);
    return NULL;
}
```

#### Necessary Conditions for Deadlock

All four must be true simultaneously:

1. **Mutual Exclusion**:
   - At least one resource must be held in non-sharable mode
   - Only one process can use the resource at a time

2. **Hold and Wait**:
   - Process holding at least one resource is waiting for additional resources
   - Held by other processes

3. **No Preemption**:
   - Resources cannot be forcibly taken away
   - Must be released voluntarily by holding process

4. **Circular Wait**:
   - Chain of processes exists where each waits for resource held by next
   - P1 waits for P2, P2 waits for P3, ..., Pn waits for P1

#### Deadlock Prevention

**Break one of the four conditions:**

**1. Eliminate Mutual Exclusion:**
- Make resources sharable (not always possible)
- Example: Read-only files can be shared

**2. Eliminate Hold and Wait:**
- Request all resources at once before execution
```c
void process() {
    lock_all_resources([lock1, lock2, lock3]);
    // Use resources
    unlock_all_resources([lock1, lock2, lock3]);
}
```

**Drawback**: Low resource utilization, may not know all resources needed in advance

**3. Allow Preemption:**
- If process can't get all resources, release held ones
```c
bool tryLockAll() {
    if (trylock(lock1)) {
        if (trylock(lock2)) {
            return true;  // Got both locks
        } else {
            unlock(lock1);  // Release first lock
            return false;
        }
    }
    return false;
}

void process() {
    while (!tryLockAll()) {
        sleep(random_backoff());  // Wait and retry
    }
    // Critical section
    unlock(lock2);
    unlock(lock1);
}
```

**4. Eliminate Circular Wait:**
- Impose total ordering on resources
- Always acquire locks in same order
```c
// Resource ordering: lock1 < lock2 < lock3

// Thread 1
lock(lock1);
lock(lock2);
// Critical section
unlock(lock2);
unlock(lock1);

// Thread 2  
lock(lock1);  // Same order!
lock(lock2);
// Critical section
unlock(lock2);
unlock(lock1);
```

#### Deadlock Avoidance

**Banker's Algorithm**:
- System decides if granting request will lead to deadlock
- Only grants if system remains in "safe state"

**Safe State**: There exists a sequence of processes that can all complete

**Example:**
```
Resources: 10 total
Currently allocated: A=2, B=3, C=1 (total=6, available=4)

A needs max 5 (needs 3 more)
B needs max 4 (needs 1 more)
C needs max 6 (needs 5 more)

Is this safe?
- B can finish (needs 1, we have 4) → releases 3 → available=7
- A can finish (needs 3, we have 7) → releases 2 → available=9
- C can finish (needs 5, we have 9) → releases 1 → available=10
Yes, safe state!
```

#### Deadlock Detection and Recovery

**Detection:**
- Maintain resource allocation graph
- Periodically check for cycles
```
P1 → R1 → P2 → R2 → P1  (cycle = deadlock)
```

**Recovery Options:**
1. **Kill one process** to break cycle
2. **Preempt resources** from processes
3. **Rollback** processes to previous safe state
4. **Kill all deadlocked processes**

#### Best Practices

1. **Lock Ordering**:
   ```c
   // Always lock in same order
   void transfer(account_t* from, account_t* to, int amount) {
       account_t* first = (from < to) ? from : to;
       account_t* second = (from < to) ? to : from;
       
       lock(first);
       lock(second);
       // Transfer
       unlock(second);
       unlock(first);
   }
   ```

2. **Timeout**:
   ```c
   if (pthread_mutex_timedlock(&lock, &timeout) != 0) {
       // Couldn't get lock, handle error
       return;
   }
   ```

3. **Try-Lock**:
   ```c
   if (pthread_mutex_trylock(&lock) != 0) {
       // Couldn't get lock, release others and retry
       release_held_locks();
       retry();
   }
   ```

4. **Minimize Lock Scope**:
   ```c
   // Bad: Long critical section
   lock();
   complex_calculation();
   shared_var++;
   more_work();
   unlock();
   
   // Good: Short critical section
   complex_calculation();
   lock();
   shared_var++;
   unlock();
   more_work();
   ```

---

## Memory Management

### Virtual Memory

**Definition**: A memory management technique that provides an abstraction of the storage resources actually available, giving the illusion of a very large, contiguous memory space.

**Why We Need Virtual Memory:**

1. **More Memory Than Physical RAM**:
   - Programs can use more memory than physically available
   - System uses disk as extension of RAM
   - Allows running large applications on limited RAM

2. **Process Isolation**:
   - Each process has own virtual address space
   - Cannot access another process's memory
   - Security and stability

3. **Simplified Programming**:
   - Programmers don't worry about physical memory locations
   - No need to manage fragmentation
   - Consistent memory layout across processes

4. **Efficient Memory Use**:
   - Load only needed parts of program (paging)
   - Share code between processes (shared libraries)
   - Better multiprogramming

#### How Virtual Memory Works

**Address Translation:**
```
Virtual Address (Process sees)
         ↓
    [Page Table]
         ↓
Physical Address (RAM location)
```

**Example:**
```
Process A sees:
Virtual 0x0000 → Physical 0x5000
Virtual 0x1000 → Physical 0x3000
Virtual 0x2000 → Physical 0x8000

Process B sees:
Virtual 0x0000 → Physical 0x7000
Virtual 0x1000 → Physical 0x2000
Virtual 0x2000 → Physical 0x6000

Same virtual addresses map to different physical addresses!
```

**Components:**

1. **Virtual Address**:
   - Address used by program
   - Divided into: Page Number + Offset

2. **Physical Address**:
   - Actual RAM location
   - Divided into: Frame Number + Offset

3. **Page Table**:
   - Maps virtual pages to physical frames
   - One per process
   - Stored in memory

4. **MMU (Memory Management Unit)**:
   - Hardware that translates addresses
   - Uses page table
   - Handles page faults

**Page Fault:**
When program accesses page not in physical memory:
```
1. CPU tries to access virtual address
2. MMU checks page table
3. Page not in memory (page fault)
4. OS interrupt handler runs
5. OS loads page from disk to RAM
6. OS updates page table
7. Instruction restarted
8. Access succeeds
```

**Virtual Memory Size:**

**Theoretical Limit:**
- 32-bit system: 2^32 = 4GB per process
- 64-bit system: 2^64 = 16 exabytes per process

**Practical Limit:**
- Limited by available disk space
- Limited by OS configuration
- Limited by swap space

**Example:**
```
Physical RAM: 8GB
Swap space: 16GB
Total virtual memory: 24GB

Can run processes totaling 24GB memory usage
(though performance degrades with heavy swapping)
```

### Paging

**Definition**: Memory management scheme that eliminates need for contiguous allocation of physical memory.

**Concept:**
- Divide virtual memory into fixed-size blocks (pages)
- Divide physical memory into fixed-size blocks (frames)
- Page size = Frame size (typically 4KB)

**Page Structure:**
```
Virtual Address (32-bit):
[Page Number: 20 bits][Offset: 12 bits]
                      ↓
               [Page Table]
                      ↓
Physical Address:
[Frame Number: 20 bits][Offset: 12 bits]
```

**Page Table Entry:**
```
+-------+-------+-------+-------+----------+
| Valid | Frame | Dirty | Ref   | Prot     |
| Bit   | Number| Bit   | Bit   | Bits     |
+-------+-------+-------+-------+----------+

Valid: 1 = page in memory, 0 = on disk
Frame Number: Physical frame location
Dirty: 1 = modified, needs write back
Ref: 1 = recently accessed (for replacement)
Prot: Read/Write/Execute permissions
```

**Translation Process:**
```c
// Virtual address: 0x12345
// Page size: 4096 bytes (0x1000)

page_number = virtual_address / page_size;  // 0x12345 / 0x1000 = 0x12
offset = virtual_address % page_size;       // 0x12345 % 0x1000 = 0x345

page_table_entry = page_table[page_number];

if (page_table_entry.valid) {
    frame_number = page_table_entry.frame;
    physical_address = frame_number * page_size + offset;
} else {
    // Page fault - load from disk
    handle_page_fault();
}
```

**Advantages:**
1. No external fragmentation
2. Easy to swap (pages are fixed size)
3. Simple to allocate
4. Efficient memory use

**Disadvantages:**
1. Internal fragmentation (last page may not be full)
2. Page table takes space
3. Two memory accesses (page table + data)

**Optimization - TLB (Translation Lookaside Buffer):**
- Cache for page table entries
- Very fast (CPU register speed)
- Holds recently used translations

```
Virtual Address
     ↓
   [TLB] ← Very fast
     |
     | TLB Miss
     ↓
 [Page Table] ← Slower
     ↓
Physical Address
```

**With TLB:**
- TLB hit: 1 memory access (data only)
- TLB miss: 2 memory accesses (page table + data)
- Typical TLB hit rate: 95-99%

### Memory Mapping

**Can Two Processes Map to Same Physical Address?**

**Yes!** This is fundamental to shared memory and shared libraries.

**Scenarios:**

#### 1. Shared Libraries
```
Process A:                Process B:
Virtual 0x7f000000        Virtual 0x7f800000
        ↓                         ↓
        +-------------------------+
                    ↓
        Physical 0x12340000 (libc.so)
```

Both processes access same physical memory (libc code).

#### 2. Shared Memory
```c
// Process A
int shmid = shmget(KEY, SIZE, IPC_CREAT | 0666);
void* ptr_a = shmat(shmid, NULL, 0);  // Virtual address 1

// Process B
int shmid = shmget(KEY, SIZE, 0);
void* ptr_b = shmat(shmid, NULL, 0);  // Virtual address 2

// ptr_a and ptr_b are different virtual addresses
// but map to SAME physical memory
```

**Write in one process, read in another:**
```c
// Process A
*(int*)ptr_a = 42;

// Process B
printf("%d\n", *(int*)ptr_b);  // Prints 42!
```

#### 3. Parent-Child After Fork
```c
int shared_var = 100;
fork();

// Before any write (COW):
// Parent virtual → Physical page A
// Child virtual  → Physical page A (SAME!)

// After write:
// Parent virtual → Physical page A
// Child virtual  → Physical page B (DIFFERENT)
```

**Conditions for Sharing:**
1. Explicit request (shared memory, mmap with MAP_SHARED)
2. Read-only data (code sections, shared libraries)
3. Copy-on-Write before modification

**Concurrent Access:**
- Processes can't map same physical address and modify simultaneously without issues
- Must use synchronization (semaphores, mutexes)
- OS doesn't prevent concurrent writes
- Programmer responsible for safety

### Heap vs Stack

Two primary areas of memory for program data.

#### Stack

**Characteristics:**
- Fixed maximum size (typically 1-8 MB)
- Grows automatically
- Very fast allocation (just move pointer)
- LIFO (Last In, First Out) structure
- Stores local variables, function parameters, return addresses

**Memory Layout:**
```
High Address
+------------------+
| Stack            |
|        ↓         | (grows down)
|                  |
|                  |
|        ↑         | (grows up)
| Heap             |
+------------------+
| BSS (uninit)     |
+------------------+
| Data (init)      |
+------------------+
| Text (code)      |
+------------------+
Low Address
```

**Stack Frame:**
```
function_a() {
    int x = 5;           [x=5]
    function_b(x);       [return addr][param=5]
}                        [y=10][z=15]
                         [return addr][param=20]
function_b(int param) {  
    int y = 10;
    int z = 15;
    function_c(20);
}

function_c(int param) {
    // Current stack state
}
```

**Stack Operations:**
```c
void function_call() {
    int local = 42;        // Push onto stack
    other_function(local); // Push return address and parameters
    // Return: Pop parameters and return address
    // On exit: Pop local
}
```

**Stack Overflow:**
```c
void recursive_function() {
    int array[1000];  // Allocate on stack
    recursive_function();  // Infinite recursion
}
// Stack grows until limit hit → Stack Overflow
```

#### Heap

**Characteristics:**
- Large size (limited by virtual memory)
- Manual management (malloc/free, new/delete)
- Slower allocation (need to find free block)
- Stays allocated until explicitly freed
- Stores dynamically allocated data

**Heap Allocation:**
```c
// Request memory from heap
int* array = (int*)malloc(1000 * sizeof(int));

// Use memory
array[0] = 42;

// Must free when done
free(array);
```

**Heap Structure (Simplified):**
```
+------------------+
| Block 1 (used)   | 
+------------------+
| Block 2 (free)   |
+------------------+
| Block 3 (used)   |
+------------------+
| Block 4 (free)   |
+------------------+
```

Each block has metadata:
```
[Size][In Use?][Previous][Next][Data...]
```

#### Comparison

| Aspect | Stack | Heap |
|--------|-------|------|
| **Allocation Speed** | Very fast (move pointer) | Slower (find free block) |
| **Deallocation** | Automatic (on function return) | Manual (free/delete) |
| **Size** | Small (1-8 MB typical) | Large (GB+) |
| **Growth** | Automatic | Manual |
| **Access Pattern** | LIFO | Random |
| **Fragmentation** | None | Can fragment |
| **Lifetime** | Function scope | Until freed |
| **Thread Safety** | Each thread has own stack | Shared (needs synchronization) |

**When to Use Each:**

**Stack:**
- Small, short-lived data
- Local variables
- Function parameters
- Return addresses
- Known size at compile time

```c
void function() {
    int x = 5;          // Stack
    int arr[100];       // Stack
    char c = 'a';       // Stack
}
```

**Heap:**
- Large data structures
- Variable-size data
- Long-lived data (survives function return)
- Shared data between functions

```c
void function() {
    int* large_array = malloc(1000000 * sizeof(int));  // Heap
    char* string = malloc(length + 1);                 // Heap
    return large_array;  // Can return heap data
}
```

### Memory Allocation

#### Dynamic Allocation

**Process:**
1. Program requests memory from OS
2. OS finds suitable memory region
3. OS marks region as allocated
4. Returns pointer to region
5. Program uses memory
6. Program returns memory to OS

**C/C++:**
```c
// Allocate
int* ptr = (int*)malloc(sizeof(int) * 100);
if (ptr == NULL) {
    // Allocation failed
    handle_error();
}

// Use
ptr[0] = 42;

// Deallocate
free(ptr);
ptr = NULL;  // Good practice
```

**How malloc() Works:**
1. Check free list for suitable block
2. If found:
   - Split block if too large
   - Mark as used
   - Return pointer
3. If not found:
   - Request more memory from OS (sbrk/mmap)
   - Add to heap
   - Return pointer

**Allocation Strategies:**

**First Fit:**
- Use first free block large enough
- Fast
- Can cause fragmentation

**Best Fit:**
- Use smallest free block that fits
- Minimizes wasted space
- Slow (must search all blocks)

**Worst Fit:**
- Use largest free block
- Leaves largest remainder
- Can cause fragmentation

#### Deallocation

**How free() Works:**
1. Mark block as free
2. Merge with adjacent free blocks (coalescing)
3. Add to free list

```c
Before free(B):
[Used A][Used B][Used C]

After free(B):
[Used A][Free B][Used C]

After free(C):
[Used A][Free B+C]  ← Coalesced

After free(A):
[Free A+B+C]  ← All coalesced
```

**Deallocation Errors:**

**Double Free:**
```c
free(ptr);
free(ptr);  // ERROR! Undefined behavior
```

**Use After Free:**
```c
free(ptr);
ptr[0] = 42;  // ERROR! Accessing freed memory
```

**Memory Leak:**
```c
int* ptr = malloc(sizeof(int));
ptr = malloc(sizeof(int));  // Lost reference to first allocation!
// First allocation never freed
```

**Correct Pattern:**
```c
int* ptr = malloc(sizeof(int) * 100);
if (ptr == NULL) {
    return ERROR;
}

// Use ptr...

free(ptr);
ptr = NULL;
```

#### What Happens When Memory is Full?

**Physical Memory Full:**
1. **Swapping**: Move inactive pages to disk
2. **OOM Killer** (Linux): Kill processes to free memory
3. **Process Killed**: System chooses process to terminate
4. **Application Crash**: malloc() returns NULL

**System Response:**
```
1. malloc() request
2. No free memory
3. Try to reclaim (swap out, drop caches)
4. Still no memory:
   - malloc() returns NULL (C/C++)
   - throw bad_alloc exception (C++)
   - OOM killer activates (Linux)
```

**OOM Killer Selection:**
- Scores each process
- Considers: memory usage, runtime, nice value, root privileges
- Kills highest scoring process
- Typical targets: memory-heavy, non-critical processes

**Prevention:**
- Monitor memory usage
- Set memory limits (ulimit, cgroups)
- Handle allocation failures gracefully
- Free memory when done
- Use resource-aware programming

### Garbage Collection

**Definition**: Automatic memory management that reclaims memory occupied by objects no longer in use.

**Languages with GC**: Java, Python, JavaScript, Go, C#

**How GC Works:**

#### 1. Mark-and-Sweep

**Algorithm:**
```
Mark Phase:
1. Start from root objects (stack variables, globals)
2. Mark each reachable object
3. Recursively mark objects referenced by marked objects

Sweep Phase:
1. Scan all heap objects
2. Collect unmarked objects (garbage)
3. Return memory to free list
```

**Example:**
```
Roots: [A, C]

Objects:
A → B → D
C → E
F (not reachable)

Mark Phase:
A: marked (root)
B: marked (A → B)
D: marked (B → D)
C: marked (root)
E: marked (C → E)
F: unmarked

Sweep Phase:
F is garbage, deallocate
```

#### 2. Reference Counting

**Algorithm:**
- Each object has reference count
- Increment when reference created
- Decrement when reference destroyed
- When count reaches 0, deallocate

**Example:**
```python
# Python uses reference counting + cycle detection

a = Object()      # refcount = 1
b = a             # refcount = 2
c = a             # refcount = 3
del a             # refcount = 2
del b             # refcount = 1
del c             # refcount = 0 → deallocate
```

**Problem: Circular References:**
```python
class Node:
    def __init__(self):
        self.next = None

a = Node()
b = Node()
a.next = b
b.next = a  # Circular reference!

del a
del b
# Objects still reference each other
# refcount never reaches 0 → memory leak
```

#### 3. Generational GC

**Observation**: Most objects die young

**Strategy:**
- Divide heap into generations
- Young generation: new objects
- Old generation: survived multiple collections
- Collect young generation frequently
- Collect old generation rarely

**Example (Java):**
```
Young Generation:
- Eden: New objects created here
- Survivor 0: Survived one GC
- Survivor 1: Survived two GCs

Objects that survive many GCs promoted to Old Generation
```

#### When GC Runs

**Triggers:**

1. **Heap Full**: No space for new allocation
2. **Periodic**: Run every N seconds
3. **Explicit**: Manual invocation
4. **Idle Time**: When application idle

**Example (Python):**
```python
import gc

# GC runs automatically when threshold reached
# Default: 700 allocations trigger generation 0 collection

# Manual GC
gc.collect()  # Force collection

# Disable GC
gc.disable()

# Enable GC
gc.enable()
```

**Example (Java):**
```java
// GC runs automatically

// Suggest GC (not guaranteed)
System.gc();

// JVM flags to control GC:
// -Xms256m (initial heap)
// -Xmx1024m (max heap)
// -XX:+UseG1GC (use G1 collector)
```

#### GC Performance Impact

**Pause Time:**
- GC stops application (stop-the-world)
- Can cause latency spikes

**Concurrent GC:**
- Runs alongside application
- Reduces pause time
- More CPU overhead

**Example Collectors:**
- **Serial GC**: Simple, single-threaded
- **Parallel GC**: Multi-threaded, throughput focused
- **CMS**: Concurrent, low pause
- **G1**: Generational, balanced
- **ZGC**: Ultra-low pause (sub-millisecond)

---

## Pointers and Memory

### What is a Pointer?

**Definition**: A variable that stores the memory address of another variable.

**Visual:**
```
Variable: int x = 42;
Memory:   [Address: 0x1000][Value: 42]

Pointer: int* ptr = &x;
Memory:  [Address: 0x2000][Value: 0x1000]
                              ↓
                     Points to x's address
```

**Syntax:**
```c
int x = 42;          // Regular variable
int* ptr = &x;       // Pointer to int, & = address-of
int value = *ptr;    // Dereference, * = value-at-address

printf("x value: %d\n", x);          // 42
printf("x address: %p\n", &x);       // 0x1000
printf("ptr value: %p\n", ptr);      // 0x1000
printf("ptr points to: %d\n", *ptr); // 42
```

### Pass by Value vs Pass by Reference

#### Pass by Value

Copy of value is passed to function.

```c
void increment(int x) {
    x = x + 1;  // Modifies local copy only
}

int main() {
    int num = 5;
    increment(num);
    printf("%d\n", num);  // Still 5!
    return 0;
}
```

**What happens:**
```
main():  num = 5
         ↓ (copy value)
increment(x): x = 5
              x = 6  (local copy)
         ↓ (return)
main():  num = 5  (unchanged)
```

#### Pass by Reference

Address is passed, allowing modification.

```c
void increment(int* x) {
    *x = *x + 1;  // Modifies original variable
}

int main() {
    int num = 5;
    increment(&num);  // Pass address
    printf("%d\n", num);  // Now 6!
    return 0;
}
```

**What happens:**
```
main():  num = 5 (address 0x1000)
         ↓ (pass address 0x1000)
increment(x): x = 0x1000
              *x = 6  (writes to 0x1000)
         ↓ (return)
main():  num = 6  (modified!)
```

#### C++ References

Cleaner syntax for pass-by-reference:

```cpp
void increment(int& x) {  // Reference parameter
    x = x + 1;  // No need for *
}

int main() {
    int num = 5;
    increment(num);  // No need for &
    std::cout << num << std::endl;  // 6
    return 0;
}
```

### Pointer Arithmetic

```c
int array[5] = {10, 20, 30, 40, 50};
int* ptr = array;  // Points to first element

printf("%d\n", *ptr);      // 10
printf("%d\n", *(ptr+1));  // 20 (moves sizeof(int) bytes)
printf("%d\n", *(ptr+2));  // 30

ptr++;  // Move to next element
printf("%d\n", *ptr);      // 20
```

**Memory layout:**
```
Address:  0x1000    0x1004    0x1008    0x100C    0x1010
Value:    [  10  ]  [  20  ]  [  30  ]  [  40  ]  [  50  ]
          ↑
          ptr
          
ptr+1 → 0x1004 (not 0x1001!)
ptr+2 → 0x1008
```

---

## System Architecture

### Linux File Philosophy

**"Everything is a file" in Linux**

**What it means:**
- Devices are files
- Processes are files (/proc)
- Network sockets are files
- Pipes are files
- Everything accessible through file descriptors

**More accurately**: "Everything is a stream of bytes"

**Examples:**
```bash
# Regular file
cat file.txt

# Device file
cat /dev/random  # Random data
echo "text" > /dev/tty  # Terminal output

# Process info
cat /proc/cpuinfo  # CPU information
cat /proc/meminfo  # Memory information
cat /proc/123/status  # Process 123 status

# Hardware
cat /dev/sda  # Hard disk
echo 1 > /sys/class/leds/led0/brightness  # LED control
```

**Benefits:**
1. Uniform interface (open, read, write, close)
2. Simple to use
3. Composability (pipes, redirection)
4. Text-based configuration

### File Descriptors

**Definition**: An integer that uniquely identifies an open file or I/O resource in a process.

**Standard File Descriptors:**
- 0: stdin (standard input)
- 1: stdout (standard output)
- 2: stderr (standard error)

**File Descriptor Table:**
```
Process A:
FD    File
0  →  /dev/pts/0 (terminal)
1  →  /dev/pts/0 (terminal)
2  →  /dev/pts/0 (terminal)
3  →  /home/user/file.txt
4  →  /var/log/app.log
5  →  Socket [TCP 0.0.0.0:8080]
```

**Usage:**
```c
// Open file
int fd = open("/path/to/file", O_RDWR);
if (fd == -1) {
    perror("open failed");
    return 1;
}

// Read from file
char buffer[1024];
ssize_t bytes_read = read(fd, buffer, sizeof(buffer));

// Write to file
const char* text = "Hello, World!";
ssize_t bytes_written = write(fd, text, strlen(text));

// Close file
close(fd);
```

**Redirection:**
```bash
# Redirect stdout to file
command > output.txt  # FD 1 → output.txt

# Redirect stderr to file
command 2> errors.txt  # FD 2 → errors.txt

# Redirect both
command > output.txt 2>&1

# Redirect to /dev/null (discard)
command > /dev/null 2>&1
```

### How Devices Communicate

**Mouse/Keyboard/Monitor communicate via:**

1. **Device Files** (character/block devices)
2. **Device Drivers** (kernel modules)
3. **Interrupts** (hardware signals)
4. **I/O Ports** (memory-mapped or port-mapped)

**Example - Keyboard:**
```
1. Key pressed
2. Keyboard controller sends interrupt
3. CPU stops current task
4. Interrupt handler (driver) reads key code
5. Data written to input buffer
6. Application reads from /dev/input/event0
7. Application processes key
```

**Example - Mouse:**
```
1. Mouse moved/clicked
2. USB controller sends data
3. USB driver processes data
4. Writes to /dev/input/mouse0
5. Application reads mouse events
```

**Example - Monitor:**
```
1. Application writes to framebuffer /dev/fb0
2. GPU driver receives data
3. GPU renders to video memory
4. Monitor displays pixels
```

### Buffers

**Definition**: Temporary storage area for data being transferred between two entities.

**Why Buffers?**

1. **Speed Mismatch**:
   - CPU very fast, I/O devices slow
   - Buffer smooths out differences

2. **Efficiency**:
   - Batch multiple operations
   - Reduce system calls

3. **Data Integrity**:
   - Ensure complete data transfer
   - Handle partial reads/writes

**Types:**

#### Input Buffer
```c
// Without buffer: Each character triggers system call
while ((c = getchar()) != EOF) {  // Inefficient!
    // System call for EACH character
}

// With buffer: Read block at once
char buffer[4096];
fread(buffer, 1, sizeof(buffer), stdin);  // One system call
```

#### Output Buffer
```c
// Without buffer
for (int i = 0; i < 1000; i++) {
    putchar('a');  // 1000 system calls!
}

// With buffer
char buffer[1000];
memset(buffer, 'a', 1000);
fwrite(buffer, 1, 1000, stdout);  // 1 system call!

// Or use stdio buffering
for (int i = 0; i < 1000; i++) {
    putchar('a');  // Buffered automatically
}
fflush(stdout);  // Flush buffer
```

#### Circular Buffer (Ring Buffer)
```c
struct circular_buffer {
    char data[BUFFER_SIZE];
    int read_pos;
    int write_pos;
};

void write_to_buffer(struct circular_buffer* buf, char c) {
    buf->data[buf->write_pos] = c;
    buf->write_pos = (buf->write_pos + 1) % BUFFER_SIZE;
}

char read_from_buffer(struct circular_buffer* buf) {
    char c = buf->data[buf->read_pos];
    buf->read_pos = (buf->read_pos + 1) % BUFFER_SIZE;
    return c;
}
```

**Buffering Strategies:**

**Unbuffered**: Each I/O operation is system call
```c
// stderr is unbuffered by default
fprintf(stderr, "Error!");  // Immediate output
```

**Line Buffered**: Flush on newline
```c
// stdout is line-buffered for terminals
printf("Hello\n");  // Flushed immediately
printf("World");    // Not flushed until \n or fflush()
```

**Fully Buffered**: Flush when buffer full
```c
// Files are fully buffered
FILE* f = fopen("file.txt", "w");
fprintf(f, "text");  // Stays in buffer
fclose(f);           // Flushed on close
```

### What Happens When 2 Processes Read/Write Same File?

**Scenario 1: Separate open() calls**

```c
// Process A
int fd_a = open("file.txt", O_RDWR);

// Process B
int fd_b = open("file.txt", O_RDWR);
```

**Result:**
- Different file descriptors
- Different file offsets
- Independent reading/writing
- Changes visible to both (eventually)

```c
// Process A
write(fd_a, "Hello", 5);

// Process B
lseek(fd_b, 0, SEEK_SET);
char buf[10];
read(fd_b, buf, 10);  // Reads "Hello" (after sync)
```

**Scenario 2: Shared file descriptor (fork)**

```c
int fd = open("file.txt", O_RDWR);
if (fork() == 0) {
    // Child uses fd
    write(fd, "Child", 5);
} else {
    // Parent uses fd
    write(fd, "Parent", 6);
}
```

**Result:**
- Same file descriptor in both processes
- Shared file offset
- Writes interleaved
- No synchronization → undefined order

**File Locking:**
```c
// Advisory locking (cooperative)
struct flock lock;
lock.l_type = F_WRLCK;  // Write lock
lock.l_whence = SEEK_SET;
lock.l_start = 0;
lock.l_len = 0;  // Lock whole file

fcntl(fd, F_SETLKW, &lock);  // Wait for lock
// Critical section
write(fd, data, size);
lock.l_type = F_UNLCK;
fcntl(fd, F_SETLK, &lock);  // Release lock
```

### System Calls

**Definition**: Programmatic interface to request services from the operating system kernel.

**Purpose:**
- Access hardware
- Perform privileged operations
- Manage resources
- Interact with kernel

**Common System Calls:**

**Process Control:**
- `fork()`: Create new process
- `exec()`: Execute program
- `wait()`: Wait for child
- `exit()`: Terminate process

**File Operations:**
- `open()`: Open file
- `read()`: Read from file
- `write()`: Write to file
- `close()`: Close file

**Memory:**
- `brk()`: Change data segment size
- `mmap()`: Map memory

**Communication:**
- `pipe()`: Create pipe
- `socket()`: Create socket
- `send()`: Send data
- `recv()`: Receive data

**How to Make a System Call:**

**Direct (Assembly):**
```assembly
; Linux x86-64 system call
mov rax, 1        ; syscall number (1 = write)
mov rdi, 1        ; file descriptor (1 = stdout)
mov rsi, message  ; buffer address
mov rdx, 13       ; message length
syscall           ; invoke system call
```

**C Library Wrapper:**
```c
// C library provides wrapper functions
ssize_t bytes = write(STDOUT_FILENO, "Hello\n", 6);
// Internally calls syscall
```

**What Happens During System Call:**

```
1. Application code (user space)
   ↓ (system call)
2. CPU switches to kernel mode
3. Kernel validates parameters
4. Kernel performs operation
5. Kernel returns result
6. CPU switches back to user mode
7. Application continues
```

**CPU Mode Switch:**
- User mode: Limited privileges
- Kernel mode: Full hardware access
- Switch via special instruction (syscall, int 0x80)
- Context saved/restored

**System Call Handler:**
```c
// Simplified kernel code
long sys_write(int fd, const char* buf, size_t count) {
    // Validate parameters
    if (!is_valid_fd(fd)) return -EBADF;
    if (!is_valid_address(buf)) return -EFAULT;
    
    // Get file structure
    struct file* file = get_file(fd);
    
    // Perform write
    ssize_t bytes = file->f_op->write(file, buf, count);
    
    return bytes;
}
```

**Performance Considerations:**
- System calls are expensive (mode switch overhead)
- Minimize system calls
- Batch operations when possible
- Use buffering

### User Space vs Kernel Space

**Memory is divided into two regions:**

```
High Memory
+------------------+
| Kernel Space     |  ← Kernel code, drivers, data
|                  |    Only accessible in kernel mode
| 0xC0000000       |
+------------------+
| User Space       |  ← Application code, data
|                  |    Accessible in user mode
| 0x00000000       |
+------------------+
Low Memory
```

**User Space:**
- Where application code runs
- Limited privileges
- Cannot directly access hardware
- Cannot access kernel memory
- Must use system calls for privileged operations

**Kernel Space:**
- Where OS kernel runs
- Full hardware access
- Manages all resources
- Handles system calls
- Runs device drivers

**Protection:**
- CPU has privilege levels (rings)
- Ring 0: Kernel mode (highest privilege)
- Ring 3: User mode (lowest privilege)
- Attempting privileged operation in user mode → exception

**Why Separation?**
1. **Security**: Applications can't interfere with kernel
2. **Stability**: Bug in application doesn't crash system
3. **Protection**: Processes isolated from each other

---

## Caching

### In-Memory Caching

**Definition**: Store frequently accessed data in RAM for fast retrieval, reducing database/disk access.

**Purpose:**
- Speed up data access
- Reduce database load
- Improve application performance
- Scale to more users

**Popular Systems:**
- **Redis**: In-memory data structure store
- **Memcached**: Distributed memory caching
- **Hazelcast**: Distributed in-memory data grid

**Architecture:**
```
Application
    ↓
   [Cache] ← Very fast (microseconds)
    ↓ (cache miss)
 [Database] ← Slower (milliseconds)
```

**Cache Patterns:**

#### 1. Cache-Aside (Lazy Loading)
```python
def get_user(user_id):
    # Try cache first
    user = cache.get(f"user:{user_id}")
    if user:
        return user  # Cache hit
    
    # Cache miss - load from database
    user = database.query(f"SELECT * FROM users WHERE id={user_id}")
    
    # Store in cache
    cache.set(f"user:{user_id}", user, ttl=3600)
    
    return user
```

#### 2. Write-Through
```python
def update_user(user_id, data):
    # Update database
    database.update(user_id, data)
    
    # Update cache
    cache.set(f"user:{user_id}", data, ttl=3600)
```

#### 3. Write-Behind (Write-Back)
```python
def update_user(user_id, data):
    # Update cache immediately
    cache.set(f"user:{user_id}", data, ttl=3600)
    
    # Queue database update (async)
    queue.add_task("update_db", user_id, data)
```

**Redis Example:**
```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

# Set value with expiration
r.setex("session:123", 3600, json.dumps(session_data))

# Get value
session = json.loads(r.get("session:123"))

# Delete
r.delete("session:123")

# Increment counter
r.incr("page_views")
```

### LRU Cache

**Least Recently Used**: Eviction policy that removes least recently accessed items when cache is full.

**Data Structure:**
- HashMap: O(1) lookup
- Doubly Linked List: O(1) move to front/remove

**Implementation:**
```python
class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}  # key -> node
        self.head = Node(0, 0)  # Dummy head
        self.tail = Node(0, 0)  # Dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def get(self, key):
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._add(node)  # Move to front (most recent)
            return node.value
        return -1
    
    def put(self, key, value):
        if key in self.cache:
            self._remove(self.cache[key])
        
        node = Node(key, value)
        self._add(node)
        self.cache[key] = node
        
        if len(self.cache) > self.capacity:
            # Remove LRU (tail.prev)
            lru = self.tail.prev
            self._remove(lru)
            del self.cache[lru.key]
    
    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    
    def _add(self, node):
        # Add to front (after head)
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None
```

**Visual:**
```
Initial: [1] ← [2] ← [3]  (capacity = 3)
Access 1: [1] ← [2] ← [3]  → [2] ← [3] ← [1]
Add 4:    [2] ← [3] ← [1] ← [4]  (evict 2)
         [3] ← [1] ← [4]
```

**Thread-Safe LRU:**
```python
import threading
from functools import wraps

class ThreadSafeLRUCache(LRUCache):
    def __init__(self, capacity):
        super().__init__(capacity)
        self.lock = threading.Lock()
    
    def get(self, key):
        with self.lock:
            return super().get(key)
    
    def put(self, key, value):
        with self.lock:
            return super().put(key, value)
```

**Python Built-in:**
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_function(n):
    # Computed result is cached
    return complex_computation(n)

# Automatic LRU caching!
```

### Cache Stampede

**Definition**: When many requests for the same expired cache key arrive simultaneously, all hit the database, causing overload.

**Scenario:**
```
Time 0: Cache entry expires
Time 1: 1000 requests arrive
        All find cache empty
        All query database simultaneously
        Database overloaded!
```

**Mitigation Strategies:**

#### 1. Cache Warming
```python
def warm_cache():
    """Preload cache before it expires"""
    popular_items = get_popular_items()
    for item in popular_items:
        cache.set(f"item:{item.id}", item, ttl=3600)

# Run periodically
schedule.every(30).minutes.do(warm_cache)
```

#### 2. Cache Locking
```python
def get_with_lock(key, loader_func):
    # Try cache
    value = cache.get(key)
    if value:
        return value
    
    # Try to acquire lock
    lock_key = f"lock:{key}"
    if cache.add(lock_key, "locked", ttl=10):  # Only one succeeds
        try:
            # Load from database
            value = loader_func()
            cache.set(key, value, ttl=3600)
            return value
        finally:
            cache.delete(lock_key)
    else:
        # Another thread is loading, wait and retry
        time.sleep(0.1)
        return get_with_lock(key, loader_func)
```

#### 3. Probabilistic Early Expiration
```python
import random
import time

def get_with_early_refresh(key, loader_func, ttl):
    value, timestamp = cache.get(key)
    
    if value is None:
        # Cache miss
        value = loader_func()
        cache.set(key, (value, time.time()), ttl)
        return value
    
    # Check if we should refresh early
    time_remaining = ttl - (time.time() - timestamp)
    beta = 1.0  # Tuning parameter
    
    if time_remaining < -beta * math.log(random.random()):
        # Refresh in background
        threading.Thread(target=refresh_cache, args=(key, loader_func, ttl)).start()
    
    return value
```

#### 4. Stale-While-Revalidate
```python
def get_stale_while_revalidate(key, loader_func, ttl):
    value, is_stale = cache.get_with_stale(key)
    
    if value is None:
        # Cache miss - load synchronously
        value = loader_func()
        cache.set(key, value, ttl)
        return value
    
    if is_stale:
        # Return stale value immediately
        # Refresh in background
        threading.Thread(target=refresh_cache, args=(key, loader_func, ttl)).start()
    
    return value
```

---

## Global Variables

### Where Are Global Variables Stored?

**Memory Layout:**
```
+------------------+
| Stack            |
+------------------+
| ↓ grows down     |
|                  |
| ↑ grows up       |
+------------------+
| Heap             |
+------------------+
| BSS (uninit)     | ← Uninitialized globals
+------------------+
| Data (init)      | ← Initialized globals
+------------------+
| Text (code)      |
+------------------+
```

**Sections:**

**Data Section** (Initialized Global Variables):
```c
int global_var = 42;        // Data section
const char* message = "Hi"; // Data section
```

**BSS Section** (Uninitialized Global Variables):
```c
int uninitialized_global;   // BSS section (auto-initialized to 0)
static int file_scope_var;  // BSS section
```

**Text Section** (Constants):
```c
const int constant = 100;   // Text section (read-only)
```

**Example:**
```c
#include <stdio.h>

int initialized_global = 10;      // Data section
int uninitialized_global;          // BSS section
static int static_global = 20;    // Data section

void function() {
    static int static_local = 30;  // Data section (not stack!)
    int local_var = 40;            // Stack
}

int main() {
    int* heap_var = malloc(sizeof(int));  // Heap
    *heap_var = 50;
    
    printf("Initialized global: %p\n", &initialized_global);
    printf("Uninitialized global: %p\n", &uninitialized_global);
    printf("Static global: %p\n", &static_global);
    printf("Static local: %p\n", &static_local);
    printf("Local var: %p\n", &local_var);
    printf("Heap var: %p\n", heap_var);
    
    free(heap_var);
    return 0;
}
```

**Output (example):**
```
Initialized global: 0x601040     (Data section)
Uninitialized global: 0x601060   (BSS section)
Static global: 0x601044          (Data section)
Static local: 0x601048           (Data section)
Local var: 0x7fffcb5e4a3c        (Stack - high address)
Heap var: 0x1e47010              (Heap)
```

---

## Algorithms

### Quicksort vs Merge Sort

#### Quicksort

**Algorithm:**
1. Choose pivot element
2. Partition array (elements < pivot on left, > pivot on right)
3. Recursively sort left and right partitions

**Implementation:**
```python
def quicksort(arr):
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quicksort(left) + middle + quicksort(right)
```

**Properties:**
- Time Complexity:
  - Best: O(n log n)
  - Average: O(n log n)
  - Worst: O(n²) (already sorted)
- Space: O(log n) (recursion stack)
- In-place: Yes (can be)
- Stable: No

#### Merge Sort

**Algorithm:**
1. Divide array in half
2. Recursively sort each half
3. Merge sorted halves

**Implementation:**
```python
def mergesort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = mergesort(arr[:mid])
    right = mergesort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

**Properties:**
- Time Complexity: O(n log n) (all cases)
- Space: O(n) (temporary arrays)
- In-place: No
- Stable: Yes

#### Comparison

| Aspect | Quicksort | Merge Sort |
|--------|-----------|------------|
| **Average Time** | O(n log n) | O(n log n) |
| **Worst Time** | O(n²) | O(n log n) |
| **Space** | O(log n) | O(n) |
| **Stability** | No | Yes |
| **In-place** | Yes | No |
| **Cache Performance** | Good | Poor |

#### Which is Faster?

**In Practice: Quicksort is usually faster**

**Reasons:**
1. **Better Cache Locality**: Access contiguous memory
2. **Lower Constant Factors**: Simpler operations
3. **In-place**: No extra memory allocation
4. **Fewer Comparisons** (on average)

**Benchmark:**
```
Array size: 1,000,000 random integers
Quicksort: 0.15 seconds
Merge Sort: 0.23 seconds
```

**When Merge Sort is Better:**
- Need guaranteed O(n log n) (worst-case matters)
- Stability required
- Linked lists (no random access)
- External sorting (data doesn't fit in memory)

#### Real-World Usage

**Programming Languages:**

**Python (Timsort)**:
- Hybrid of merge sort and insertion sort
- Stable, O(n log n) worst-case
```python
arr.sort()  # Uses Timsort
```

**C++ (Introsort)**:
- Hybrid of quicksort, heapsort, insertion sort
- Starts with quicksort
- Switches to heapsort if recursion too deep
- Uses insertion sort for small subarrays
```cpp
std::sort(arr.begin(), arr.end());  // Uses Introsort
```

**Java**:
- Arrays of primitives: Dual-Pivot Quicksort
- Arrays of objects: Timsort
```java
Arrays.sort(primitiveArray);  // Quicksort variant
Arrays.sort(objectArray);     // Timsort
```

**C (qsort)**:
- Typically quicksort with randomized pivot
```c
qsort(arr, n, sizeof(int), compare);
```

**Optimizations:**

**Hybrid Approaches:**
```python
def optimized_quicksort(arr, threshold=10):
    if len(arr) <= threshold:
        # Use insertion sort for small arrays
        return insertion_sort(arr)
    
    # Use quicksort for larger arrays
    pivot = median_of_three(arr)  # Better pivot selection
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return optimized_quicksort(left) + middle + optimized_quicksort(right)
```

**Three-Way Partitioning** (handle duplicates efficiently):
```python
def three_way_quicksort(arr, low, high):
    if low >= high:
        return
    
    lt = low
    gt = high
    pivot = arr[low]
    i = low
    
    while i <= gt:
        if arr[i] < pivot:
            arr[lt], arr[i] = arr[i], arr[lt]
            lt += 1
            i += 1
        elif arr[i] > pivot:
            arr[i], arr[gt] = arr[gt], arr[i]
            gt -= 1
        else:
            i += 1
    
    three_way_quicksort(arr, low, lt - 1)
    three_way_quicksort(arr, gt + 1, high)
```

---

## Summary

This guide covers fundamental operating system concepts essential for understanding how modern computers work:

- **Processes and Threads**: The building blocks of concurrent execution
- **Memory Management**: Virtual memory, paging, and efficient resource allocation
- **Synchronization**: Preventing race conditions and deadlocks in concurrent programs
- **Inter-Process Communication**: How processes share data safely
- **System Architecture**: File descriptors, system calls, and the Linux philosophy
- **Caching**: Improving performance through intelligent data storage strategies

Understanding these concepts is crucial for:
- Writing efficient, concurrent programs
- Debugging complex multi-threaded applications
- Optimizing system performance
- Designing scalable software architectures
- Working with operating systems and kernels

---

**Note**: Some sections marked "To be defined" in the original document indicate areas for future expansion. This README provides a comprehensive foundation in operating system concepts with practical examples and real-world applications.
