# Memory Hierarchies and Matrix Multiplication

## Start with single processors

•Most applications run at < 10% of the “peak” performance 

•Much of the performance is lost on a single processor, moving data

Parallelizing slow serial code only produces slow parallel code.

Linear speedup is not necessarily fast code

Matrix multiply is extreme, but 10x from sequential optimizations is common.

## Costs in modern processors

–Idealized models and actual costs

### Idealized Uniprocessor Model

•**Processor names** **variables**

–Integers, floats, doubles, pointers, arrays, structures, etc

•**Processor performs** **operations** **on those variables:**

–Arithmetic, logical operations, etc. 

–Load/Store variables between memory and registers

•**Processor** **controls** **the order, as specified by program**

–Branches (if), loops, function calls, etc.

–*Compiler translates into* *“**obvious**”* *lower level instructions*

–*Hardware executes instructions in order specified by compiler*

–*Read returns the most recently written data*

•**Idealized Cost**

–Each operation (+,*, etc.) has roughly the same cost

–And reading/writing variables is “free”

### Compilers and assembly code

•Compilers for languages like C/C++ and Fortran:

–Check that the program is legal

–Translate into assembly code

–Optimizes the generated code

![image-20230505203314322](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505203314322.png)

### Compilers Manage Memory and Registers

•Compiler performs “register allocation” to decide when to load/store vs reuse

![image-20230505203443961](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505203443961.png)

### Compiler Optimizes Code

Besides register allocation, the compiler performs optimizations

## Memory hierarchies

### More Realistic Uniprocessor Model

•**Memory accesses (load / store) have two costs**

–**Latency:** cost to load or store 1 word (α)

–**Bandwidth:** Average rate (bytes/sec) to load/store a large chunk or inverse time/byte, β

![image-20230505204609088](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505204609088.png)

### Latency vs Bandwidth

•Memory systems and networks

![image-20230505204636745](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505204636745.png)

Latency and Bandwidth on Intel Haswell

Latency (α) 10s-100 ns

Bandwidth 2-40 GB/s

  (β) .025-.5 ns/B 

![image-20230505204801306](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505204801306.png)

Memory Bandwidth Gap

![image-20230505204831909](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505204831909.png)

Memory Latency Gap is Worse 

![image-20230505204845075](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505204845075.png)

•**Most programs have a high degree of locality**

–**spatial locality:** accessing things nearby previous accesses

–**temporal locality:** reusing an item that was previously accessed

![image-20230505205230365](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505205230365.png)

### Cache Basics

•**Cache** **is fast (expensive) memory which keeps copy of data; it is hidden from software**

–**Simplest example: data at memory address xxxxxxx10 is stored at cache location 10**

•**Cache hit:** in-cache memory access—cheap

•**Cache miss:** non-cached memory access—expensive

–**Need to access next, slower level of memory**

![image-20230505205520262](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505205520262.png)

•**Cache line length: # of bytes loaded together in one entry**

-Ex: If either xxxxx1100 or xxxxx1101 is loaded, both are

![image-20230505205704646](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505205704646.png)

•**Associativity**

-direct-mapped: only 1 address (line) in a given range in cache

•Data at address xxxxx1101 stored at cache location 1101

•(Only 1 such value from memory)

### Why Have Multiple Levels of Cache?

•**On-chip vs. off-chip**

–On-chip caches are faster, but smaller

•**A large cache has delays**

–Hardware to check longer addresses in cache takes more time

–Associativity, which gives a more general set of data in cache, also takes more time

•**Some examples:**

–Cray T3E eliminated one cache to speed up misses

–Intel Haswell (Cori Ph1) uses a Level 4 cache as “victim cache”

•**There are other levels of the memory hierarchy**

–Register, pages (TLB, virtual memory), …

–And it isn’t always a hierarchy

### Approaches to Handling Memory Latency

![image-20230505210340606](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505210340606.png)

![image-20230505210758145](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505210758145.png)

More realistic example

![image-20230505210814458](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505210814458.png)

![image-20230505214206999](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505214206999.png)

![image-20230505214218180](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505214218180.png)

![image-20230505214242008](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505214242008.png)

Memory Benchmark (CacheBench)

![image-20230505225755290](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505225755290.png)

![image-20230505225823834](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505225823834.png)

## Parallelism within single processors

### What is Pipelining? 

![image-20230505230000242](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505230000242.png)

•Pipelining doesn’t change latency (90 min)

•Bandwidth limited by slowest pipeline stage

•  Peak bandwidth is 1/slowest

•Speedup <=  # of stages

Laundry Bandwidth

![image-20230506110346999](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506110346999.png)

## SIMD: Single Instruction, Multiple Data

![image-20230506110549336](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506110549336.png)

vector operation

•Need to:

•Expose parallelism to compiler (or write manually)

•Be contiguous in memory and cache aligned (in most cases)

#### Data Dependencies Limit Parallelism

**Parallelism can get the wrong answer if instructions execute out of order** 

