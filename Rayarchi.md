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
- task：允许函数在不同的工作进程上异步执行，这些异步Ray函数就是task（主要为了任务并行，主要针对无状态）
  - stateless，fine-grained load balancing，support for object locality
- actor（actor模型）：支持类，Actor 本质上是一个有状态的工作进程（或服务）（主要使得ray支持有状态计算）
  - 封装了一组方法，这些方法可以被远程调用，并且串行执行，一次方法执行类似于任务执行，
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

## Computation Model
- dynamic task graph（动态任务图）：在该模型中，当远程函数或 actor 方法的输入可用时，系统会自动触发它们的执行
- 可以把一个函数（或者是类）的一段执行过程抽象为一个图
## Architecture
- application layer由三种进程组成
  - Driver（执行用户程序的进程）：一个python进程，负责提交task或actor，不执行他们。driver在head node上
  - Worker（无状态进程，执行由Driver或其他Worker调用的任务，由系统自动启动并分配任务。当一个远程函数被声明时，该函数会被自动发布到所有 worker。Worker 以串行方式执行任务，并且在任务之间不维护任何本地状态）
  - Actor（有状态进程，仅在被调用时执行其对外暴露的方法。串行执行，但每一次方法执行都依赖于前一次方法执行所产生的状态），actor只会在创建的时候被调度一次，放置在某个node上，后续不会存在migration
- System Layer
  - GCS（Global Control Store）：维护系统的全部控制状态，本质是pub-sub的key-val存储系统，具有shard功能，在每个shard上使用chain replication保证容错，同时GCS存储所有对象元数据（object metadata）
  - Distributed Scheduler：Ray使用两级层次化调度器（由全局调度器和每个节点的本地调度器组成）。为了避免全局调度器成为瓶颈，在某个节点上创建的任务会首先提交给该节点的本地调度器。本地调度器会优先尝试在本地调度任务，除非该节点已经过载（即本地任务队列超过预定义阈值），或者无法满足任务的资源需求（例如缺少 GPU）如果本地调度器决定不在本地执行任务，它会将任务转发给全局调度器。由于该调度策略优先在调度层次结构的叶子节点（即本地节点）上调度任务，因此我们称其为自底向上的调度器
    - 全局调度器策略：利用任务的资源约束来决策
        1. 找出所有满足该任务条件的节点
        2. 选择预计等待时间最短的节点lowest estimated waiting time
            - 时间=（1）任务在该节点上排队等待的时间（即队列长度乘以平均任务执行时间）（2）传输任务远程输入所需的时间（即远程输入的总大小除以平均带宽）
    - 通过Heartbeat获取节点的任务队列长度和资源可用性，通过GCS获取任务输入对象的位置和大小
    - 使用simple exponential averaging（指数加权）来估计任务平均执行时间和平均传输带宽
    - 如果GCS变成瓶颈，会启动副本（通过GCS共享信息）来保证扩展性
  - In memory distributed object store（内存，分布式存储）：存储每个任务（无状态计算）的输入和输出，在同一个节点上运行的任务之间支持零拷贝数据共享
    - 如果某个任务的输入数据不在本地，会在执行前把输入复制到本地，输出同样写入本地。只在本地内存读写数据，数据始终在内存。LRU策略逐出到磁盘。对象存储只支持不可变数据。不支持分布式对象，对象必须能被放入单个节点
  - 实现：GCS每个分片使用Redis存储，本地调度和全局调度都是事件驱动的单线程。传输大对象时会拆分对象使用多个TCP连接传输
- Lineage（血缘信息）：一个对象或结果是通过哪些计算步骤，依赖了什么输入产生的（主要用于容错，当某个节点或对象丢失时，可以根据lineage重新执行，把对象算回来
- Object metadata：对象的信息（对象ID，对象存在哪，对象大小，是否可用，引用计数）

## Target
1. 修改global scheduler的调度策略（比如节点选择的策略）
   - background：每个任务或者参与者都有指定的资源需求，节点可以处于以下的状态之一
   - 可行
       - 可用：节点拥有所需的资源，并且现在这些资源是空闲的
       - 不可用：节点拥有所需的资源，但是现在这些资源被占据
   - 不可行：节点不具备所需的资源（如仅有CPU的节点无法执行GPU任务）
   - 已有策略
       - Default：Ray 将任务或 Actor 调度到前 k 个节点上。节点首先按已调度任务或 Actor 的节点排序（以实现局部性），然后按资源利用率低的节点排序（以实现负载均衡）。在前 k 个节点中，随机选择节点。在实现方面，Ray 会根据集群中每个节点的逻辑资源利用率计算其得分。如果利用率低于阈值（由操作系统环境变量控制RAY_scheduler_spread_threshold，默认值为 0.5），则得分为 0；否则，得分即为资源利用率本身（得分 1 表示节点已完全利用）。Ray 会从得分最低的前 k 个节点中随机选择最佳节点进行调度。该阈值取集群中节点数与环境变量k取值的最大值。默认值为节点总数的 20%。RAY_scheduler_top_k_fractionRAY_scheduler_top_k_absolute
       - SPREAD：尝试把任务或参与者分散到可用的节点上，忽略约束，避免任务在节点上集中
       - Placement Group Scheduling：schedule the task or actor to where the placement group is located
       - Node Affinity Scheduling：允许将任务或 Actor 调度到由其节点 ID 指定的特定节点上。可以强制让某个Task或Actor在某个节点运行（若节点没有可用资源将持续等待）
         - 防止被智能调度器优化，创建时必须知道节点ID
       - Locality-Aware Scheduling（Only for task）：优先选择在本地拥有大量任务参数的节点，此优先级高于DEFAULT策略
