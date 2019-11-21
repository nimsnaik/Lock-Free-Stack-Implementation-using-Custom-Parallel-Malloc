## Project Proposal

### Summary

Lock free stacks offer some speedup over fine grained locked stacks. One part of our project is to implement and observe this speedup. However the standard library implementation of malloc includes locks and would thus implicitly add locks to the code. We would like to implement a parallel implementation of malloc that has fewer locks and is optimized for the given problem.

### Background

Lock free data structures are a much-researched topic and have been implemented by various methods. We would like to implement a “A Scalable Lock-free Stack Algorithm” [1]. This algorithm claims to continue to work well on expanding stack sizes using a modification of the elimination tree method [2] of lock free stack implementation.<br/><br/>
However, as soon as the question of dynamic memory allocation arises, malloc locks need to be considered as well.  While the glibc malloc is thread-safe, it is not optimized for a multi-threaded environment [3]. While there are implementations such as jemalloc [4] which are optimized for multi-threaded environments, we would like to implement a parallel malloc optimized for linked list allocation.

### Challenges

The first challenge would be to implement a correct lock free stack that outperforms a standard locked stack implementation. We would have to follow the ideas presented in the paper to implement the lock free data structure and design tests that use the lock free data structure in order to verify correctness and measure speed. We also plan to explore some more lock free implementations of stacks to see if this one really is the best for our current problem. <br/><br/>
The second challenge is the malloc implementation. We plan to start with our baseline (213) implementation of malloc and modify it to be thread safe. We will then need to find the most optimal way of implementing malloc for multi-threaded linked lists. While we want our malloc to work the best at the linked list problem, we also want to ensure correctness and baseline performance for all use cases. We also need to optimize memory allocation based on access patterns, evaluating how the changing allocation patterns impact the various use cases of the stack.

### Goals and Deliverables:

In our poster session we want to explain the design choices we made while creating the parallel malloc. We would also like to show graphs of the change in performance we observed due to each of these design choices and comparison of our malloc with 1. The thread safe baseline implementation of malloc and 2. The glibc malloc. <br/><br/>
Our first main goal is to achieve correctness in both the malloc and lock free stack implementation and create a comprehensive test suite to check correctness for both. Both the stack and malloc should individually perform better than the baseline versions.
The second goal is to optimize malloc to perform extremely well for this specific application and create many tests with various access patterns. <br/><br/><br/>
_Plan to Achieve_ <br/>
- A lock free linked list implementation that is correct and uses our implementation of malloc 
- Achieve significant speedup while moving from locked to lock free stacks and from glibc malloc to our implementation of malloc
- An idea of how malloc implementations will affect memory access patterns within the program <br/>

_Hope to achieve_ <br/>
- Comparable performance with existing implementations of malloc optimized for multi-threaded environments
- A lock free malloc as well (if possible)
- Trying out our malloc on other locked or lock free data structures <br/>

### Checkpoint Progress

In line with the spirit of the theme of the class, we are proceeding with our project by implementing the lock free stack and parallel malloc in parallel. We plan on integrating both of them once both the implementations are sufficiently unit tested. 
Progress of both the implementations are reported separately in the following sections -
Lock Free Stacks
To explore lock free stacks using linked lists, it is first necessary to read the existing literature on the subject and determine what approach best suited our project, time-line and learning needs.
Literature Review
The first two papers reviewed was the diffracting tree and the elimination stack. While these are not optimal methods of creating a lock free linked list, understanding the logic behind these classic approaches was essential before implementing a lock free linked list. Diffracting trees make contention happen at various different points on a combining tree such that the final counter or value is incremented only by one thread whenever a write collision occurs, and the answer is then ‘diffracted’ out. The tree reduces contention between threads by ensuring that all the threads are not contending for a single shared variable. However, when this approach is used on a single threaded program, there is a very high overhead. In the sense it does not scale down well. [1] 



The elimination stack approach is slightly better in that there are multiple small trees dynamically created instead of a single large tree. While the overhead is lower than the diffracting tree, the algorithmic complexity is very high for not as much value-add in a simple stack implementation nor any significant speedup. [2]
  
