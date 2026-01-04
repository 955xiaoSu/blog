# Araña: Discovering and Characterizing Password Guessing Attacks in Practice

[Araña: Discovering and Characterizing Password Guessing Attacks in Practice ](https://www.usenix.org/system/files/sec23fall-prepub-22-islam.pdf)总结：

* Araña: Discovering and Characterizing Password Guessing Attacks in Practice 是一篇 2023 年发表在 USENIX 的研究型测量类论文。共同一作是来自 University of Wisconsin–Madison 的  Mazharul Islam 与 Cornell Tech 的 Marina Sanusi Bohuk，其它作者为来自 University of Wisconsin–Madison 的 Paul Chung、Rahul Chatterjee 以及来自 Cornell Tech 的 Thomas Ristenpart。
* 该论文研究的领域是发现并识别——Remote password guessing attacks。通过研究，尝试将 attacks 分类提取特征，为安全防护提供思路。
* Problem
  1. 现存数据集中没有任何关于此类攻击的基本事实；
  2. 以往的工作只单独标记向登录服务发送大量请求的 IP 地址，这样的做法 miss many attacks。
* Key Findings
  * 设计了一个名为 Araña 的分析框架，发现并描述了针对两所大学的 29 个攻击集群，确定了这两所大学的身份验证系统受到的攻击的关键特征和模式，并讨论了身份验证系统应如何应对此类攻击。
*   Methodology

    * 首先通过 Gossamer 收集登录日志，以如下格式提取信息；

    <figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

    * 基于“恶意登录只占少数、恶意登录大多数情况下会失败、恶意登录是通过脚本控制且 ip 来自于 proxy server”的三点假设，以 **NR** 与 **FF** 为度量，根据实验事实，选取 thresholds 过滤 benign behavior，将剩余的 malicious L sets 作为下一步的研究对象；

    <figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

    * 选取 nominal features IP, ISP, UA, and DATE ，设计 normalized difference，denoted as ND(x,y)=|x-y|/(x+y) 作为 clustering 的依据。当两个 cluster 的距离小于通过 knee locator method 选出的 threshold，便将这两个 cluster 归为一类；

    <figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

    * 通过发起攻击的时空统计特征，将攻击分成 29 类。再通过“Type of username password pairs”和“Delivery of requests”实现对 attack campaigns 更高程度的抽象。



    <figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>
*   Suggestions

    * 针对此类攻击，并没有太好的解决办法，因此作者只泛泛地描述了如下的防御策略：

    > _“For example, locking user accounts due to a small number of incorrect attempts rarely translates to higher security, whereas discouraging users from reusing passwords from other websites and using breach alerting services can be very effective. Proactive breach alerting \[27] using services such as HIBP \[22] would be very helpful in combating credential stuffing attack”_


* 该论文实事求是，不夸大研究价值，写作上严丝合缝，实验部分与 Intro 部分前后呼应。
