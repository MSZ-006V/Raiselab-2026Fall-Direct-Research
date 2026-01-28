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

## RayKube - Ray k8s extension
- K8s相关插件（KubeRay）
- 每个Ray集群由一个头节点Pod和一组工作节点Pod组成，支持自动扩缩容（根据工作负载需求调整Ray集群大小，支持添加或移除Ray Pod，支持异构计算节点（包括GPU））
- 相关CRD：RayCluster，RayJob，RayService
1. Ray Cluster
2. Ray Job
3. Ray Service

## Concepts
- Ray支持将计算任务从单机串行执行扩展到分布式异步执行。比如借助远程API（remote API），可以将计算任务提交到计算集群，动态的将任务拆分成更细粒度的计算任务，然后并行执行任务，异步的收集计算结果