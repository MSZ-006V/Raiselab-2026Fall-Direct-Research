## Ray Architecture
- 开源框架，用于AI和Python有用的
- Ray可以自动处理：编排（管理分布式系统的组件），进度安排（协调任务的执行时间和地点），容错性（确保任务完成，即使存在故障点），自动扩缩容（动态需求调整分配的资源数量）
- ray计算框架由三层组成
    - Ray AI Lib
    - Ray Core：Python通用分布式计算库
    - Ray Cluster：一组连接到公共Ray头节点的节点
- Ray有五个原生库
    - Data：在Train，Tune，prediction中提供数据相关
    - Train：有容错能力的分布式多节点多核模型训练
    - Tune：hyper parameter调优，用以优化模型性能
    - Serve：可扩展，可编程服务
    - RLlib：RL相关
## Ray Core
- 构建和运行分布式应用程序的组件，可以将Python或Java函数和类转换为分布式无状态任务或有状态Actor
- 支持传递参数，对象（存储在object_store中）
- task：允许函数在不同的工作进程上异步执行，这些异步Ray函数就是task
- actor（actor模型）：支持类，Actor 本质上是一个有状态的工作进程（或服务）
- 对象objects：ray会把object存储在集群的任何位置（分布式共享内存 object store）

## RayKube - Ray k8s extension
- K8s相关插件（KubeRay）
- 每个Ray集群由一个头节点Pod和一组工作节点Pod组成，支持自动扩缩容（根据工作负载需求调整Ray集群大小，支持添加或移除Ray Pod，支持异构计算节点（包括GPU））
- 相关CRD：RayCluster，RayJob，RayService
1. Ray Cluster
2. Ray Job
   - 独立的应用程序，包含来源自同一脚本的Ray任务，对象和Actor的集合
3. Ray Service
   - 

## Concepts
- Ray支持将计算任务从单机串行执行扩展到分布式异步执行。比如借助远程API（remote API），可以将计算任务提交到计算集群，动态的将任务拆分成更细粒度的计算任务，然后并行执行任务，异步的收集计算结果
- Ray头节点（Head Node）：头节点与其他的工作节点系统，但是还运行负责集群管理的单例进程（Autoscaler，GCS，Ray driver processes）
- Ray工作节点（Worker Node）：工作节点不允许任何头节点管理进程，只运行Ray任务和Actor中的用户代码
- Autoscaler（自动扩缩容）：运行在主Pod的sidecar模式，自动扩缩容机制仅响应任务和 Actor 的资源请求，而不响应应用程序指标或物理资源利用率

1. 看Ray，怎么优化，有没有人做相关的插件，有什么研究的点
2. 看有没有相关的论文，描述这方面的设计
3. 总之就是看ray的bottleneck在哪, 然后如何下手进行研究
- 架构视角，组件怎么分工，谁控制谁
- 执行路径视角：一次task/actor调度，从提交到执行经过了什么
- bottleneck怎么找：时间，通信，内存，调度决策

q1：ray的分布式内存，怎么保证调度的快速性