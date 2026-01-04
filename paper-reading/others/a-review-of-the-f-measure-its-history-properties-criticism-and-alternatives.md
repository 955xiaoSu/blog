# A Review of the F-Measure: Its History, Properties, Criticism, and Alternatives

[A Review of the F-Measure: Its History, Properties, Criticism, and Alternatives](https://dl.acm.org/doi/full/10.1145/3606367) 总结：

* A Review of the F-Measure: Its History, Properties, Criticism, and Alternatives 是一篇 2023 年发表在 ACM 的综述。第一作者是来自 The Australian National University, Australia 的  Peter Christen，第二作者与第三作者分别是来自 Imperial College London, UK 的 David J. Hand 与 The Australian National University, Australia 的 Nishadi Kirielle。
* 该综述介绍了 F-measure —— 在分类领域衡量分类性能的一种方法。该综述总结了 F-measure 的历史、属性，并讨论了对 F-measure 的批评，提供 F-measure 的替代方案，以及如何有效使用它的建议（方便起见，该综述仅讨论了 binary classification 的情况）。
* 背景：创建分类方法的核心挑战是评估分类方法的性能。分类方法可以分成 problem-based 和 accuracy-based 两类，F-measure 被用于衡量 accuracy-based 分类方法的性能。
* 机制与适用场景： F-measure 结合召回率和精确率，产生一个数字，通过该数字可以评估和比较分类方法。仅当研究人员在给定问题下对负类（通常占大多数）根本不感兴趣时，F-measure 才是合适的分类方法的性能测量工具。
*   历史：&#x20;

    *   F-measure 概念的提出可以追溯到 Van Rijsbergen 1979 年出版的 Information Retrieval，其被用于评估搜索引擎检索到的排序文档的质量。召回率 R、精度 P 与检索效率 E 的定义如下：

        > — Precision, P, is the fraction of retrieved documents that are relevant.&#x20;
        >
        > — Recall, R, is the fraction of relevant documents that are retrieved.
        >
        > ![](<../../.gitbook/assets/image (53).png>)


    * F-measure 被广泛应用于信息提取、文本分类等多个计算机领域。
    * 根据 google scholar 的检索结果显示，1980-2020年 对 F-measure 的检索数量总体上呈上升趋势。

    <figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>
*   属性：该综述讨论的 F-measure 机制假定为当 score(x) > thresholed 时，x 被划分入 class 1，否则被划分入 class 0。召回率 R、精度 P 与检索效率 E 的定义如下：

    > — Recall R = TP/(TP + FN),
    >
    > &#x20;— Precision P = TP/(TP + FP).
    >
    > ![](<../../.gitbook/assets/image (58).png>)
* 对 F-measure 的批评：
  * 忽略了 TN；
  * 使用不同的 P 与 R 也有可能得到相同的 F-measure score；
  * P 与 R 受到 threshold 的影响是不对称的；
  * 这是一个有用的数字总结，但它不能代表正在评估的分类方法的任何客观特征。
* F-measure 的替代方法与使用建议：
  * 仅当研究人员在给定问题下对负类（通常占大多数）根本不感兴趣时，才考虑 F-measure；
  * 评估分类方法时，不仅要报告 F-measure，而且还要报告召回率和精度；
  * 如果偏向可解释的测量，可使用 F\*-measure；
  * 如果想明确赋予精确度和召回率相应的权重，则可以分别使用方程 (2) 或 (4) 中的一般加权 Fβ 或 Fα 版本；
  * 如果无法指定特定的分类阈值，建议使用 H-measure。
* 评估：该综述文笔优美，论证充分。美中不足的是，4.1、4.3 section 与 Appendix A 的论述高度重叠，并且在 A.2 Modifying the F-Measure 一节中，讨论 w 的取值时，没有给出有科学价值的判断，只传递了“笼统取值 w=1/2 即可”的信息。

附：关于课堂上“一刀切”算法的 P-R curve 绘制

<figure><img src="../../.gitbook/assets/P-R_curve.jpg" alt=""><figcaption></figcaption></figure>
