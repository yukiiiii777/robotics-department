# 03 · OpenFlex 资源汇总

> 厂商（成都长数科技 / OpenArmX 生态）官方资源一站式索引。链接失效时优先回 GitHub 仓库找最新入口。

## 官方资源

| 资源 | 链接 | 说明 |
| --- | --- | --- |
| **GitHub 源码** | <https://github.com/OpenFleX-Wheeled-Humanoid/OpenFleX> | 全球首款 27 自由度开源轮式双臂机器人，持续更新；含基础/高级中文说明文档、安全指南、一键安装脚本 |
| **官方技术文档站** | <http://docs.openarmx.com/> | 整机方案、系统包、购买配置等权威资料 |
| **开箱教程（飞书）** | <https://ix81a8yrmke.feishu.cn/wiki/OEgiw44kwiIqlIksSOgc6CDVnde> | 《OpenFleX_设备木箱开箱教程_V1.0.docx》 |
| **启动系列教程（B站）** | <https://b23.tv/xJIrJYF> | 合集·OpenFleX 启动系列教程 |
| **电气及机械文档（飞书）** | <https://ix81a8yrmke.feishu.cn/wiki/ZNnEwP1ZDiAKowkWrQ4cu4GUnbf> | 🔒 密码见部门内部群（含密码不入公开库） |
| **项目架构设计报告** | <https://www.kdocs.cn/l/ctZCGTwcscGB> | 《OpenFleX_项目架构设计报告_V1.0（持续更新）》 |

## 相关 GitHub 仓库

| 仓库 | 用途 |
| --- | --- |
| [OpenFleX-Wheeled-Humanoid/OpenFleX](https://github.com/OpenFleX-Wheeled-Humanoid/OpenFleX) | 主仓库：驱动、地图、模型、ROS 2 工作空间 |
| [OpenFleX-Wheeled-Humanoid/openflex_mujoco](https://github.com/OpenFleX-Wheeled-Humanoid/openflex_mujoco) | MuJoCo 全身模型（底盘+升降+双臂+头部），`python -m mujoco.viewer openflex_mujoco.xml` 即可把玩 |

## 主仓库结构速查

```
openflex_all/
├── openflex_drivers/            # 驱动包（.deb、.whl）
├── openflex_maps/               # 地图文件
├── openflex_models/             # 模型/权重文件
└── openflex_ws/                 # ROS 2 工作空间
    └── src/
        ├── OpenFlex/            # 项目文档与安装脚本
        ├── openflex_integrated/ # 全身集成（bringup/description/GUI/manager）
        ├── openflex_chassis/    # 底盘子系统（swerve 驱动、Livox 驱动、导航）
        ├── openflex_lift_slide/ # 升降柱子系统（CANopen）
        ├── openflex_armx/       # 双臂子系统（bringup/hardware/description/teleop_vr）
        ├── openflex_head/       # 头部子系统（bringup/hardware/description）
        ├── openflex_vr_bridge/  # VR 位姿桥接（UDP → ROS 2 话题）
        └── openflex_EXO/        # 外骨骼遥操作
```

## 技术栈速览

- **系统**：Ubuntu 22.04 + ROS 2 Humble，`ros2_control` 统一管理全身关节（100Hz）
- **遥操**：PICO 4 Ultra / Meta Quest 全身遥操（UDP 5100 端口桥接，约 90Hz）；支持外骨骼遥操
- **规划**：MoveIt 2 双臂规划 + 全身碰撞检测
- **导航**：FAST-LIO2 + PGO 建图、ICP 定位、Nav2 + 自研 NMPC 局部控制器
- **VLA**：LeRobot 框架，4× RealSense 相机同步采集，ACT 训练推理（HDF5）；支持 Pi0.5 / X-VLA
- **RL**：MuJoCo、Isaac Sim 强化学习
- **感知**：头部双目 RGB / D435i、腕部 D405 ×2、底盘 D435 + Livox MID360

## 上手建议路径

1. **官方配置主机** → 直接读主仓库 `openflex-documentation(Basic).md`（中文版 `openflex-说明文档(基础)-CN.md`）
2. **自备主机** → 先跑 `install_openflex_drivers_and_build.sh` 一键装驱动编译，再按子系统逐个调（底盘 → 升降 → 双臂 → 头部 → 全身）
3. **安全第一** → 动真机前必读 `Safety_Guide.md` / `安全指南.md`

## 许可协议

OpenFleX 开源代码与模型采用 **CC BY-NC-SA 4.0**（非商业、署名、相同方式共享）——部门教学科研使用没问题，商用需联系厂商。
