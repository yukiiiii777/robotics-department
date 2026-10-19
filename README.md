# 人工智能实验室 · 机器人部门

> 在真实机器人上，学会让机器「感知 → 决策 → 执行」。

本仓库是**人工智能学院人工智能实验室机器人部门**的新成员入口，包含部门介绍、设备资料、新生学习路线、VLA 数据闭环与导航开发的实操索引，以及全部官方资料链接。

---

## 目录

- [一、部门简介](#一部门简介)
- [二、我们的设备](#二我们的设备)
- [三、学习方向](#三学习方向)
- [四、新生学习路线（12 周）](#四新生学习路线12-周)
- [五、核心技术地图](#五核心技术地图)
- [六、资料与链接](#六资料与链接)
- [七、仓库结构](#七仓库结构)
- [八、常用命令速查](#八常用命令速查)
- [九、协作规范](#九协作规范)

---

## 一、部门简介

机器人部门以真实机器人为载体，围绕「移动操作 + 具身智能数据闭环」，培养从底盘导航、机械臂控制到 VLA 模型的全栈机器人开发能力。

我们相信具身智能不能只停留在论文和仿真里——部门让每位成员在真机上完成「感知 → 决策 → 执行」的完整闭环开发：

**VR 遥操 → 数据采集 → VLA 训练 → 真机推理**

## 二、我们的设备

### OpenFlex Ultra 轮式双臂机器人（成都长数科技）

开源生态整机平台，集移动底盘、升降、双臂、多视角视觉与高算力于一体。

| 项目 | 参数 |
| --- | --- |
| 整机自由度 | 27 DOF（头部 ×2 + 双臂 ×16 + 升降 ×1 + 四轮四转底盘 ×8） |
| 整机尺寸 | 高约 1700 mm，重约 120 kg |
| 双臂能力 | 单臂额定负载约 5 kg、峰值约 12 kg，臂展 714 mm |
| 升降机构 | 行程约 700 mm，垂直负载 50 kg |
| 视觉配置 | 头部 RealSense D435 + 腕部 D405 ×2 + 底盘 D435 |
| 激光雷达 | Livox MID360 3D 雷达 |
| 控制主机 | AMD 锐龙 9 Pro / 16 GB / 1 TB |
| 算力扩展 | **自配技嘉 RTX 4090（24 GB）** |
| 软件栈 | Ubuntu 22.04 + ROS 2 Humble + MoveIt 2 + Nav2 + LeRobot VLA 工作流 |
| 模型支持 | Pi0 / Pi0.5 / X-VLA 等 |

### PiPER 轻量六轴机械臂（松灵科技）

| 项目 | 参数 |
| --- | --- |
| 自由度 | 6 |
| 有效负载 | 1.5 kg |
| 工作半径 | 626 mm |
| 重复定位精度 | ±0.1 mm |
| 本体重量 | 4.2 kg |
| 开发支持 | ROS 1 / ROS 2、Python SDK、拖动示教、Gazebo / Isaac Sim |

**定位**：机械臂开发的入门练手台，适合先在小臂上跑通「运动学 → 抓取 → 数采 → 部署」全流程。

### PICO 4 Ultra VR 一体机

| 项目 | 参数 |
| --- | --- |
| 处理器 | 骁龙 XR2 Gen 2 |
| 存储 | 12 GB + 256 GB |
| 显示 | 双目 4K+（单眼 2160×2160）@ 90 Hz |
| 透视 | 双目 3200 万像素彩色透视 + iToF 深度 |
| 交互 | 手势追踪 + 手柄 |

**用途**：第一视角遥操作示教，把人的操作直接转化为 VLA 训练数据。

## 三、学习方向

| 方向 | 学什么 | 主要载体 |
| --- | --- | --- |
| 机械臂 | 正逆运动学、轨迹规划、MoveIt 2、双臂协同、抓取策略 | OpenFlex 双臂 / PiPER |
| VLA | VR 遥操示教、多视角数据集构建、LeRobot 工作流、Pi0.5 / ACT 训练与推理 | OpenFlex + PICO + 4090 |
| 底盘导航 | 激光 SLAM（MID360）、ICP 定位、Nav2、NMPC 控制、代价地图 | OpenFlex 底盘 |
| 通用基础 | Linux、ROS 2、Python / C++、Git、MuJoCo / Isaac Sim | 全体必修 |

## 四、新生学习路线（12 周）

> 目标：12 周内从零基础到「能在真机上完成一次完整的数据闭环」。
> 每周 6~8 小时，按阶段推进；每个阶段结束有明确产出物。

### 阶段一：地基（第 1–2 周）

| 周 | 学什么 | 产出 |
| --- | --- | --- |
| 1 | Linux 基础命令、文件系统、SSH、环境变量；Python 基础（函数/类/文件读写）；Git 基本操作（clone / add / commit / push） | 在自己的电脑上配好开发环境，能 clone 本仓库并提交一次 PR |
| 2 | ROS 2 Humble 核心概念：节点、话题、服务、参数、launch、TF2、RViz；`ros2 topic echo / hz` 等调试命令 | 跑通 turtlesim；能画出「话题 → 节点」连接图并解释 |

**验收**：能独立解释「一个 ROS 2 节点如何订阅话题并发布结果」。

### 阶段二：仿真与手臂（第 3–5 周）

| 周 | 学什么 | 产出 |
| --- | --- | --- |
| 3 | URDF / xacro 机器人建模；MoveIt 2 运动规划框架（SRDF、规划组、IK） | 在 RViz 里让 PiPER 模型完成一次点到点运动规划 |
| 4 | PiPER Python SDK：关节控制、末端位姿控制、拖动示教；CAN 通信基础 | 用脚本让真机 PiPER 完成「移动到指定点 → 闭合夹爪」 |
| 5 | MuJoCo / Isaac Sim 基础；仿真与真机的差异（动力学、延迟、噪声） | 在仿真中复现上面的抓取动作 |

**验收**：能在仿真和真机上完成同一个抓取动作，并说出两者差异。

### 阶段三：真机与数据（第 6–8 周）

| 周 | 学什么 | 产出 |
| --- | --- | --- |
| 6 | OpenFlex 开箱、上电、控制中心、KAN 通讯；VR 设备配对；安全规范（急停、速度限制、场地净空） | 独立完成一次开机自检与 VR 连接 |
| 7 | VR 遥操作全身控制；LeRobot 录制流程（见 [docs/vla-pipeline.md](docs/vla-pipeline.md)） | 采到第一份 30 episode 的数据集 |
| 8 | 数据校验：LeRobot Studio 可视化，剔除模糊/偏视/过快的 episode；数据规范 | 一份清洗后的高质量数据集 + 采集记录 |

**验收**：能独立走完「开服务 → 录制 → 校验」全流程，数据集可直接用于训练。

### 阶段四：训练与部署（第 9–11 周）

| 周 | 学什么 | 产出 |
| --- | --- | --- |
| 9 | VLA 原理：VLM 骨干 + 动作头；Pi0.5 / ACT 的区别；过拟合与数据量的关系 | 能讲清楚 VLA 的输入输出与训练数据格式 |
| 10 | 云端训练：A100 40G 环境部署、`lerobot-train` 参数含义（见 [04 · 手册精要](docs/04-OpenFlex-PI0.5数采训练推理手册精要.md)） | 完成一次 Pi0.5 微调训练 |
| 11 | 本地部署与离线推理：权重放置、`HF_HUB_OFFLINE=1`、缓存清理；失败案例分析 | 真机上跑通一次推理，并记录成功率 |

**验收**：模型能在真机上完成指定任务，能分析失败原因（数据问题 / 模型问题 / 环境问题）。

### 阶段五：导航与项目（第 12 周起）

| 周 | 学什么 | 产出 |
| --- | --- | --- |
| 12 | 建图 → 3D 转 2D 地图 → ICP 定位 → Nav2 导航（见 [05 · 导航精要](docs/05-OpenFlex导航开发指南精要.md)） | 在实验室内完成一次自主导航到指定点 |
| 之后 | 加入项目组，参与「导航 + 操作」分层任务；对接竞赛与课题 | 项目 Demo / 技术报告 |

### 分方向深耕（第 8 周后按兴趣选择）

- **机械臂方向**：双臂协同、力控、抓取位姿生成、MoveIt 2 插件开发
- **VLA 方向**：数据增广、多任务数据混合、ACT / Pi0.5 / X-VLA 对比实验
- **导航方向**：NMPC 调参、代价地图优化、建图精度、VLA 与 Nav2 分层调度

## 五、核心技术地图

### VLA 数据闭环（部门主线）

```
VR 遥操示教 → LeRobot 录制 → 数据校验 → 云端训练（Pi0.5）→ 本地部署推理 → 失败再采数据
```

- 环境：Ubuntu 22.04 + ROS 2 Humble + LeRobot 0.4.x
- 训练：A100 40G，`--policy.type=pi05`，`--steps=30000`，bf16 + 梯度检查点
- 推理：`HF_HUB_OFFLINE=1` 强制本地权重，推理前必清缓存
- 详细命令与参数：[04 · OpenFlex-PI0.5 手册精要](docs/04-OpenFlex-PI0.5数采训练推理手册精要.md)

### 导航链路

```
建图（mapping.launch.py）→ 3D map.pcd → pcd_to_nav2_map → 2D map.yaml
定位（ICP + safe_odom_mux）→ map→base_link
导航（Nav2：SmacPlanner2D + NMPC + 代价地图 + 速度平滑）
```

- 关键：Nav2 的 `odom_topic` 必须指向 `/odom_safe`；避障输入是 `/scan`（由点云转激光）
- 推荐架构：**VLA 不学长距离导航**，导航交给 Nav2，VLA 负责到位后的精细操作
- 详细参数与排障：[05 · OpenFlex 导航开发指南精要](docs/05-OpenFlex导航开发指南精要.md)

## 六、资料与链接

| 资料 | 链接 | 说明 |
| --- | --- | --- |
| OpenFleX 设备木箱开箱教程 V1.0 | https://ix81a8yrmke.feishu.cn/wiki/OEgiw44kwiIqlIksSOgc6CDVnde | 飞书文档，开箱必读 |
| OpenFleX 启动系列教程（视频） | https://b23.tv/xJIrJYF | B 站合集，跟着做一遍 |
| OpenFleX GitHub 源码 | https://github.com/OpenFleX-Wheeled-Humanoid/OpenFleX | ROS 2 运控、VR / 外骨骼遥操、LeRobot-VLA |
| 电气及机械文档（飞书） | https://ix81a8yrmke.feishu.cn/wiki/ZNnEwP1ZDiAKowkWrQ4cu4GUnbf | 内部资料，密码见部门群 |
| 项目架构设计报告 V1.0 | https://www.kdocs.cn/l/ctZCGTwcscGB | 金山文档，持续更新 |
| OpenArmX 官方文档 | http://docs.openarmx.com | 整机方案、控制器、软件包 |
| Pi0.5 数采-训练-推理手册 | [docs/04 手册精要](docs/04-OpenFlex-PI0.5数采训练推理手册精要.md) + `docs/pdf/` 原件 | 部门内部资料 |
| 导航开发指南 | [docs/05 导航精要](docs/05-OpenFlex导航开发指南精要.md) + `docs/pdf/` 原件 | 部门内部资料 |
| LeRobot Studio | https://lerobot.studio | 数据集在线可视化校验 |
| 松灵 PiPER 官网 | https://www.agilex.ai | 机械臂参数与 SDK |

## 七、仓库结构

```
.
├── README.md                                   # 本文件：部门总览 + 学习路线 + 资料索引
├── docs/
│   ├── 00-部门介绍.md                          # 部门定位、设备、方向
│   ├── 01-新生学习路线.md                      # 从零到真机闭环的分步清单
│   ├── 02-VLA入门讲义.md                       # VLA 原理与模型架构
│   ├── 03-OpenFlex资源汇总.md                  # 官方资源与仓库索引
│   ├── 04-OpenFlex-PI0.5数采训练推理手册精要.md # 数采 → 训练 → 推理全流程
│   ├── 05-OpenFlex导航开发指南精要.md          # Nav2 + NMPC 导航开发
│   ├── 06-招新演讲稿.md                        # 2-3 分钟部门介绍稿
│   └── pdf/                                    # 厂商原始 PDF 手册
└── assets/
    └── poster.png                              # 招新海报
```

**按顺序读**：00 → 01 → 02 →（上手时查 04 / 05）→ 03 找链接。

## 八、常用命令速查

### 环境初始化（每个新终端都要执行）

```bash
cd $HOME/openflex_all/openflex_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export OPENFLEX_WS=$HOME/openflex_all/openflex_ws
export OPENFLEX_VLA_DIR=$OPENFLEX_WS/src/openflex_vla
export OPENFLEX_SCHEMA=$OPENFLEX_VLA_DIR/config/schemas/chassis_lio_act_vel_base.yaml
export OPENFLEX_CAMERA_CONFIG=$OPENFLEX_VLA_DIR/config/cameras/cameras_config_30fps.yaml
```

### 启动录制三件套

```bash
# 终端 1：相机 + 全身 VLA 录制服务
ros2 launch openarmx_lerobot wholebody_vla_record.launch.py \
  enable_base_camera:=true camera_config:=$OPENFLEX_CAMERA_CONFIG

# 终端 2：VR 遥操作
ros2 launch openarmx_lerobot wholebody_vr_teleop.launch.py

# 终端 3：录制主程序
./src/openflex_vla/lerobot_scripts/lerobot-record-openflex.sh \
  --schema_config=$OPENFLEX_SCHEMA --camera_config=$OPENFLEX_CAMERA_CONFIG \
  --dataset.repo_id=YOUR_NAME/pick_cube --dataset.num_episodes=31 \
  --dataset.fps=30 --dataset.single_task="grasp_black_cube_to_blue_box"
```

### 导航

```bash
ros2 launch swerve_navigation full_system.launch.py \
  pcd_path:="$HOME/MID_360_nav/openflex_maps/3d/$MAP_NAME/map.pcd" \
  map_yaml:="$HOME/MID_360_nav/openflex_maps/2d/$MAP_NAME/map.yaml" \
  initial_pose:="[0.0,0.0,0.0,0.0,0.0,0.0]" use_rviz:=true
```

### 健康检查

```bash
ros2 run tf2_ros tf2_echo map base_link   # 定位链路
ros2 topic hz /odom_safe                  # 安全里程计（应 50 Hz）
ros2 topic hz /scan                       # 避障输入
ros2 topic echo --once /cmd_vel_safe      # 速度指令
```

## 九、协作规范

1. **真机使用需预约**，操作前确认急停位置与场地净空；两人在场方可运行。
2. **数据命名**：`任务名_日期_采集人`，例：`pick_cube_20261020_yuu`。
3. **分支规范**：`main` 只放稳定内容；个人改动走 `feat/xxx` 或 `docs/xxx` 分支，PR 合并。
4. **文档优先**：踩过的坑写进 `docs/`，不写进聊天记录。
5. **安全第一**：任何会移动机械臂/底盘的操作，先在仿真或低速模式下验证。

---

*人工智能学院 · 人工智能实验室 · 机器人部门*
