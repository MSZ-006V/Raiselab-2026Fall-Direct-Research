## 问题
1. 编译提示没有'em', 'empy'
    - 由于装了anaconda，默认使用conda环境，但是ros2中指定的是系统的python，需要暂时deactive
    ```
    export PATH=/usr/bin:$PATH
    which python3
    # 应输出：/usr/bin/python3

    source /opt/ros/galactic/setup.bash
    cd ~/projects/pdm/picas/ros2-picas
    rm -rf build install log
    colcon build --allow-overriding rclcpp --cmake-args -DPICAS=TRUE
    ```
2. 运行example发现没有按照优先级调度
    - 需要按照githuh仓库教程，设置当前用户可以设置的最大优先级，如下。然后重启，再重新运行
    ```
    yfu  hard    rtprio  99
    yfu  soft    rtprio  99
    ```
3. example执行
    ```
    source install/setup.bash
    
    ./build/picas_example/sstest   # 新编写的
    ./build/picas_example/example   # single-threaded version
    ./build/picas_example_mt/example_mt   # multi-threaded version
    ```
4. 编写新的类后，在src/rclcpp中编写cpp文件，在include/rclcpp编写hpp文件，随后修改CMakeList注册新的模块