The paper we are implementing uses elimination back-off array, where a thread first tries to put its value on the shared stack, if it faces contention, it then goes to a collision array where collisions are handled. Push/Pop collisions are handled by replacing the head with the new head that is to be pushed. If there is no collision or a “like” (push/push) collision then the thread, after waiting for a fixed period tries to add directly to the stack again. Unlike the funnel and tree approach, this method has no overhead when no contentions are present. [3]
 
Implementations
For our project, we decided to implement 3 stack approaches using linked lists and measure the relative performance of all of them. The first approach is a simple locked linked list using mutexes. We considered a fine grained locked linked list; however this approach did not make as much sense in a stack where only the top value is usually accessed. The baseline locked implementation is complete and correct.
The second approach is the one with compare and swap that is discussed briefly in the PCA lecture slides. This approach has two major things which have to be implemented which is the ABA issue solution and keeping track of hazard pointers held by other slides. The implementation with ABA resolution is completed and correct.
The third implementation is with the elimination back off array referenced above. While the paper has some pseudo code to explain the main ideas, we implement this algorithm ourselves. The implementation for this is going on. While the implementation works in most cases, if no thread collides with a given thread and it is unable to swap itself onto the stack, sometimes it is stuck in an almost infinite loop, and it segfaults at certain collisions which we are yet to debug.
Preliminary results (Lock-free vs Single locked)
We conducted a preliminary timing analysis of the first two implementations (using standard malloc and free) :




The results are calculated using 1,4,8 and 16 threads with 1000000 psuedo-random stack operations each. The time is only measured after all threads complete and all remaining items are popped from the stack by the main thread for both implementations.
While the lock free version is faster than the locked version, we hope to observe a more linear speedup with the new algorithm.
Challenges
The compliler on the gcc machine is gcc 4.8 (before they completely implemented the atomic operations) hence some operations like atomic_load aren’t there (or we haven’t found them). If the problem persists we might consider moving to a different 4 (and run 8 threads) core machine as we care more about the relative speedup between implementations than the absolute time taken to execute the program.
While the algorithm is explained in the paper, implementing the algorithm is challenging as there are some portions which are not explained/left out of the code. A significant amount of time was spent working code from the paper. The implementation is not complete, but should be by the end of the week.
Testing the program for correctness was also non-trivial as there are multiple places for collisions to occur and a sufficiently large test had to be conducted to ensure that errors are found. We also manually stepped through the code to ensure that collisions at any point in the code are handled correctly.
The above times were calculated using clock() linux function which is less than ideal. We plan to get more exact cycle counts using tools such as perf for the final evaluation. Additionally, the elimination array contains certain tunable parameters such as the size of the collision array, the location in the array where the thread attempts to collide which have to be tuned to optimize performance.
Immediate Goals
The hazard pointer reference has to be handled both the lock free and elimination array implementation. The ABA problem also needs to be fixed in the elimination array implementation.
The elimination array implementation needs to be fixed and pass all correctness checks.

[1] http://groups.csail.mit.edu/tds/papers/Shavit/SZ-diffracting.pdf
[2] https://groups.csail.mit.edu/tds/papers/Shavit/ST-elimination.pdf
[3] https://people.csail.mit.edu/shanir/publications/Lock_Free.pdf

Parallel Malloc
Baseline implementation
Our initial few days was spent on setting up a baseline that we can use to measure speed up for our implementation of parallel malloc. We decided to re-use our 15-213 malloc implementation and make it thread safe. The current malloc implementation was modified to acquire and release a single global lock upon every malloc and free request. To test the working, we decided to also reuse the trace files that the lab uses. The current mdriver implementation of the lab runs each trace file one after the other, restoring the heap to a clean state every time before starting a new test.
We modified the mdriver code to run all the traces in parallel. We used pthreads to spawn as many threads as there are traces and assigned to one trace to each thread. The heap will be initialized to a clean state once and will continue to receive requests until all threads have finished running their assigned trace file. While doing so, we retained the correctness checks, utilization and throughput calculations carried out by the mdriver code. Each thread runs each of these checks with the trace that it is assigned to.
Adapting the mdriver code structure, especially the timing measurement logic to work with this design did take up a few days but we were finally able to get it up and running.
Below is the result of our serial malloc implementation running on a shark machine –
 














