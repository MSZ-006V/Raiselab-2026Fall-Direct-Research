## Benchmark experiment
### Metrics
1. Task Completion Time
    - 单任务Latency（P50，P95，P99）
    - makespan（所有任务完成时间）
2. Throughput(每秒完成Task数，在saturation下的最大QPS)
3. CPU Utilization, Memory Utilization, Object store Utilization
4. Scheduling Overhead(Task queued time), Object transfer time

### Scenario
1. 纯CPU-bound task
- 计算密集，无数据依赖，简单计算任务
- 测试重点：default与Spread在多节点下的throughput，是否出现单节点overload，是否有hotspot
2. 数据共享（locality heavy）
- 场景：设置一个大object（1GB），所有task都需要读
- 测试重点：object transfer次数，网络带宽，latency
3. 混合workload
- 场景：一部分短task（80%），一部分长task（20%）
- 测试重点：cluster imbalance，是否有scheduling starvation
4. fragmentation test
- 场景：task需要0.5CPU，但是node有8CPUc