2. SPREAD会牺牲部分data locality，是否可以综合他们来进行调度？
    - score = α * load_balance + β * data_locality
4. Ray的actor churn问题：假设Actor被频繁创建然后又销毁

idea:
1. 可以2，4：waiting time bound，先再了解一下SPREAD的具体逻辑（和default的区别，是不是会完全为了load-balancing而服务，而不管任务的具体情况）
2. 第四点可以考虑ddl aware，不做hard ddl，可以尽量做成soft ddl（针对task）
3. 像任务打包的时候附带的参数（比如num_cpu一样，携带一部分的参数）
4. 了解SPREAD的逻辑，查看如何修改代码
5. 需要跑preliminary的实验
6. 每一个node里面，任务队列是什么，执行的顺序是什么？我可以修改任务执行的顺序吗
7. 任务所需资源不够的情况下（task或actor）
  - 默认：无限等待pending
  - method1：
    - task：设置一个max_retry=3, 等待时间指数上升
    - actor：尝试对node上的actor进行迁移，腾出足够空间容纳新的actor
    ```
    network layer

    actor3 25%
    node1: actor1 70%, actor2 20%
    node2: actor4 80%

    actor2 (node1 -> node2)

    node1: actor1 70%, 
    node2: actor4 80%, actor2 20%
    ```
  - method2:
    - task, actor max_retry = 3, 等待时间指数上升
    - max_retry, timeout = 3s

1. Default: 尽量把任务“挤”到已有节点上，减少节点扩散，提高局部性。按优先级进行排序（资源局部性，减少跨节点对象传输，提高Cache/Object reuse, 减少节点冷启动和Actor分散
  - 步骤：1. 过滤出满足资源需求的节点 2. 优先选择：已经运行同类task的节点，资源刚好够用的节点，数据本地性高的节点 3. 选择第一个节点
  - 问题：不会主动均匀分布！
2. Spread：刻意把任务分散到不同节点上，最大化节点级并行
  - 步骤：1. 过滤出满足资源需求的节点 2. 优先选择当前任务数最少的节点, 尽量选择“还没跑过这个 task / actor 的节点” 
  - 忽略data locality

node1 -> 2 cpu, 1 gpu ---> 不停
node2 -> 16 cpu, 8 gpu

3. 若没有足够资源，任务会处于pending（等待资源）的状态，不会拆分任务，ray会假设这是一个暂时的资源短缺问题，会不断重试调度（无限等待）
4. task & actor
  - task：无限并行，无状态，一次性任务，纯函数式的计算请求
    - task失败只需要分配到其他节点重试即可
    ```
    func(int a, int b): return a + b;
    ```
  - actor：带状态、长期存在的计算实体，所有对它的调用都是单线程串行执行
    - actor失败需要ray处理状态丢失，重建等问题
    ```
    class counter
      private cnt;
      add{
        cnt++;
      }

    node1.add()

    node3.add()

    // 读取cnt = 2；
    ```
- 

5. 相关文件
- ray/src/ray/raylet/scheduling/scheduling_policy.cc
- ray/src/ray/raylet/scheduling/cluster_resource_scheduler.cc
- ray/src/ray/raylet/scheduling/locality_aware_scheduling.cc

6. next week plan：
  - 跑preliminary的实验
  - 先把修改raylet的层面跑通

```
# script.py
import ray

@ray.remote
def hello_world():
    [cpu=1, gpu=2, waiting_timeout=4]
    return "hello world"

# Automatically connect to the running Ray cluster.
ray.init()
print(ray.get(hello_world.remote()))
```

（可行性）
- high prior：不想被network阻塞，做一个high prior queue，对于特定object，
  - send 之前有一个队列，变成一个priority queue
  - object transmission
  - Node1: queue: a, b, c, d -> d, a, b, c
- Node interconnection
  - find which channel(default is gRPC?)
  - high priority gRPC channel, best-effort gRPC channel
  - gRPC 是否long-live

- benchmark test
  - envir：小data
  - light workload
  - heavy workload，stress test，