1. 画图，跑完benchmark
2. node的任务task队列，目前可能是fifo或者best effort，尝试是否能够新建一个queue
   - 新的queue是高速通道，小任务就可以走高速通道（计算workload小，或者mem占用少。但是具体怎么评价一个小任务呢？有什么threshhold相关的）
   - 每一个node里面，任务队列是什么，执行的顺序是什么？我可以修改任务执行的顺序吗
   - benchmark：heavy data locality，
3. workload-aware algorithm
   - 通过某个模型来针对任务进行预测，在raylet设置一个queue，进行排队
   - 

```
Ray 的 Object 大小分类与传输机制：

小对象 (通常 <= 100KB)： Ray 会直接把它们塞进 gRPC 的元数据（Control Plane）里，直接在进程间传。

大对象 (大于 100KB)： 存入 Plasma 共享内存。当跨节点传输时，由底层的 Object Manager 通过网络拉取（Data Plane）。

网络通道 (Node Interconnection) 是怎么样的？

Default is gRPC? 是的。Ray 底层的节点间通信（包括 Task 派发、状态同步、Object 传输）严重依赖 gRPC。

gRPC 是否 long-live（长连接）？ 是的。 Raylet 之间为了追求极低的延迟，会维护长期的 gRPC 连接池（Connection Pool）。它们不会每次传数据都重新握手建连，那是不可接受的。

你的 Idea 解决的真实痛点 (队头阻塞)：
由于连接是复用的（long-lived），如果 Node A 正在向 Node B 发送一个 5GB 的模型权重，底层的 TCP buffer 和 gRPC stream 会被完全占满。此时，如果有一个 1MB 的高优先级控制信号（比如“停止训练”）需要传过去，它只能在发送队列（Send Queue）里傻等。

你的方案落地机制：

Send Queue 优先级重排 (d, a, b, c)： 在 Object Manager 将数据块推入 gRPC 流之前，劫持队列，根据对象的大小或用户标记的 Tag 进行优先级重排。

Multi-Channel (双通道隔离)： 仅仅重排队列可能不够，因为正在传的 5GB 大文件已经进入网卡缓冲了。你的构想“High priority gRPC channel + Best-effort gRPC channel”非常精准。为小对象/高优对象单独开辟一条物理或逻辑连接，是彻底解决 HoL Blocking 的正解。
```

```
Ray 的官方 API 文档（Python 层）是绝对不会讲这个的。你需要看 Ray Architecture Whitepaper (v2)，以及直接翻看 GitHub 上的 C++ 源码。

代码路径寻找（基于 ray-project/ray 仓库）：

路径： src/ray/raylet/scheduling/

核心类： ClusterTaskManager（负责管理队列和调度）

在这个文件里（如 cluster_task_manager.cc 和 .h），你可以看到真实存在的队列定义：

tasks_to_schedule_：等待调度的任务。

infeasible_tasks_：资源暂时不满足（比如要 100 个 CPU，但全集群最大节点只有 64 个）的任务队列。

调度逻辑循环： 你会看到 ScheduleAndDispatchTasks() 函数，这就是遍历 Pending Queue 并尝试将任务派发给 Worker 的地方。

如果你要实现你的 Object 传输优先级，你需要看的不是 Task 队列，而是 Object Manager 的代码：

路径： src/ray/object_manager/

核心类： object_manager.cc 和底层的 RPC 发送逻辑。
```