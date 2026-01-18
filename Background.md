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
    - 介绍一下分布式计算流程
    - 目前分布式计算框架（任务调度框架）的问题：straggler，churn, trust
    - 相关的框架，优缺点，可能可以优化的工程细节
    - 

- 关键词：Crowd Computing, Mobile Crowd Computing
- 定义：利用用户终端（手机，IoT）的闲置CPU，通过任务拆分，把计算下发到终端执行
- 终端是计算节点，节点不可信，需要随时离线
- 经典问题：任务调度（task placement），容错 & Checkpoint

### 任务拆分
- 常见的任务拆分模型：data parallel（拆数据，计算逻辑相同。问题是数据分布不均，节点算力差距），task graph、DAG（拆分为多个子任务，存在依赖关系。经典问题如DAG Scheduling NP-hard，节点掉线=DAG断裂），pipeline/Streaming（任务是连续流，每个节点处理一个阶段，问题就是单点瓶颈

1. 一个例子
- 分布式图像特征提取- 如10万张图片，拆分为每个task处理1000张（无状态，独立执行）
- 分为Split，Placement, Execution, Aggregation
- 待解决问题：straggler（低端手机拖慢整体进度），Churn（节点随时消失），Energy，Trust，Overhead等

### 待解决问题
- straggler：执行链中被少数执行特别慢的task拖慢
- churn：节点下线（比如频繁加入，离开，容易造成task丢失和DAG断裂）
- energy：
- trust：edge device的计算是否可信（恶意节点？软件被篡改，伪造）
- overhead：系统开销（比如调度+通信+状态维护成本加起来比计算成本还大）

### 架构相关
1. Ray Architecture: task-based distributed runtime
- 可以实现runtime自动拆分task，调度，传输数据等
- 问题：心跳开销大，node churn崩溃（GCS强依赖，什么是GCS？）worker太重（Python，什么意思？）
- 改进idea：针对ray architecture在edge/mobile下的性能问题，做出一些改进    
- scheduler改造：加node volatility model，battery-aware placement（实验 churn vs completion time）    
- control plane精简：替代GCS的强一致依赖，使用更弱的membership model    
- task granularity：自适应 task size（比如根据device的性能），或者是小task + speculative execution- 实验细节    
- 可以随机kill，使用bandwidth cap，或者是energe budget    
- ray archi使用python

2. KubeEdge(k8s extension)
- 不适合手机，假设节点长期在线，需要container+kubelet，基础设施比较沉重

3. BOINC(volunteer Computing)
- 比较老的架构，特点是节点不可靠，支持checkpoint，支持replication。但是会有高延迟，批处理等缺点
```
Central Server  |Scheduler  |Clients (volunteers)
```

- DAG(directed acyclic graph): 有向无环图，用于表达一个任务链

## Ray Archi
- 关键词：高性能计算，支持python，pandas，numpy，分布式异步调用，内存调度，加速机器学习
- Dask：也是python分布式数据平台，底层是分布式计算架构

## idea
- 针对straggler问题，能不能根据设备的算力，动态的调节任务的大小（或者是多个client分配到的任务规模不一样）

## docu
- Ray: A Distributed Framework for Emerging AI Applications
- Hoplite: Efficient and Fault-Tolerant Collective Communication for Task-Based Distributed Systems

是否是node-aware，任务分配是什么逻辑，任务调度是什么逻辑
1. node fail，是否可以设置一个bound时间，如果超时就重新分配，
2. increase fault-tolerance，add bound time，node recovery
- 看一下做什么方向，然后再详细的介绍一下
在high level的下面，增加什么，跟tool关系不大