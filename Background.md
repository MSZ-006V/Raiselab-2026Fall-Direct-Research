## 2025 Fall Research plan
- 背景：在一个Server任务执行系统中，不断有新的task来到（unpredictable arrival），系统需要调度这些任务执行。对于每一个任务，需要在限定时间（budget，可假设为100ms）内完成执行
    - task: 计算任务，内部包含多个子任务，具有固定的执行时间（可假设为50~100ms）
    - metrics：每个任务拥有两个评价指标，QoS与ACC
    - 假设通信模型为Client - Server架构，存在固定的网络延迟，可假设为30~60ms
    - 每个Task的执行组成部分可分为：Total Time = Network Delay + Execution Time
- Tradeoff：在每个任务执行budget固定的情况下（假设为100ms），根据传输过程中的网络延迟，动态调整Execution Time来保证Task可以在限定时间内执行完
    - 暂时不关心执行质量（如QoS与ACC），关心如何在限定时间内完成该Task
- 考虑的点包括
    1. 异构核（如CPU，GPU多种异构系统下执行）
    2. 抢占（Task和Task之间可以根据优先级抢占）
    3. Budget Tradeoff：根据网络延迟动态调整执行时间
- Task Execution Time = Network Delay + k * (m * QoS + n * ACC) * Expect Execution Time <= Time Budget
### 调研方向
1. ROS2 - Real time system相关，如调度器，任务执行器等等
    - 了解ROS2是什么，系统架构是什么样的
    - 阅读一些相关论文，看看别人是怎么做的
    - 阅读ROS2的default执行器（可能是FIFO的）和魔改版本
2. 出发点：首先在满足第三点（即通过tradeoff QoS和ACC，保证任务在budget内执行完成）的基础上，考虑抢占或者异构核执行
3. 了解self driving pipeline(perception, localization, planning等)
4. Linux LST (Least slack time scheduling): Least slack time (LST) scheduling or least laxity first is an algorithm for dynamic priority scheduling. [wiki](https://en.wikipedia.org/wiki/Least_slack_time_scheduling)

## 2026 Spring Research Plan
- 需要做一个slide！！！

- 关键词：Crowd Computing, Mobile Crowd Computing
- 定义：利用用户终端（手机，IoT）的闲置CPU，通过任务拆分，把计算下发到终端执行
- 终端是计算节点，节点不可信，需要随时离线
- 经典问题：任务调度（task placement），容错 & Checkpoint

### 任务拆分
- 常见的任务拆分模型：data parallel（拆数据，计算逻辑相同。问题是数据分布不均，节点算力差距），task graph、DAG（拆分为多个子任务，存在依赖关系。经典问题如DAG Scheduling NP-hard，节点掉线=DAG断裂），pipeline/Streaming（任务是连续流，每个节点处理一个阶段，问题就是单点瓶颈

1. 一个例子
- 分布式图像特征提取
- 如10万张图片，拆分为每个task处理1000张（无状态，独立执行）
- 分为Split，Placement, Execution, Aggregation

### 架构相关
1. Ray Architecture

2. KubeEdge(k8s extension)

3. BOINC(volunteer Computing)