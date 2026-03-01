```
1. priority_queue，目前只是通信，未来可能是全方面
2. distributed large workload，比如对于对于cpu heavy workload，每个node有不同算力，depend on system status
   1. 比如目前cpu util是60%，那么就切分成3份，减小对于单个cpu uti的增加
   2. 比如目前cpu util是30%，那么就只切成2份，保证locality，减少消息的传输
3. memory_aware(locality), workload_aware(任务), priority_aware
```




4. - big pic：大的workload（可能需要拆分）
5. 在multitask envir，
- 目前：第一步首先完成fix priority
- 第二步：memory priority，workload——aware，或者说对于某个对象所属于的task，是否在某个chain里面，然后可以怎么保证
- 其实就是把ray做成一个类似ros的东西
- 最终实现
\
- 下周目前：画图，分为high p，median p，low p，画图，对比fixed prior和default的东西
- 