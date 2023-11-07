# 1dFuzz: Reproduce 1-Day Vulnerabilities with Directed Differential Fuzzing

这是一篇 2023 年发表在 CCS 上的[研究型论文](https://dl.acm.org/doi/abs/10.1145/3597926.3598102)，第一作者是来自清华大学的杨松涛，通讯作者是来自清华大学的张超老师。

现有研究表明大多数用户没有及时升级和修补程序， 容易受到 1day vulnerabilities 的影响，并且漏洞时间窗口的持续时间非常长（例如几个月甚至几年），使 1day vulnerabilities 成为巨大的潜在威胁。该论文提出一种解决方案——**1dFuzz** ，实现对 **1day vulnerabilities 的识别、定位与利用**。

### 背景与前人工作

1day 存在的根本原因是**从漏洞发布到补丁应用的时间差。**1day 相较于 0day 的特殊之处在于每个人都能通过漏洞发布得知漏洞的存在，这使得 1day 的利用比 0day 更加容易。为了对抗 1day 带来的威胁，现有两种解决方案：

1. 应用 patch；
2. 在软件供应链中，从系统或网络层面扫描 1day。

第一种方案依赖于用户的自觉，已被证明是不可靠的方案。若要采用第二种方案，自然引出以下两个子问题：

1. _Where are the patches in the patched binary?_
2. _how to generate PoCs to trigger patches?_

解决子问题 1 的基本思路是差分测试。差分测试可以分为三类：（1）static binary diffing；（2）dynamic binary diffing；（3）AI-based binary diffing。差分测试面临的困难在于——编译策略、代码更新等一系列操作都会造成 binary 的差异，该如何剔除这些噪声，准确识别并定位 patch？

解决子问题 2 的基本思路有两种：（1）symbolic execution；（2）directed fuzzing。

### 解决方案

将 “**识别、定位并利用 1day vulnerabilities**” 拆解成以上两个子问题后，以下考虑逐步解决这两个子问题。

#### 解决子问题 1

现存 AI 无法识别细粒度的 patch，故解决子问题 1 时不考虑_（3）AI-based binary diffing_。

为了探究如何定位并识别 patch 的问题，作者研究了 1031个 patches 后指出，patch 独特且常见的特征是 **trailing call sequences (TCS)**，其定义与分类如下：

> Definition 1. Trailing Call Sequence (TCS). Given a program P and an input I, we denote the last N function calls before program P exits as the trailing call sequence TCS(P,I,N).

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

基于 TCS，可以实现对 patch 的识别与定位，从而解决子问题 1。具体实现过程：通过静态分析，构建 CG & CFG，做函数对齐去除噪声。对于每对对齐的函数，通过遍历 CG & CFG 的方式，获得两组被调用函数序列，比较这两组序列并检查它们是否不同。 如果一个集合具有另一集合中不存在的被调用函数序列，将把对齐的函数标记为 candidate patched function，这将通过定向模糊测试进一步探索。以上描述的过程对应下图红色方框部分。

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

#### 解决子问题 2

symbolic execution 无法解决大型 binary 的路径爆炸问题，故解决子问题 2 时不考虑_（1）symbolic execution_。

为了探究如何触发 patch 的问题，首先提出一种基于 TCS 的定向 metric：基本思路是衡量 distance(source function, target function)。函数之间的 distance 定义为 1 + 1 / (2+CN) （CN 是从 caller → callee，call sites 的数量）；找到 target function 的 dominator function（前置必经函数），将 distance(dominator function, target function) 定义为测试用例的 distance。

完成了对 metric 的定义，下一步采用 **directed fuzzing** 方案尝试触发 patch，达到解决子问题 2 的目的。在 directed fuzzing 中，距离越小的 seed 优先级越高，但为了避免陷入局部最优解，1dFuzz 也会以小概率随机挑选一些种子优先测试。测试借助 1dSan 来捕获 PoCs。

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

1dFuzz 详细的实现流程请参考下图：

&#x20;

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

部分实验效果如下：

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

基于 TCS 设计的 1dFuzz 误报率较高，这是一个 limitation。此外作者将对测试场景进行扩展（非 C / C++ programs）以及探索补丁新特征留给 future work 去解决。
