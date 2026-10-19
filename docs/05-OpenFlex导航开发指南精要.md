# 05 · OpenFlex 导航开发指南 精要

> 提炼自官方《OpenFlex 导航开发指南》（原版 PDF 见 `docs/pdf/`），基于《使用说明书（高级篇）》导航章编写。
> 平台：ROS 2 Humble，工作空间 `~/openflex_all/openflex_ws`。命令与参数保持官方原样。

## 1. 导航系统三层链路

```
建图（离线）──► 定位（在线）──► 导航（在线）

建图: mapping.launch.py → 3D map.pcd → pcd_to_nav2_map → 2D map.yaml/map.pgm
定位: icp_registration（点云配准）+ safe_odom_mux（安全里程计）
导航: Nav2（full_system.launch.py）= 全局规划 + NMPC局部控制 + 代价地图 + 避障
```

**运行时数据流**：Nav2 Goal → BT Navigator（行为树）→ SmacPlanner2D 全局路径 + NMPC 局部跟踪 + Smoother 平滑 + Behavior Server 恢复行为 → Velocity Smoother（加减速限制）→ `/cmd_vel_safe` → swerve 底盘控制。

**避障输入**：`/fastlio2/body_cloud`（3D 点云）→ `pointcloud_to_laserscan` → `/scan`（2D）→ 局部/全局代价地图障碍层。

**定位与里程计供给**：

| 数据/模块 | 作用 | 频率 |
| --- | --- | --- |
| map.pcd | ICP 全局定位用 3D 点云地图 | 静态 |
| map.yaml | Nav2 路径规划用 2D 栅格地图 | 静态 |
| /fastlio2/lio_odom | LIO/LIVO 短期里程计 | 10 Hz |
| /odom | 轮式里程计（底盘输出） | 50 Hz |
| safe_odom_mux | LIO 与轮式 odom 安全切换 | 50 Hz |
| icp_registration | 实时点云与 3D 地图配准 | ~100 Hz |

最终复合变换：`map → base_link = map → odom × odom → base_link`，兼顾高频平滑与全局准确。

## 2. 前置：建图与转图

### 2.1 建图（生成 3D 地图）

```bash
cd ~/openflex_all/openflex_ws
source install/setup.bash
export MAP_NAME=your_map
mkdir -p "$HOME/MID_360_nav/openflex_maps/3d/$MAP_NAME"
ros2 launch swerve_navigation mapping.launch.py
```

RViz 关注 `/pgo/global_cloud` 与 `/fastlio2/body_cloud`，完成后保存：

```bash
ros2 service call /pgo/save_maps interface/srv/SaveMaps \
  "{file_path: '$HOME/MID_360_nav/openflex_maps/3d/$MAP_NAME', save_patches: true}"
```

### 2.2 3D 转 2D 导航地图

```bash
mkdir -p "$HOME/MID_360_nav/openflex_maps/2d/$MAP_NAME"
ros2 run pgo pcd_to_nav2_map \
  --pcd "$HOME/MID_360_nav/openflex_maps/3d/$MAP_NAME/map.pcd" \
  --out-dir "$HOME/MID_360_nav/openflex_maps/2d/$MAP_NAME" \
  --poses "$HOME/MID_360_nav/openflex_maps/3d/$MAP_NAME/poses.txt" \
  --resolution 0.05 \
  --z-min 0.00 --z-max 2.30 \
  --obstacle-z-min 0.10 --obstacle-z-max 1.40 \
  --inflation-radius 0.10
```

### 2.3 地图效果调试速查

| 现象 | 调整 |
| --- | --- |
| 地面被当成障碍 | 提高 `--obstacle-z-min`（0.10 → 0.15） |
| 高处结构投影成障碍 | 降低 `--obstacle-z-max` |
| 地图噪点多 | `--sor-mean-k 20 --sor-stddev 1.0` |
| 地图太细碎 | `--resolution` 放宽到 0.10 |
| 障碍边界太贴机器人 | 增大 `--inflation-radius` |

## 3. 快速开始：启动完整导航

```bash
export MAP_NAME=your_map
ros2 launch swerve_navigation full_system.launch.py \
  pcd_path:="$HOME/MID_360_nav/openflex_maps/3d/$MAP_NAME/map.pcd" \
  map_yaml:="$HOME/MID_360_nav/openflex_maps/2d/$MAP_NAME/map.yaml" \
  initial_pose:="[0.0,0.0,0.0,0.0,0.0,0.0]" \
  use_rviz:=true
```

定位链路（icp_registration + safe_odom_mux）会自动拉起，无需单独启动。

**RViz 操作五步**：① 确认地图与机器人模型 → ② `2D Pose Estimate` 点实际位姿（初始位姿不准必做）→ ③ 观察实时点云与地图重合 → ④ `Nav2 Goal` 点目标 → ⑤ 观察全局/局部路径与 `/cmd_vel_safe`。

**运行检查**：

```bash
ros2 run tf2_ros tf2_echo map base_link   # 全局定位链路
ros2 topic hz /odom_safe                  # 安全里程计
ros2 topic hz /scan                       # 避障输入
ros2 topic echo --once /cmd_vel_safe      # 实际速度指令
```

