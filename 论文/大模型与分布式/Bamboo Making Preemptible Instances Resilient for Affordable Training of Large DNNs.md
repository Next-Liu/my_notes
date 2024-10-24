**Bamboo: Making Preemptible Instances Resilient for Affordable Training of Large DNNs**



### 背景：

现有的容错技术（如基于检查点的恢复）在频繁中断的情况下效率不高。（抢占率：使用预付费（spot）实例进行分布式深度学习训练时，实例被云计算服务提供商因为优先级更高的任务或市场价格上涨而中断（preempted）的频率。这个比率反映了实例被抢占的频繁程度，对于设计容错和高效的训练系统至关重要）通过模拟不同的抢占率来评估Bamboo系统的性能，包括训练吞吐量、成本和性能成本比（value）

### 创新点：

- Bamboo通过在训练流水线中引入冗余计算来应对中断。
- 每个节点不仅计算自己的层次（layers），还计算邻居节点的部分层次。
- 利用流水线并行（pipeline parallelism）中的“流水线气泡”（pipeline bubbles）来安排冗余计算，以此减少额外开销。
- 当节点被中断时，它的前一个节点可以使用预先计算好的冗余信息来恢复训练，无需从头开始。



**RC（Redundant Computation）**、**Redundant Layers**：每个节点在同一流水线中的前一个节点上复制自己的模型分区(即,层分片)，**核心思想：**让每个节点在自己的层上运行正常的(向前和向后)计算，并为其后继节点在副本层上运行冗余的(向前和向后)计算，**这种设计允许在一个节点被抢占（preempted）或失败时，其前一个节点（predecessor node）已经有了足够的信息（例如模型层、激活值等）来继续训练过程，而不需要从头开始或从检查点（checkpoint）恢复。**和**Check-N-Run中的间歇性差分checkpoint思想相似，不用从头恢复**

### **冗余计算分为：**

**FRC：**前向冗余计算（每个结点对本层进行前向计算和后继节点的模型层执行前向冗余计算），这样可以允许其前一个节点利用已经计算好的冗余数据继续执行训练，从而减少恢复时间。

**执行时机**：FRC 被安排在流水线中的空闲时间（bubble time）内执行，即在节点完成自己的前向传播但还未接收到后继节点的数据时进行。

**BRC：**后向冗余计算

- 当一个节点失败时，它的后继节点（即流水线中的下一个节点）会丢失一些中间数据（如梯度信息）。
- BRC通过在前一个节点（即流水线中的上一个节点）上重新计算这些丢失的数据来恢复模型状态。

**懒加载冗余计算（Lazy BRC）：** **只在节点实际被中断时**才执行反向冗余计算。

**流水线气泡：** 在流水线并行训练中自然存在的空闲时间（前向传播和反向传播的间隔）



### 冗余计算的调度

RC计算在时间和空间都需要开销，解决措施

( 1 )将FRC调度到管道气泡中，以减少前向计算开销；

( 2 )惰性执行BRC，以减少后向计算/通信开销；

( 3 )将不必要的中间结果交换到CPU内存，以减少GPU开销。

![](D:\学习笔记\论文\pictures\Snipaste_2024-10-10_09-59-06.png)

### **Bamboo系统总览**

关注参数：**D：数据并行流水线的数量**（训练任务被分成多少份在多个节点同时运行），**P是流水线深度（模型被分割成多少部分在不同节点同时处理）**。

P*D代表训练规模：当节点因为抢占而丢失时，Bamboo可能会因此而减少其管道深度（P）和/或管道数量（D）。这是因为部分训练任务因为节点的丢失而无法继续执行。**自动伸缩以维持规模：** 为了应对节点的抢占，Bamboo会请求更多的实例（节点）加入训练，以试图将集群的大小恢复到P × D定义的目标规模。这是为了确保训练任务有足够的计算资源。



相比checkpoint方法不用从头恢复，可以从上一个结点快速恢复状态，吞吐量提高，成本减少