```
1. priority_queue，目前只是通信，未来可能是全方面
2. distributed large workload，比如对于对于cpu heavy workload，每个node有不同算力，depend on system status
   1. 比如目前cpu util是60%，那么就切分成3份，减小对于单个cpu uti的增加
   2. 比如目前cpu util是30%，那么就只切成2份，保证locality，减少消息的传输
3. memory_aware(locality), workload_aware(任务), priority_aware
```

priority_queue(ray architecture)
位置：src/ray/object_manager/push_manager.cc/ScheduleRemainingPushes()
作用场景：当node1上的task要执行时，发现需求的某个大对象不存在本地，触发了raylet调度，该请求通过本地节点的raylet传到了另外一个节点的raylet（假设node2），node2的节点通过读取本地的object store，得到对象，通过push_manager发回
push_manager发送原理：对于大对象，切分为chunk进行发送，原生使用round_robin方法
想要实现的效果：在push_manager发回时，通过透传的某个优先级，优先把对应任务需要的对象先发回（非抢占）
实现步骤
1. 一路透传从Python层面（在定义task的时候，传入一个参数）
2. 该参数伴随着raylet请求从node1传送到node2，node2获取对象后，给push_manager进行发送
3. 那么目前push_manager内会有多个等待发送的大对象，根据不同对象对应任务的优先级，按照优先级从高到低进行发送

初步实现
1. 首先修改ScheduleRemainingPushes()内，修改round_robin算法为SJF算法（即按照对象切分后的chunk数量）进行选择下一个要发送的对象（按照越小的对象发送优先级越高进行决定）
2. 修改后，重新编译，启动

1. 如何编写对应的benchmark实验?
2. 修改后如何编译，打包，应用？