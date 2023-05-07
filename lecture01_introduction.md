# Application of Parallel Computers

最近看到一些文章中有关于模型的计算力消耗问题，也就是flops。论文中通常会比较在差不多的flops上两个模型的差距。比如说DenseNet 中就放出了一张在flops差不多的情况下，其与Resnet的对比图来说明DenseNet所需计算力小而正确率高的优势。那么，flops到底是什么？怎么体现模型所需的计算力消耗的？又是怎么计算的呢？本篇文章总结了以上三个问题，同时给出了计算flops的开源库。

## **一、什么是flops**

对flops有疑惑，首先得先捋清这个概念：

- **FLOPS**：注意全大写，是floating point operations per second的缩写，意指每秒浮点运算次数，理解为计算速度。是一个衡量硬件性能的指标。
- **FLOPs**：注意s小写，是floating point operations的缩写（s表复数），意指浮点运算数，理解为计算量。可以用来衡量算法/模型的复杂度。

网上打字很容易全小写，造成混淆，本问题针对模型，应指的是FLOPs。

我们知道，通常我们去**评价一个模型时，首先看的应该是它的精确度**，当你精确度不行的时候，你和别人说我的模型预测的多么多么的快，部署的时候占的内存多么多么的小，都是白搭。但当你模型达到一定的精确度之后，就需要更**进一步的评价指标来评价你模型**：1）**前向传播时所需的计算力**，它反应了对硬件如GPU性能要求的高低；2）**参数个数**，它反应所占内存大小。为什么要加上这两个指标呢？因为这事关你模型算法的落地。比如你要在手机和汽车上部署深度学习模型，对模型大小和计算力就有严格要求。模型参数想必大家都知道是什么怎么算了，而前向传播时所需的计算力可能还会带有一点点疑问。所以这里总计一下前向传播时所需的计算力。它正是由**FLOPs**体现，那么**FLOPs**该怎么计算呢？

## **二、如何计算flops**

我们知道，在一个模型进行前向传播的时候，会进行卷积、池化、BatchNorm、Relu、Upsample等操作。这些操作的进行都会有其对应的计算力消耗产生，其中，卷积所对应的计算力消耗是所占比重最高的。所以，我们这里主要讲一下卷积操作所对应的计算力。

我们以下图为例进行讲解：

> 作者：冷月@知乎



