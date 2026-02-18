1. 画图，跑完benchmark
2. node的任务task队列，目前可能是fifo或者best effort，尝试是否能够新建一个queue
   - 新的queue是高速通道，小任务就可以走高速通道（计算workload小，或者mem占用少。但是具体怎么评价一个小任务呢？有什么threshhold相关的）
   - 每一个node里面，任务队列是什么，执行的顺序是什么？我可以修改任务执行的顺序吗
   - benchmark：heavy data locality，
3. workload-aware algorithm，