**Types of dependencies**

•RAW: Read-After-Write

  X = A ; B = X;

•WAR: Write-After-Read

  A = X ; X = B;

•WAW: Write-After-Write

  X = A ; X = B;

•No problem / dependence for RAR: Read-After-Read

#### Control Dependencies Limit Parallelism

Conditionals, function calls, breaks, and dynamically changing loop bounds may also prevent vectorization

```c++
for (i = 0, i<n, i++) {
  if (a[i] == 14) n = 10;
}
```

#### Memory Alignment and Strides

![image-20230506113114049](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506113114049.png)

#### FMA: Fused Multiply Add

![image-20230506113243250](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506113243250.png)

•In theory, the compiler understands all of this

–It will rearrange instructions to maximizes parallelism, uses FMAs and SIMD

–While preserving dependencies

### Compilers Optimizations

•Unrolls loops (because control isn’t free)

•Fuses loops (merge two together)

•Interchanges loops (reorder)

•Eliminates dead code (the branch never taken)

•Reorders instructions to improve register reuse and more

•Strength reduction (multiply by 2, into shift left) 

•Strip-mine: turn one loop into nested one

Strip-mine + interchange = tiling

https://dl.acm.org/doi/pdf/10.1145/197405.197406

### Take-Aways

**Even “serial” processors use parallelism**

•Instruction level parallelism (ILP): need to consider instruction mix

•SIMD instructions

•Special instructions like fused multiply add (FMA)

**Compilers help with this, but the programmer can make their job easy or hard**

•Dependencies are a big source of problems for the compiler

•They need to work conservatively

## Case study: Matrix **Multiplication**

### Why Matrix Multiplication

•An important kernel in many problems

–Dense linear algebra is a motif in every list

–Closely related to other algorithms, e.g., transitive closure on a graph

–**And dominates training time in deep learning (CNNs)**3 × 256 × 256

•Optimization ideas can be used in other problems

•The best case for optimization payoffs

•The most-studied algorithm in high performance computing

### A Simple Model of Memory

•Assume just 2 levels in the hierarchy, fast and slow

•All data initially in slow memory

–$m$ = number of memory elements (words) moved between fast and slow memory 

–$t_m$ = time per slow memory operation (inverse bandwidth in best case)

–$f $= number of arithmetic operations

–$t_f$ = time per arithmetic operation << $t_m$

–$CI = f / m$ average number of flops per slow memory access

•Minimum possible time = $f * t_f$ when all data in fast memory

•Actual time 

–$f * t_f + m * t_m = f * t_f * (1 + t_m/t-f * 1/CI) $

•Larger $CI$ means time closer to minimum$ f * tf$ 

![image-20230506120100811](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506120100811.png)

### Validating the Model

•How well does the model predict actual performance? 

–Actual DGEMV: Most highly optimized code for the platform

•Model sufficient to compare across machines

•But under-predicting on most recent ones due to latency estimate

![image-20230506164928998](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506164928998.png)

### Simplifying Assumptions

•What simplifying assumptions did we make in this analysis?

–Constant “peak” computation rate

–Fast memory was large enough to hold three vectors

–The cost of a fast memory access is 0

•OK for registers

•Not for cache (even L1)

–Memory latency is constant

•Could simplify further by ignoring memory operations in X and Y

–Not bad: time to read matrix (bandwidth) may dominate

![image-20230506165853908](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506165853908.png)

![image-20230506165916826](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506165916826.png)

![image-20230506165934530](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506165934530.png)

![image-20230506170143470](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506170143470.png)

![image-20230506170158869](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506170158869.png)

![image-20230506170226856](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506170226856.png)

![image-20230506170257093](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506170257093.png)

![image-20230506170316235](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506170316235.png)

![image-20230506170337971](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506170337971.png)

## ATLAS from UTK Became Standard

•ATLAS is faster than all other portable BLAS implementations and it is comparable with machine-specific libraries provided by the vendor.

![image-20230506172147722](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506172147722.png)

## Take-Aways

![image-20230506172342144](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230506172342144.png)



## Summary

•Performance programming on uniprocessors requires

–understanding of memory system

–understanding of fine-grained parallelism in processor

•Simple performance models can aid in understanding

–Two ratios are key to efficiency (relative to peak)

1.computational intensity of the algorithm: 

•q = f/m = # floating point operations / # slow memory references

2.machine balance in the memory system: 

•tm/tf = time for slow memory reference / time for floating point operation

•Want q > tm/tf  to get half machine peak

•Blocking (tiling) is a basic approach to increase q

–Techniques apply generally, but the details (e.g., block size) are architecture dependent

–Similar techniques are possible on other data structures and algorithms

## Q&A

1.What is the key to understand algorithm efficiency in our simple memory model? 



2.What is the key to understand machine efficiency in our simple memory model? 



3.What is tiling? 



4.Why does block matrix multiply reduce the number of memory references? 



5.What are the BLAS? 



6.Why does loop unrolling improve uniprocessor performance? 