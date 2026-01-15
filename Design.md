## Design - v1
- 场景：single client, single server, single executor
- Feature: 优先级调度，Priority Management
- 系统中同一时间存在一个Executor与一个ReadySet，ReadySet中缓存各种任务，每个任务在进入系统时，会被分配一个优先级，然后进入ReadySet等待
- Polling Point: 查询底层，更新ReadySet。当以下两种情况发生时触发Polling Point
    - Executor完成当前任务后，将触发一次（同时对ReadySet中任务按照优先级排序）
    - Executor如果空闲，则使用固定interval触发Polling Point
- Executor每次从ReadySet中取出一个优先级最大的task进行执行
- 当有任务到达，计算出该任务的Network Delay，随后得到该任务剩余的Computation Budget
- 根据剩余的计算Budget，选择不同版本的Task任务

## rclcpp可修改点
- executor：single_threaded_executor, multi_threaded_executor
- strategies: allocator_memory_strategy
- 可以在ros2-picas-example/picas_example/src/example.cpp里面，比如设置5个不同的任务，然后根据budget来智能计算一个优先级
- 计算公式修改了，slack time = ci - (cur - timestamp)
- 任务属性
    - 当前时刻: t_cur
    - 任务DDL: Di
    - 剩余执行时间：Ci
    - slack time = Di - t_cur - Ci
    - 由slack time来决定Priority and Execution Version

## QoS计算框架
- QoS评价指标：throughtput，latency，accuracy
- 对于整个系统（in time window 100ms）
    - throughtput
        $$throughtput = min(1, \frac{T_{cur}}{A_{cur}})$$
        - T_cur: 在time_window内完成的所有任务（包括miss deadline和no miss deadline）
        - A_cur：在时间窗口内到达的任务
        - time_window: 统计窗口（1s）
        - 范围：0 - 1
    - data freshness: 在当前这个time window内完成的所有chain的data freshness的加权组合
    - $$QoS（系统级）= w_{th}Q_{th} + w_{df}Q_{df}$$
- 对于单个任务
    - 参数：Di(deadline), Ai(arrival time), Ci(execution time), Fi(finish time)
    - accuracy：分为两种 deadline accuracy，temporal accuracy
        - deadline accuracy：acc_d = 1(Fi <= Di), acc_d = 0(Fi > Di)
        - temporal accuracy: acc_t = max(0, 1 - (Fi - Ai) / (Di - Ai))
            - Fi越靠近Di，acc_t越低；Fi理Di越远，acc_t越大
            - 区间 0 <= acc_t <= 1
        $$
        acc_t(\tau_i) = \lambda*acc_d(i) + (1 - \lambda)*acc_t(i)
        $$
- 对于一条链（chain）
    - Accuray（系统级,在一个时间）(也可以转换为根据不同类型的任务，拥有不同的权重)
    $$
    Acc_{sys} = \frac{1}{N}\sum^{N}_{i = 1}acc(\tau_i)
    $$
    - data freshness
    $$
    Data\ Freshness = min(1, 1-\frac{lag}{lag_{max}}) \\
    lag = t_{comple} - t_{first} \\ 
    lag_{max} = P_{max} + L_{net} + L_{exec} + L_{queue} + \epsilon \\
    P_{max} = 最长period \\
    L_{net} = avg\ 网络延迟 \\
    L_{exec} = avg\ 执行时间 \\
    L_{queue} = avg\ scheduling\ pending
    $$
