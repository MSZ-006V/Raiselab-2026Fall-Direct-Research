## Design - v1
- 场景：single client, single server, single executor
- Feature: 优先级调度，Priority Management
- 系统中同一时间存在一个Executor与一个ReadySet，ReadySet中缓存各种任务，每个任务在进入系统时，会被分配一个优先级，然后进入ReadySet等待
- Polling Point: 查询底层，更新ReadySet。当以下两种情况发生时触发Polling Point
    - Executor完成当前任务后，将触发一次（同时对ReadySet中任务按照优先级排序）
    - Executor如果空闲，则使用固定interval触发Polling Point
- Executor每次从ReadySet中取出一个优先级最大的task进行执行