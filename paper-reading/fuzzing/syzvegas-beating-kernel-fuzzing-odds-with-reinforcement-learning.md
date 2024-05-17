# SyzVegas: Beating Kernel Fuzzing Odds with Reinforcement Learning

[SyzVegas: Beating Kernel Fuzzing Odds with Reinforcement Learning](https://www.usenix.org/system/files/sec21-wang-daimeng.pdf) 是一篇 2021 年发表在 USENIX 的研究型论文（工程创新）。一作是来自 University of California, Riverside 的  Daimeng Wang，其它作者同样来自 University of California, Riverside，他们是 Zheng Zhang, Hang Zhang Zhiyun Qian, Srikanth V. Krishnamurthy, Nael Abu-Ghazaleh。

该论文的贡献：基于 Syzkaller 借助 reinforcement learning，修改 Syzkaller 中两个最关键的决策点——task selection and seed selection，提出了新 kernel fuzzer——**SyzVegas**，对比 Syzkaller，在执行效率与覆盖率方面有了一定的提升。

### 背景

1. _Kernel fuzzing is uniquely **challenging** for the following reasons: (1) modern OS kernel often has a huge code base and many dependencies across components; (2) the input to an OS kernel is via the system call interface that needs special handling; and (3) an OS kernel maintains a massive state space that a single input (i.e., test case) may not be able reproducible._&#x20;
2. workflow overview of Syzkaller

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### **问题**

1.  Syzkaller 硬编码策略无法适应动态变换的模糊测试环境与任务，同时基于作者的 observations，Syzkaller 性能方面仍存在提升的空间；                                                                                                                      &#x20;

    <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>
2. 作者将问题 1 转化成如下问题：确定 **SyzVegas** 的目标在于提升 coverage 的前提下，（1）选择最有价值的模糊测试任务；（2）选择最有效的突变种子。

### **设计与实现**

对于上述**问题 2**，既然要求选出最有价值的任务和种子，则需要一个标准进行衡量。如何建构这个标准？

基于 observations，作者发现“正确”的任务和种子是随着时间的推移而动态变化的，因此需要一种自动化的方式来识别给定时间最有价值的任务，并且适当地调用与该任务相关的最佳种子。

> To identify these, a reinforcement-learning scheme is a natural fit, wherein a model to maximize the **coverage rewards** relative to the **time cost of execution** is applicable.

在众多的 reinforcement-learning 中，Multi-Armed Bandit (MAB) 是最适合的，因为 MAB 的设计目标是在面对不确定性的情况下，用局部最优去追求全局最优，最大化总体累积奖励。加上 Syzkaller 本质上是对未知空间进行搜索，reward 分布是未知的。因此在 reward 未知的前置条件下，选用 MAB 能在 exploration 与 exploitation 之间做到很好的平衡。

最终，作者决定在 **SyzVegas** 中通过 Adversarial MAB（especially, Exp3），去解决[**问题 2**](syzvegas-beating-kernel-fuzzing-odds-with-reinforcement-learning.md#wen-ti)。

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>High-level idea/design of SyzVegas</p></figcaption></figure>

因为  goal 是在最短的时间内追求最高的覆盖率，而 reward 作为一种导向指标，因此 reward 的基本定义是：变异程序的覆盖率与原始种子程序的覆盖率之间的差值除以变异任务的时间成本。

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>Task Selection</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Seed Selection</p></figcaption></figure>

**SyzVegas** 的实现效果与更多其它细节，读者可自行去原文查阅。



目前笔者对顶级研究者的研究思路总结如下：

phenomenon

→ observation & intuition

→ strategy(scenario taken into account) & implementation

→ evaluation & rethink

→ improverment?





