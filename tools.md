1. 更改swap大小，部署conda再试试，不用docker
2. 目前进度，尝试部署，然后跑实验，记录一些数据
实验可以是固定个数据集，然后运行一次，查看消耗的资源等等，这个可以用nsys来记录查看
环境
CUDA 11.4
L4T R35.2.1 with Jetpack 5.0.2

## 实验
- exp1：baseline
- epx2: 把llama切为两半，比如m1和m2，通过cgroup关闭swap，然后比如切后，细分模型就可以整个放入ram中了，当每个阶段的模型运行完后，结果放入pinned mem，然后加载第二个阶段的，读取pinned mem，继续执行，对比时间
    - 可以设置一个pinned mem，因为m1的结果是要输入m2的，m1的结果就可以放置于这部分


jetson_clocks：锁定时钟频率，保证实验数据的稳定
cgroup： 是 Linux 内核提供的机制，用于 资源管理和限制，主要用于控制 Jetson 设备上的 CPU、内存、I/O 和 GPU 资源的分配
tegrastats ： 记录CPU温度，功耗等等参数
NVIDIA Nsight Systems：https://developer.nvidia.com/nsight-systems性能工具，查看时钟频率，各种CPUGPU占用参数等
```
nsys profile .... -o a.nsys-rep
```

参考资料
https://github.com/BestAnHongjun/LMDeploy-Jetson