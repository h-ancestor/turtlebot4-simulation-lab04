# Bài thực hành 04 – TurtleBot 4 Simulation, SLAM Toolbox và Nav2

Workspace ROS 2 Humble cho mô phỏng TurtleBot 4 Lite trong world `warehouse`, khảo sát bằng SLAM Toolbox, lưu map và dùng AMCL/Nav2. Source tự viết nằm trong package `turtlebot4_lab`; simulator, SLAM và Nav2 dựa trên các package TurtleBot 4/ROS 2 có sẵn.

## Nội dung

- `turtlebot4_lab/`: package ROS 2 gồm launch simulation/SLAM/localization/Nav2, cấu hình SLAM, world warehouse và node survey waypoint.
- `maps/map.yaml`, `maps/map.pgm`: map occupancy được lưu từ `/map` do SLAM Toolbox phát ra. Kích thước 560×472, độ phân giải 0.05 m/cell, origin `[-13.1, -11.9, 0]`. Bản sao trong `turtlebot4_lab/maps/` được cài theo package để giá trị mặc định của localization launch hoạt động.
- `docs/map_week04.png`: ảnh raster của map đã lưu.
- `docs/tuan05_rviz_navigation_yolo.png`: ảnh RViz từ lần kiểm chứng tích hợp ở tuần 5, dùng cùng world/map; không phải ảnh quay quá trình mapping.


## Trạng thái và giới hạn đã biết

- Gazebo warehouse/TurtleBot 4 Lite đã chạy; `/scan`, `/odom` và `/cmd_vel` đã được quan sát trong ROS graph. Có thể dùng teleop để điều khiển robot.
- SLAM Toolbox đã lưu occupancy map 560×472. Tuyến khảo sát tập trung khu vực aisle trung tâm, chưa phủ phần lớn/toàn bộ warehouse; một số vùng có nhiễu và biên chưa thật mượt. Xem `docs/map_week04.png` 
- Map đã được nạp trong cùng world cho AMCL/Nav2. Trong lượt kiểm chứng tích hợp về sau, ba Nav2 goal được gửi bằng chương trình của bài tuần 5 và cả ba báo `SUCCEEDED`; pose AMCL cuối khoảng `(-4.74, 6.05)` so với goal `(-5, 6)`. Đây là kiểm chứng Nav2/AMCL trên map


## Môi trường và cài dependency

Ubuntu 22.04 + ROS 2 Humble Desktop:

```bash
sudo apt update
sudo apt install ros-humble-desktop ros-humble-turtlebot4-simulator \
  ros-humble-turtlebot4-navigation ros-humble-slam-toolbox \
  ros-humble-teleop-twist-keyboard
```

## Build

Tại thư mục gốc repository:

```bash
source /opt/ros/humble/setup.bash
colcon build --base-paths turtlebot4_lab --symlink-install
source install/setup.bash
```

## Chạy simulation và kiểm tra sensor

```bash
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch turtlebot4_lab simulation.launch.py world:=warehouse model:=lite rviz:=true
```

Ở terminal khác, kiểm tra topic:

```bash
ros2 topic list
ros2 topic hz /scan
ros2 topic echo /odom --once
```

## Mapping (SLAM Toolbox)

Chạy SLAM:

```bash
ros2 launch turtlebot4_lab slam.launch.py use_sim_time:=true
```

Điều khiển bằng bàn phím ở terminal khác:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Khảo sát chậm, đi các aisle và quay về khu vực đã đi để hạn chế drift. Quan sát Fixed Frame `map`, LaserScan và OccupancyGrid trong RViz. Khi map hiển thị hợp lý, lưu:

```bash
ros2 run nav2_map_server map_saver_cli -f maps/map
```

## Localization và Nav2 trên map đã lưu

Giữ simulation chạy. Khởi chạy localization với map trong repo:

```bash
ros2 launch turtlebot4_lab localization.launch.py \
  map:="$(pwd)/maps/map.yaml" use_sim_time:=true
```

Sau khi xác nhận initial pose khớp vị trí spawn và AMCL hội tụ, mở Nav2:

```bash
ros2 launch turtlebot4_lab navigation.launch.py use_sim_time:=true
```

Đặt `2D Pose Estimate` trong RViz nếu cần; sau đó dùng `Nav2 Goal` cho từng vị trí trong vùng đã khảo sát. Theo dõi scan/costmap để bảo đảm đường đi an toàn.
## Topics chính

- `/scan` — LiDAR (`sensor_msgs/msg/LaserScan`).
- `/odom` — odometry (`nav_msgs/msg/Odometry`).
- `/cmd_vel` — lệnh vận tốc tới robot.
- `/map` — occupancy grid từ SLAM hoặc map server.
- `/initialpose`, `/amcl_pose` — pose ban đầu và pose AMCL.
- `/tf`, `/tf_static` — transform giữa `map`, `odom`, base và sensor.