![img](https://pic1.zhimg.com/80/v2-e0f030eb62bc80e33a2af8ad3d0cc234_1440w.webp)

先说结论：卷积层 计算力消耗 等于上图中两个立方体 (绿色和橙色) 体积的乘积。即flops =

**推导过程：**卷积层 wx + b 需要计算两部分，首先考虑前半部分 wx 的计算量：

令 :

- k 表示卷积核大小;
- c 表示输入 feature map 的数量;

则对于输出 feature map 上的**单个** Unit 有：

**`k \* k \* c 次乘法，以及 k \* k \* c - 1 次加法`**



![img](https://pic2.zhimg.com/80/v2-cdca4af4354d45ea483a3181d635ae2d_1440w.webp)

用上图形象化解释就是：

Image大小为 5x5，卷积核大小为 3x3，那么一次3x3的卷积（求右图矩阵一个元素的值）所需运算量：(3x3)个乘法+(3x3-1)个加法 = 17。要得到右图convolved feature （3x3的大小）：17x9 = 153

如果输出 feature map 的分辨率是 H * W ，且输出 o 个 feature map，则输出 feature map 包含 Unit的总数就是 H * W * o。

因此，该卷积层在计算 wx 时有:

```text
k * k * c * H * W * o 次乘法          --（1）
(k * k * c - 1) * H * W * o 次加法    --（2）
```

再考虑偏置项 b 包含的计算量：

由于 b 只存在加法运算，输出 feature map 上的每个 Unit 做一次偏置项加法。因此，该卷积层在计算偏置项时总共包含：

```text
H * W * o 次加法      --（3）
```

将该卷积层的 wx 和 b 两部分的计算次数累计起来就有：

**式(1) 次乘法:**

```text
k * k * c * H * W * o 次乘法
```

**式(2) + 式(3) 次加法:**

```text
(k * k * c - 1) * H * W * o  + H * W * o  = k * k * c * H * W * o
```

**可见，式(2) + 式(3) = 式 (1)**

对于带偏置项的卷积层，乘法运算和加法运算的次数相等，刚好配对。定义一次加法和乘法表示一个flop，该层的计算力消耗 为：

**`k \* k \* c \* H \* W \* o`**

刚好等于图中两个立方体（绿色和橙色）体积的乘积。全连接层的算法也是一样。

## **三、计算flops的开源库**

> 作者：留德华叫兽@知乎

示例代码如下，它求出了VGG16的flops和参数量：

![img](https://pic1.zhimg.com/80/v2-03b36baaa7007081fe97261d76096960_1440w.webp)

可以看到，算上import 和 print（）也仅仅6行代码！不仅输出了整个框架的复杂度，还能输出每一层的复杂度 以及 该层占整个网络的比重

最后贴出常见backbone的flops：

![img](https://pic4.zhimg.com/80/v2-44d89f763a7fdcaafa0063837f939c97_1440w.webp)

## PPT

Parallel Computing: Faster Solutions

西摩·克雷*(Seymour Cray)*被誉为是无可非议的“超级计算机之父”：

If you were plowing a field, which would you rather use? 

Two strong oxen or 1024 chickens?

### What is a Parallel Computer

![image-20230503205228184](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503205228184.png)

![image-20230503211429936](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503211429936.png)

A **shared memory multiprocessor** (SMP*) by connecting multiple processors to a single memory system

A **multicore processor** contains multiple processors (cores) on a single chip

\* Technically, SMP stands for “Symmetric Multi-Processor” 

![image-20230503211744057](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503211744057.png)

A **distributed memory multiprocessor** has processors with their own memories connected by a high speed network

Also called a **cluster** 

A **high performance computing (HPC)** system contains 100s or 1000s of such processors (nodes)

![image-20230503212203998](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503212203998.png)

A **Single Processor Multiple Data (SIMD)** computer has multiple processors (or functional units) that perform the same operation on multiple data elements at once

Most single processors have **SIMD units** with ~2-8 way parallelism

Graphics processing units (**GPUs**) use this as well

对数据进行并行操作，2-8个并行路线

### What’s **not** a Parallel Computer

#### Concurrency vs. Parallelism

•Concurrency: multiple tasks are **logically** active at one time.

•Parallelism: multiple tasks are **actually** active at one time.

![image-20230503214023129](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503214023129.png)

#### Parallel Computer vs. Distributed System

•A distributed system is **inherently** distributed, i.e., serving clients at different locations

•A parallel computer may use **distributed memory** (multiple processors with their own memory) for more performance

![image-20230503214259770](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503214259770.png)

### Units of Measure for HPC 

**High Performance Computing (HPC) units are:**

–**Flop: floating point operation, usually double precision unless noted**

–**Flop/s: floating point operations per second**

–**Bytes: size of data (a double precision floating point number is 8 bytes)**

#### Top 500

Yardstick: Floating Point Operations per Second (FLOP/s) Rmax of Linpack

•Solve Ax=b, Matrix A is dense with random entries

•Dominated by dense matrix-matrix multiply 

| **#**  | **Site**                                                | **Manufacturer** | **Computer**                                                 | **Country**       | **Cores**      | **Rmax**  **[****Pflops****]** | **Power**  **[MW]** |
| ------ | ------------------------------------------------------- | ---------------- | ------------------------------------------------------------ | ----------------- | -------------- | ------------------------------ | ------------------- |
| **1**  | **RIKEN       Center for Computational Science**        | **Fujitsu**      | **Fugaku****     Supercomputer** **Fugaku****,   **    **A64FX  48C 2.2GHz, Tofu interconnect D** | **Japan**         | **7,630,848**  | **442.0**                      | **29.9**            |
| **2**  | **Oak  Ridge      National Laboratory**                 | **IBM**          | **Summit     IBM Power System,  **    **P9  22C 3.07GHz,** **Mellanox**  **EDR, NVIDIA GV100** | **USA**           | **2,414,592**  | **148.6**                      | **10.1**            |
| **3**  | **Lawrence  Livermore      National Laboratory**        | **IBM**          | **Sierra     IBM Power System,**  ** **    **P9  22C 3.1GHz,** **Mellanox**  **EDR, NVIDIA GV100** | **USA**           | **1,572,480**  | **94.6**                       | **7.4**             |
| **4**  | **National  Supercomputing      Center in Wuxi**        | **NRCPC**        | **Sunway**  **TaihuLight****     NRCPC** **Sunway  SW26010,  **    **260C  1.45GHz** | **China**         | **10,649,600** | **93.0**                       | **15.4**            |
| **5**  | **NVIDIA  Corporation**                                 | **NVIDIA**       | **Selene     DGX A100** **SuperPOD****,**  ** **    **AMD  64C 2.25GHz, NVIDIA A100, Mellanox HDR** | **USA**           | **555,520**    | **63.5**                       | **2.65**            |
| **6**  | **National**  **University of      Defense Technology** | **NUDT**         | **Tianhe-2A     ANUDT TH-IVB-FEP,  **    **Xeon  12C 2.2GHz, Matrix-2000** | **China**         | **4,981,760**  | **61.4**                       | **18.5**            |
| **7**  | **Forschungszentrum**  **Jülich**  **(FZJ)**            | **Atos**         | **JUWELS  Booster Module **    **BullSequana**  **XH2000,**  ** **    **AMD  EPYC 24C 2.8GHz, NVIDIA A100, Mell. HDR** | **Germany**       | **449,280**    | **44.1**                       | **1.76**            |
| **8**  | **Eni  S.p.A**                                          | **Dell  EMC**    | **HPC5     PowerEdge C4140,**  ** **    **Xeon  24C 2.1GHz, NVIDIA T. V100, Mellanox HDR** | **Italy**         | **669,760**    | **35.5**                       | **2.25**            |
| **9**  | **Texas  Advanced Computing Center / Univ. of Texas**   | **Dell**         | **Frontera     Dell C6420,**  ** **    **Xeon  Platinum 8280 28C 2.7GHz, Mellanox HDR** | **USA**           | **448,448**    | **23.5**                       |                     |
| **10** | **Saudi  Aramco**                                       | **HPE**          | **Dammam-7     Cray CS-Storm,**  ** **    **Xeon  G. 20C 2.5GHz, NVIDIA T. V100, IB HDR 100** | **Saudi  Arabia** | **672,520**    | **22.4**                       |                     |

![image-20230503220605386](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503220605386.png)

![image-20230503221019266](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230503221019266.png)

世界最强超算芯片Fujitsu A64FX：继承于SPARC64架构的Arm超级处理器

从下一代超级计算到人工智能 (AI) 和图形密集型技术， 三星 一直处于技术前沿，最新开发的先进 8GB HBM2 Aquabolt 是精心设计的高带宽存储器， 相对于其前一代 HBM2 Flarebolt，Aquabolt 极大地提高了性能并降低了能耗。

Linpack是国际上使用最广泛的测试高性能计算机系统浮点性能的基准测试。通过对高性能计算机采用高斯消元法求解一元 N次稠密线性代数方程组的测试，评价高性能计算机的浮点计算性能。Linpack的结果按每秒浮点运算次数（flops）表示。

在电子技术中，脉冲信号是一个按一定电压幅度，一定时间间隔连续发出的脉冲信号。脉冲信号之间的时间间隔称为周期；而将在单位时间（如1秒）内所产生的脉冲个数称为频率。频率是描述周期性循环信号（包括脉冲信号）在单位时间内所出现的脉冲数量多少的计量名称；频率的标准计量单位是Hz（赫）。电脑中的系统时钟就是一个典型的频率相当精确和稳定的脉冲信号发生器。频率在数学表达式中用“f”表示，其相应的单位有：Hz（赫）、kHz（千赫）、MHz（兆赫）、GHz（吉赫）。其中1GHz=1000MHz，1MHz=1000kHz，1kHz=1000Hz。计算脉冲信号周期的时间单位及相应的换算关系是：s（秒）、ms（毫秒）、μs（微秒）、ns（纳秒），其中：1s=1000ms，1 ms=1000μs，1μs=1000ns。

CPU的主频，即CPU内核工作的时钟频率（CPU Clock Speed）。通常所说的某某CPU是多少兆赫的，而这个多少兆赫就是“CPU的主频”。很多人认为CPU的主频就是其运行速度，其实不然。CPU的主频表示在CPU内数字脉冲信号震荡的速度，与CPU实际的运算能力并没有直接关系。主频和实际的运算速度存在一定的关系，但目前还没有一个确定的公式能够定量两者的数值关系，因为CPU的运算速度还要看CPU的流水线的各方面的性能指标（缓存、指令集，CPU的位数等等）。由于主频并不直接代表运算速度，所以在一定情况下，很可能会出现主频较高的CPU实际运算速度较低的现象。比如AMD公司的AthlonXP系列CPU大多都能已较低的主频，达到英特尔公司的Pentium 4系列CPU较高主频的CPU性能，所以AthlonXP系列CPU才以PR值的方式来命名。因此主频仅是CPU性能表现的一个方面，而不代表CPU的整体性能。

CPU的主频不代表CPU的速度，但提高主频对于提高CPU运算速度却是至关重要的。举个例子来说，假设某个CPU在一个时钟周期内执行一条运算指令，那么当CPU运行在100MHz主频时，将比它运行在50MHz主频时速度快一倍。因为100MHz的时钟周期比50MHz的时钟周期占用时间减少了一半，也就是工作在100MHz主频的CPU执行一条运算指令所需时间仅为10ns比工作在50MHz主频时的20ns缩短了一半，自然运算速度也就快了一倍。只不过电脑的整体运行速度不仅取决于CPU运算速度，还与其它各分系统的运行情况有关，只有在提高主频的同时，各分系统运行速度和各分系统之间的数据传输速度都能得到提高后，电脑整体的运行速度才能真正得到提高。

提高CPU工作主频主要受到生产工艺的限制。由于CPU是在半导体硅片上制造的，在硅片上的元件之间需要导线进行联接，由于在高频状态下要求导线越细越短越好，这样才能减小导线分布电容等杂散干扰以保证CPU运算正确。因此制造工艺的限制，是CPU主频发展的最大障碍之一。

#### Other Algorithms / Arithmetic

•Mixed precision iterative refinement approach solved a matrix of order 16,957,440 on Fugaku.

–Composed of nodes made up of Fujitsu’s ARM A64fx Processor 

–The run used 158,976 nodes of Fugaku, 7,630,848 cores 

–Used a random matrix with large diagonal elements to insure convergence.

•Mixed precision HPL achieved 2.004 Eflop/s

–4.5 X over DP precision HPL (442 PFLOPS).

–67 Gflops/Watt

•Same accuracy compared to full 64 bit precision

Gordon Bell Prizes: Science at Scale

成立于 1987 年，现金奖励为 10，000 美元（自 2011 年起），由 HPC 的先驱戈登·贝尔资助。
在将 HPC 应用于科学、工程和数据分析应用方面的创新。

### **Science using High Performance Computing**

#### Simulation: The Third Pillar of Science

![image-20230505113243633](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505113243633.png)

**Simulations Show the Effects of Climate Change in Hurricanes**

HPC for Astrophysics

HPC for Energy Efficiency in Industry

HPC for Carbon Capture

Screening known drugs for COVID-19

#### **The Fourth Paradigm of Science** 

![image-20230505114713473](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505114713473.png)

Data analytics in science and engineering

High Performance Data Analytics (HPDA) is used for data sets that are:

•too big

•too complex

•too fast (streaming)

•too noisy

•too heterogeneous

for measurement alone

Data Growth is Outpacing Computing Growth

High Performance Data Analytics (HPDA) for Genomics

Analysis of Genomic Data

#### **The Fifth Paradigm of Science** 

![image-20230505114942306](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505114942306.png)

Machine learning demands more computing

![image-20230505114958888](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505114958888.png)

Data Analytics via Supervised Learning

Big Data, Big Model, and Big Iron

GANs to build convergence maps of weak gravitational lensing

### The Motifs of Parallel Computing

#### How to cover all applications

•Phil Colella’s famous 7 “dwarfs” of scientific computing (simulation)

![image-20230505115403637](C:/Users/86155/AppData/Roaming/Typora/typora-user-images/image-20230505115403637.png)

Motifs: Common Computational Methods

Analytics vs. Simulation Motifs

| **7 Giants of Data** | **7 Dwarfs of Simulation** |
| -------------------- | -------------------------- |
| Basic  statistics    | Monte Carlo methods        |
| Generalized N-Body   | Particle methods           |
| Graph-theory         | Unstructured meshes        |
| Linear  algebra      | Dense Linear Algebra       |
| Optimizations        | Sparse Linear Algebra      |
| Integrations         | Spectral  methods          |
| Alignment            | Structured Meshes          |

#### Motifs of Genomic Data Analysis

**Application problems**

•Overlap: Find all overlaps in a set

•Assembly: find / correct overlaps

•Distance: how good are overlaps

•Index: lookup in database

Large shared memory platforms were most common – limits science questions and approaches

### **Why the Fastest Computers are Parallel Computers**

Power Density Limits Serial Performance

•*Faster processors* → *increase power*

–Dynamic power is proportional to V2fC

–Increasing frequency (f) also increases supply voltage (V) à  cubic effect

–Increasing cores increases capacitance (C) but only linearly

–Save power by lowering clock speed and adding parallelism

•High performance serial processors waste power

-Speculation, dynamic dependence checking, etc. burn power

-Implicit parallelism discovery

•More transistors, but not faster serial processors
