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
2. 