## 4. Nav2 架构要点

配置文件：`src/openflex_chassis/navigation_layer/swerve_navigation/config/nav2_params.yaml`

| 服务器 | 功能 | 插件 |
| --- | --- | --- |
| BT Navigator | 行为树导航决策 | Nav2 行为树节点集 |
| Controller Server | 局部路径跟踪 | `nmpc_controller::NmpcController`（自研） |
| Planner Server | 全局路径规划 | `nav2_smac_planner/SmacPlanner2D` |
| Smoother Server | 路径平滑 | SimpleSmoother |
| Behavior Server | 恢复行为 | Spin / Backup / Wait |
| Velocity Smoother | 速度平滑 | VelocitySmoother |

**开发要点**：`odom_topic` 指向 **`/odom_safe`**（而非原始 `/odom`），Nav2 只认安全融合后的里程计——这是三层定位架构的落地。

## 5. NMPC 局部控制器（开发重点）

OpenFlex 导航区别于标准差速机器人的核心。

- **模型**：底盘抽象为可全向平移的二阶非线性模型（state = [px, py, θ, vx, vy, ω]，control = [ax, ay, α]）
- **求解器**：acados/CasADi，预测时域 N=20、Tf=2.0s，SQP_RTI 实时迭代，每控制周期（20Hz）执行一次
- **代码**：`nmpc_controller/src/nmpc_controller.cpp`；求解器生成 `script/generate_solver.py`

**舵轮 native 模式（四转四驱关键优势）**：`swerve_native_mode=true` 时车头始终朝向终点方向，底盘横移贴住路径，避免蛇形摆动。

**内置局部 A\* 避障**：路径前方 2.5m 内代价 ≥0.40 判定阻塞 → odom costmap 上 8 邻域 A* 绕行 → 平滑后拼回局部路径。

## 6. 代价地图要点

| | 局部代价地图 | 全局代价地图 |
| --- | --- | --- |
| 参考帧 | odom | map |
| 更新频率 | 10 Hz | 5 Hz |
| 尺寸 | 5×5 m 滚动窗口 | 全图 |
| 分辨率 | 0.05 m | 0.05 m |
| 层结构 | 障碍层（/scan）+ 膨胀层 | 静态层（map_server）+ 障碍层 + 膨胀层 |

- 机器人足迹：0.74 m × 0.62 m 矩形（含安全裕量）
- 膨胀半径 0.30 m，代价衰减因子 3.0

## 7. 故障排查清单（按「输入→处理→输出」顺序）

| 层级 | 检查项 | 正常标准 |
| --- | --- | --- |
| 传感器 | `ping 192.168.1.173` | MID360 有响应 |
| 传感器 | `ros2 topic hz /livox/lidar` | ~10 Hz |
| 传感器 | `ros2 topic hz /livox/imu` | ~200 Hz |
| 定位 | `tf2_echo map base_link` | 持续输出 |
| 定位 | RViz 点云与地图重合 | 误差 <20~30cm、朝向 <10° |
| 里程计 | `ros2 topic hz /odom_safe` | 50 Hz 稳定 |
| 避障 | `ros2 topic hz /scan` | 稳定无空洞 |
| 控制 | `echo --once /cmd_vel_safe` | 速度值合理 |

**常见故障**：

| 现象 | 处理 |
| --- | --- |
| 初始定位失败 | RViz `2D Pose Estimate` 给更接近真实的初值 |
| 定位漂移/跳变 | 检查实时点云与 3D 地图重合度 |
| 长走廊定位不稳 | 降速，让雷达看到几何特征 |
| 重定位失败 | 检查 `sc_data/` 目录（ScanContext 数据库）是否存在 |
| 机器人不动但无报错 | 查 `/odom_safe` 是否有数据、`initial_pose` 是否给准 |
| 频繁触发恢复行为 | 膨胀半径过大或 footprint 偏大导致「无路可走」 |

## 8. 进阶：导航与 VLA 分层（推荐架构）

```
语言任务
 → 高层任务规划：选择目标区域和子任务
 → Nav2/NMPC：长程导航、避障、全局路径
 → VLA：到位后的底盘微调、升降、头部观察、双臂操作
 → ros2_control：关节、电机、舵轮底层闭环
```

## 9. 关键文件索引

| 用途 | 路径 |
| --- | --- |
| 完整导航 Launch | `swerve_navigation/launch/full_system.launch.py` |
| Nav2 参数 | `swerve_navigation/config/nav2_params.yaml` |
| NMPC 控制器 | `nmpc_controller/src/nmpc_controller.cpp` |
| NMPC 求解器生成 | `nmpc_controller/script/generate_solver.py` |
| 安全里程计复用器 | `swerve_navigation/scripts/safe_odom_mux.py` |
| ICP 定位 | `mapping_localization_layer/icp_registration/` |
| 点云转激光 | `navigation_layer/pointcloud_to_laserscan/` |
| 3D 地图转 2D | `mapping_localization_layer/pgo/src/pcd_to_nav2_map.cpp` |