Below is the performance of the thread safe malloc implemented through a pthread_mutex_lock on the same machine –
 

We will be using the above thread safe malloc as our baseline for speed up measurements.
Literature Survey of current multi-threaded mallocs available 
Our immediate goal in the next few days to adapt our current version of serial malloc into a version that performs better for multi-threaded programs.
We came across a bunch of malloc implementations written for parallel multi-threaded programs. Across these implementations, the common issues that they try to address are listed below –
1)	Speed – malloc() and free() in multi-threaded version should perform as well as state of the art serial memory allocator
2)	Minimize Fragmentation 
3)	Scalability – the performance of the allocator must scale well as the number of processors/threads increase
4)	Avoid False sharing – The allocator must not try and avoid sharing data between different threads that is on the same cache line
While (1) and (2) are pretty much carried over from the demands that even a serial memory allocator is expected to meet, (3) and (4) are unique challenges that get introduced as we try to cater to the allocation requests from a multi-threaded environment. 
The most popular search hits for a parallel malloc on the web currently seem to be Intel’s tbbmalloc, jemalloc, tcmalloc, hoard and nedmalloc. A common theme across these implementations is to make a best attempt at servicing the request without locks failing which a central memory allocator is used. Based on the availability of details of the implementation and our estimate of the time needed to implement these, tcmalloc and hoard seem most suitable for the purposes of this project. Our aim is to implement one of these at the minimum and both if time permits. Our immediate aim though will be adapt our malloc implementation to mimic TC Malloc whose implementation snippets are described below - 

TC Malloc 
The TC Malloc  assigns each thread its own thread local cache. Objects are moved from the central heap into the thread local cache and periodic garbage collection is used to move the memory back.
                              

TC Malloc manages the heap as a span – which is a sequence of pages. A span is used to allocate memory to the central heap and the thread local caches.
Allocation – 
TC Malloc distinguishes requests based on the size. Sizes below a certain threshold will be serviced by the Thread Cache while those greater will be serviced by the central heap. The span that is used to service the request will be marked as belonging to a larger object(serviced from the central heap) or a smaller object (serviced from the thread local cache).
Both the thread cache and central heap will maintain a free list of objects classified by the size. Based on the size of the request, the corresponding list will be used to service the request.
Deallocation – 
The span is used as a marker to figure out the list that the freed address belongs to. Based on the list returned, we will jump to either the thread cache list
Garbage Collection –
A thread cache is garbage collected if its object size exceeds a certain limit. Our aim would be experiment with this threshold. 
Immediate goals
1)	Implement TC Malloc – week of 18 – Nov – implementation for 5 days followed by testing during the weekend
2)	Measure speed up of 15-213 traces with the TC Malloc implementation compared to the baseline – Week of 25 – Nov. Proceed with integration if things look ok

Code base
https://github.com/shatur93/15618_project_parallel_malloc (currently set to private to comply with mdriver.c copyrights and 213 academic integrity policy)

Eventual goals for the whole project -
1)	Measure performance of lock free stack/linked list with thread safe malloc
2)	Measure performance of lock free stack/linked list with TC Malloc
3)	Analyze and perform optimizations for lock free stack/linked list with TC Malloc

### Platform Choice

We will be using the GHC machines and either OpenMP or pthread library to generate our shared memory, multi-threaded environment.

### Schedule

|WEEK| GOALS                                                                                                  |
|----|--------------------------------------------------------------------------------------------------------|
| 1	 | Modify existing malloc to be thread safe and test, understand lock free linked stacks                  |
| 2	 | Implement and test lock free stacks                                                                    |
| 3	 | Implement a generic parallel malloc and multiple usecases of the stacks with varying access patterns   |  
| 4	 | Modify malloc to perform best case for all the multiple use cases                                      |
| 5  | Continue to modify malloc, while benchmarking performance using various metrics                        |                      

### References 

[1] https://people.csail.mit.edu/shanir/publications/Lock_Free.pdf <br/>
[2] https://groups.csail.mit.edu/tds/papers/Shavit/ST-elimination.pdf <br/>
[3] https://sourceware.org/glibc/wiki/MallocInternals <br/>
[4] http://jemalloc.net/ <br/>
