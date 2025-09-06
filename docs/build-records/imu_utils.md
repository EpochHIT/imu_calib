添加头文件：
#include <fstream>
#include <iomanip>
#include <string>
#include <vector>
循环索引由 int 改为 std::size_t，消除编译警告。

重新编译：
cd /home/cwkj/catkin_imu_ws
catkin_make -DCeres_DIR=/usr/lib/aarch64-linux-gnu/cmake/Ceres -DCERES_DIR=/usr/lib/aarch64-linux-gnu/cmake/Ceres