# ChameleMon: Shifting Measurement Attention as Network State Changes

[ChameleMon: Shifting Measurement Attention as Network State Changes](https://dl.acm.org/doi/10.1145/3603269.3604850) 是一篇 2023 年发表在 ACM 的研究型论文。共同一作是来自 National Key Laboratory for Multimedia Information Processing, School of Computer Science, Peking University 的 **Kaicheng Yang，Yuhan Wu 和 Ruijie Miao**，通讯作者是来自同样单位的 Tong Yang。ChameleMon 提供了 [prototype](https://github.com/ChameleMoncode/ChameleMon)。

### 背景与前人工作

Network measurement 的目标是支持更多任务并以更少的内存实现更高的准确性。

> _There are mainly two kinds of flow-level measurement tasks in Network measurement : **1) packet accumulation tasks and 2) packet loss tasks.**_&#x20;

前人工作的局限性在于：要么只能完成两种任务之一，受限于其数据结构无法拓展完成另一种任务；要么能同时完成两种任务，但无法充分利用资源，导致至少线性级别的内存 / 带宽开销。

### 解决方案

本论文经研究总结，提出一个实用的测量系统需要满足以下三个要求：

1. 通用性：同时支持丢包测量任务和包累积测量任务；&#x20;
2. 效率：以亚线性空间复杂度实现高精度；&#x20;
3. 注意力转换：在不同网络状态下关注不同类型的任务。

本论文提出了一种新型的测量系统——**ChameleMon**，该系统能同时满足以上三个要求，突破了现有工作的瓶颈。

ChameleMon 的关键设计是通过动态的两个维度，实现随着网络状态的变化而转移测量注意力：1）在两种任务之间动态分配内存； 2）动态监控重要流量。为了实现关键设计，本论文提出了一项技术：利用费马小定理设计灵活的数据结构——**FermatSketch**。 FermatSketch 具有可除性、加法性和减法性，支持以上两种测量任务。 ChameleMon 的 workflow 如下：

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

#### Step 1: Capture flow-level statistics

对于一个 flow，通过 Classifier 将其分为四类（分类依据：flow size）：

* Heavy-hitter(HH)
* heavy-losses(HLs)
* light-losses(LLs) - according to sample rate divied into two classes
  * sampled LL candidate
  * non-sampled

UpStream 统计进入网络的 flow，DownStream 统计离开网络的 flow。

#### Step 2: Collect from edge

中央控制器定期从边缘交换机收集 sketches 以支持持续测量。 为避免与 flow 插入发生冲突，收集 sketches 时，每个边缘交换机都会将时间线划分成连续的固定长度时间间隔（称为 epoch），以及复制一组 sketches 作为备份。&#x20;

#### Step 3: Perform analysis

每个 epoch，中央控制器对收集的 sketches 进行分析（共支持七项测量任务）。 通过分析上下游的编码器，中央控制器可以支持丢包检测。 通过分析流分类器和上游 HH 编码器，中央控制器可以支持 HH 检测和其他五个数据包累积任务。

#### Step 4: Shift attention

每个 epoch，中央控制器通过分析收集到的 sketches 来监控实时网络状态（若能监控所有的 victim flow，则为 healthy 状态，反之为 ill 状态），在运行时重新配置边缘交换机的数据平面。通过调整内存分配，来控制两种任务的占比；通过调整 Tl 和 Th 这两个阈值，动态选择最重要的流。

实验结果能够证明 ChameleMon 达到了其所描述的效果：

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

美中不足的是，该论文在整体排版上不够美观，page5-8 没有任何图表，EVALUATION 部分草草收尾，附录部分又占比过大，不免令读者怀疑作者是否存在写作仓促的情况。

