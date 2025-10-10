## ROS2
- [ROS2 Docu](https://docs.ros.org/en/galactic/index.html)
- architecture:  
![alt text](image.png)
1. Client Layer: 针对不同语言（C++/Python）提供对应API调用，底层Client Library为C实现
2. DDS Layer: 中间层，解绑Client Layer与DDS Implementation Layer
3. DDS Implementation Layer: 底层模块库，包括序列化，通信，传输等
- Node: A node may publish data to any number of topics and simultaneously have subscriptions to any number of topics.  
![alt text](image-1.png)
- Topic: 
- Sub/Pub Module: 
- Task: a callback unit, can be execute by a Executor
    - types:
        - a subscription callback
        - timer response
        - request
### Executor & Callback groups
- Executor: 轮询 / 等待来自 middleware（或内部队列）的可用工作，然后把相应回调交给线程去执行
    - 包括SingleThreadExecutor, MultiThreadExecutor两种
    - 调度粒度：Executor从底层队列/唤醒点拿到某个任务并在某个线程内执行；它不会在回调内部做抢占（即回调运行期间不会被 Executor 强行打断）
    - 支持自定义Executor
    - 不是系统调度器，不关心Callback调度相关
- Callback Groups(Multi-Thead Exection): 可以对不同的Callback进行分组，实现更细粒度的Multi-Thread Executor控制
    - Mutually Exclusive Callback Group: only one callback will be executed by a SingleThreadedExecutor
    - Reentrant Callback Group: 
    - Callbacks belonging to different callback groups (of any type) can always be executed parallel to each other
- Scheduling机制
    - default: FIFO
    - message and events will be queued on the lower layer of the stack, the executor only know whether there are any messages for a certain topic or not, Executor uses this information to process the messages(使用Round-Robin)
    - 对于上面这些问题，rclcpp Waitset & rclc Executor部分解决（如支持callback执行顺序的细粒度控制）
### Paper
- [ECRTS: Response-Time Analysis of ROS 2 Processing Chains Under Reservation-Based Scheduling](https://drops.dagstuhl.de/storage/00lipics/lipics-vol133-ecrts2019/LIPIcs.ECRTS.2019.6/LIPIcs.ECRTS.2019.6.pdf)
    - 针对ROS2 Executor scheduling

## Self Driving Pipeline
1. Perception
    - Camera, Lidar: parsing raw sensor data into meaningful representations of the environment
2. Localization
    - The system knows its position on the map
3. Map
    - Modeling environment and generating raster maps
4. Planning
    - Plan the next course of action
5. Control -> Decision
#### ROS2 in self-driving
- Perception, localization, map can be divided into different ROS node, communicate with topic/service/action (for single client <-> single server)
- ROS Quality of Service: can control message reliability, delay, history cache, timeout, etc
- Challenge: 
    - Network delay: 