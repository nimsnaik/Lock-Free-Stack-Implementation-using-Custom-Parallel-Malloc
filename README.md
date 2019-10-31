## Project Proposal

### Summary

Lock free linked stacks offer some amount of speedup over locked linked stacks, however the standard library implementation of malloc includes locks and would thus implicitly add locks to the code. We would like to implement a parallel implementation of malloc that has fewer locks and is optimized for the given problem.

### Background

Lock free data structures are a much-researched topic and have been implemented by various methods. We would like to implement a “A Scalable Lock-free Stack Algorithm” [1]. This algorithm claims to continue to work well on expanding stack sizes using a modification of the elimination tree method [2] of lock free stack implementation. \
However, as soon as the question of dynamic memory allocation arises, malloc locks need to be considered as well.  While the glibc malloc is thread-safe, it is not optimized for a multi-threaded environment [3]. While there are implementations such as jemalloc [4] which are optimized for multi-threaded environments, we would like to implement a parallel malloc optimized for linked list allocation.

### Challenges

The first challenge would be to implement a correct lock free stack that outperforms a standard locked stack implementation. We would have to follow the ideas presented in the paper to implement the lock free data structure and design tests that use the lock free data structure in order to verify correctness and measure speed. We also plan to explore some more lock free implementations of stacks to see if this one really is the best for our current problem. \
The second challenge is the malloc implementation. We plan to start with our baseline (213) implementation of malloc and modify it to be thread safe. We will then need to find the most optimal way of implementing malloc for multi-threaded linked lists. While we want our malloc to work the best at the linked list problem, we also want to ensure correctness and baseline performance for all use cases. We also need to optimize memory allocation based on access patterns, evaluating how the changing allocation patterns impact the various use cases of the stack. \

### Goals and Deliverables:

In our poster session we want to explain the design choices we made while creating the parallel malloc. We would also like to show graphs of the change in performance we observed due to each of these design choices and comparison of our malloc with 1. The thread safe baseline implementation of malloc and 2. The glibc malloc. \
Our first main goal is to achieve correctness in both the malloc and lock free stack implementation and create a comprehensive test suite to check correctness for both. Both the stack and malloc should individually perform better than the baseline versions.
The second goal is to optimize malloc to perform extremely well for this specific application and create many tests with various access patterns. \
_Plan to Achieve_ \
•	A lock free linked list implementation that is correct and uses our implementation of malloc \ 
•	Achieve significant speedup while moving from locked to lock free stacks and from glibc malloc to our implementation of malloc \
•	An idea of how malloc implementations will affect memory access patterns within the program \
_Hope to achieve_ \
•	Comparable performance with existing implementations of malloc optimized for multi-threaded environments \
•	A lock free malloc as well (if possible) \
•	Trying out our malloc on other locked or lock free data structures \

### Platform Choice

We will be using the GHC machines and a either OpenMP or pthread library to generate our shared memory, multi-threaded environment.

### Schedule

|WEEK| GOALS                                                                                                  |
|----|--------------------------------------------------------------------------------------------------------|
| 1	 | Modify existing malloc to be thread safe and test, understand lock free linked stacks                  |
| 2	 | Implement and test lock free stacks                                                                    |
| 3	 | Implement a generic parallel malloc and multiple usecases of the stacks with varying access patterns   |  
| 4	 | Modify malloc to perform best case for all the multiple use cases                                      |
| 5  | Continue to modify malloc, while benchmarking performance using various metrics                        |                      

### References 

[1] https://people.csail.mit.edu/shanir/publications/Lock_Free.pdf \
[2] https://groups.csail.mit.edu/tds/papers/Shavit/ST-elimination.pdf \
[3] https://sourceware.org/glibc/wiki/MallocInternals \
[4] http://jemalloc.net/ \
