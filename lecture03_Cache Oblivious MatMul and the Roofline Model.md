# Cache Oblivious MatMul and the Roofline Model

## Recursive Matrix Multiplication

![image-20230507104004660](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507104004660.png)

![image-20230507104016073](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507104016073.png)

![image-20230507104534122](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507104534122.png)

Arith(n) = # arithmetic operations in RMM( . , . , n)

​       = $8 · Arith(n/2) + 4(n/2)^2$ if n > 1,  else 1

​       = $2n^3$  … same operations as usual, in different order

​       = $O(n^3)$ this is our f = # flops

![image-20230507105402059](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507105402059.png)

Alternate Data Layouts

![image-20230507120359800](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507120359800.png)

## Theory: Communication lower bounds

![image-20230507211539960](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507211539960.png)

![image-20230507211600111](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507211600111.png)

## **Since** **matmul** **is flop-limited, can we do better than $O(n^3)$?**

### Strassen’s Matrix Multiply

![image-20230507211758282](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507211758282.png)

![image-20230507211833865](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507211833865.png)

![image-20230507211853507](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507211853507.png)

## Basic Linear Algebra Subroutines (BLAS)

![image-20230507212116687](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212116687.png)

![image-20230507212107498](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212107498.png)

### Dense Linear Algebra: BLAS2 vs. BLAS3

•Different computational intensity, different performance

![image-20230507212140995](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212140995.png)

![image-20230507212146369](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212146369.png)

## Some reading on MatMul

•Sourcebook on Parallel Computing Chapter 3. 

•Web pages for reference:

–[BeBOP Homepage](http://bebop.cs.berkeley.edu/) 

–[ATLAS Homepage](http://www.netlib.org/atlas/index.html) 

–[BLAS](http://www.netlib.org/blas/) (Basic Linear Algebra Subroutines), Reference for (unoptimized) implementations of the BLAS, with documentation. 

–[LAPACK](http://www.netlib.org/lapack/) (Linear Algebra PACKage), a standard linear algebra library optimized to use the BLAS effectively on uniprocessors and shared memory machines (software, documentation and reports) 

–[ScaLAPACK](http://www.netlib.org/scalapack/) (Scalable LAPACK), a parallel version of LAPACK for distributed memory machines (software, documentation and reports) 

•"[Performance Optimization of Numerically Intensive Codes](http://www.siam.org/catalog/mcc12/se12.htm)", by Stefan Goedecker and Adolfy Hoisie, SIAM 2001. 

•“Tuning Strassen's Matrix Multiplication for Memory Efficiency,” Mithuna S. Thottethodi, Siddhartha Chatterjee, and Alvin R. Lebeck in Proceedings of Supercomputing '98, November 1998 [postscript](http://www.cs.duke.edu/ari/arch/papers/sc98.ps)

•“Recursive Array Layouts and Fast Parallel Matrix Multiplication” by Chatterjee et al. IEEE TPDS November 2002.

Many related papers at bebop.cs.berkeley.edu

## Take-Aways

## Roofline Model

How fast can an algorithm go in practice?

### What is a Performance Model?

A formula to estimate performance

![image-20230507212417016](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212417016.png)

![image-20230507212446915](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212446915.png)

Examples we’ve seen for time

### Why Use a Performance Model?

Understand performance behavior

§Differences between Architectures, Programming Models, implementations, etc. 

![image-20230507212534110](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212534110.png)

§Identify performance bottlenecks

![image-20230507212554198](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212554198.png)

![image-20230507212614391](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212614391.png)

§**Determine when we’re done optimizing**

### Serial and Shared Memory Machines

![image-20230507212649284](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212649284.png)

### History of the Roofline Model

![image-20230507212738097](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212738097.png)

![image-20230507212759116](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212759116.png)

### What’s in the Roofline Model?

![image-20230507212825653](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212825653.png)

![image-20230507212919154](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507212919154.png)

![image-20230507213011692](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213011692.png)

### How good is flop/s as a model?

#### DGEMV

![image-20230507213201414](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213201414.png)

#### Data Movement Complexity

![image-20230507213437277](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213437277.png)

#### Machine Balance and Computational Intensity

![image-20230507213650936](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213650936.png)

![image-20230507213738931](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213738931.png)

#### Computational Intensity

![image-20230507213808505](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213808505.png)

### (DRAM) Roofline

![image-20230507213944959](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507213944959.png)

![image-20230507214013287](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214013287.png)

### The Roofline Performance Model

![image-20230507214209261](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214209261.png)

![image-20230507214231272](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214231272.png)

![image-20230507214241822](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214241822.png)

### Roofline Example 

![image-20230507214311694](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214311694.png)

![image-20230507214336514](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214336514.png)

![image-20230507214348719](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214348719.png)

### Roofline Across Algorithms

![image-20230507214406909](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214406909.png)

### Takeaways

![image-20230507214421536](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230507214421536.png)
