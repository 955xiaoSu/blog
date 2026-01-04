# SyzDirect: Directed Greybox Fuzzing for Linux Kernel

这是一篇 2023 年发表在 CCS 上的[研究型论文](https://dl.acm.org/doi/10.1145/3576915.3623146)，第一作者是来自复旦大学的 Xin Tan，共同一作和通讯作者是来自复旦大学的 [Yuan Zhang](https://yuanxzhang.github.io) 老师。作者提供了 [prototype](https://github.com/seclab-fudan/SyzDirect)（需等待作者 push）。

现有 DGF 主要关注 user application，无法很好地适应 OS Kernel 此类测试对象。 目前最先进的  OS Kernel DGF 是 GREBE。 然而，GREBE 是一种针对特定任务量身定制的方法，而不是通用的 DGF 解决方案，并且 GREBE 需要 PoC 作为输入，这在典型的 DGF 场景中是不现实的。因此，该论文提出一种解决方案——**SyzDirect**，第一个针对 Linux Kernel 的通用 DGF 解决方案。

### 背景与前人工作

现有的 DGF 主要采用两种快速接近目标位置的策略：**距离引导探索**和**无效输入剪枝**，作者决定将这两种策略保留以提高效率。

此外，由于测试对象为 OS Kernel，Kernel 的特性自然是值得注意的。Kernel 与上层软件的交互接口为 syscall。syscall 一般具有 3\~5 个参数，每个参数往往是复杂的结构体。syscall 之间存在依赖关系，这说明 syscall 的调用序列也会影响程序的状态。 如果认为 user application 接受的输入是随机的字符流，是一种一维的输入，那么 OS kernel 接受的输入 syscall 可以认为是一种二维的输入。这意味着 Kernel Fuzzing 面临着更大的状态空间等待搜索，那么如何用有限的算力去探索一个巨大的状态空间，变成一个亟待解决的难题。

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption><p>syscall</p></figcaption></figure>

综上，Kernel Fuzzing 遇到的 challenge 主要有以下两个：

1. 与采用固定入口点的 user application 不同，OS Kernel 实现了数百个系统调用（syscall）作为与 Kernel  交互的主要接口；
2. 除了 syscall 之外，准备所需的 syscall 参数以满足目标执行路径的约束也很重要。 否则，DGF 很容易陷入无效的探索。

进一步精确定义 Challenge：**Entry Point & Argument Preparation。**

要做的工作：解决 **Entry Point & Argument Preparation**，使程序进入到特定的状态，达到 reproduce 与 patch testing 的目标。

### 解决方案

#### 解决 Entry Point

作者通过前期的调研发现，由于 Linux Kernel 中大量使用间接调用，应用现有的控制流分析技术来识别入口系统调用会引入很多误报，因此不能仅依赖于控制流分析。作者仔细思考了 Linux Kernel 的本质，认为 Linux Kernel 是一个资源管理器，其将底层设备抽象为文件、套接字、设备等。syscall 是对资源进行操作（创建、更新、回收等）的接口，而内核函数则提供操作的实际实现。那么是否可以尝试通过内核函数来刻画 syscall 呢？这就引入了作者的 key insight：

> We can match kernel functions with syscalls based on what resources they operate on and how they operate on the resource.

基于此 key insight，作者对 operation 和 resouce 进行建模，从而利用内核函数完成了对 syscall 的刻画。

<figure><img src="../../.gitbook/assets/image (89).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (90).png" alt="" width="563"><figcaption></figcaption></figure>

此外，作者定义了 anchor func 来辅助定义 Entry Point。定义 Entry Point 流程如下：

1. 接受给定目标代码位置；
2. 使用基于类型的间接调用分析，构造 CFG，并识别可以达到目标的原始系统调用；
3. 利用 CFG 和内核函数建模，将目标函数与系统调用变体进行匹配；
4. 将可达的原始系统调用和匹配的系统调用变体的交集，作为目标的入口系统调用位置，修剪不可行的分析结果。

以 Fig.1 为例，阐述以上流程。

<figure><img src="../../.gitbook/assets/image (91).png" alt="" width="468"><figcaption><p>Fig.1</p></figcaption></figure>

调用链：rds\_rdma\_extra\_size → rds\_sendmsg → rds\_proto\_ops → RDS socket → The creation of RDS socket → trace into rds\_create and obtain the assignment of rds\_proto\_ops → rds\_sendmsg → SOCK\_SEQPACKET → the anchor function rds\_sendmsg requires a AF\_RDS SOCK\_SEQPACKET resource。

#### 解决 Argument **Preparation**

为了解决 Argument Preparation，需要开展 Syscall Dependency Inference 和 Syscall Argument Refinement 两部分工作。前者主要参考了 [Healer](https://dl.acm.org/doi/10.1145/3477132.3483547) 提出的算法，后者主要借助了 Syzkaller 中的 Syzlang，实现了对不可达路径的 prune（即删去下图画红线的部分）。

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

解决 Entry Point 和 Argument Preparation 后，作者将得到的结果称为 **template**。利用 template 去指导 mutation，极大提高了 reproduce 和 patch testing 的效率。SyzDirect 采用了基于距离的探索，距离越短，seed 优先级越高，seed 得到的能量也越多。SyzDirect 实现流程可参考下图：

&#x20;

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

与 Syzkaller 和 SyzGo 对比的部分实验效果如下：

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

SyzDirect 仍具有一些局限性：

1. 静态分析不执行对参数条件等相关系统调用的深入分析；
2. 无法生成满足触发错误的严格约束的参数；
3. 生成对象参数（例如文件系统映像）仍然非常具有挑战性。
