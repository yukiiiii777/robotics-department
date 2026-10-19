# 人工智能实验室 · 机器人部门

> 在真实机器人上，学会让机器「感知 → 决策 → 执行」。

本仓库是机器人部门的**资料中枢**：设备说明、新生学习路线、VLA 入门讲义、OpenFlex 官方资源与操作手册精要，全部收录于此，持续更新。

## 📦 我们的设备

| 设备 | 说明 |
| --- | --- |
| **OpenFlex Ultra 轮式双臂机器人**（成都长数科技） | 全身 27 自由度（双臂×16 + 头部×2 + 升降×1 + 四轮四转底盘×8）；四路 RGB-D 相机 + Livox MID360 激光雷达；自配技嘉 RTX 4090（24G）算力包；预装 Ubuntu 22.04 + ROS 2 Humble、VR 全身遥操、MoveIt 2、Nav2、LeRobot VLA 工作流 |
| **PiPER 轻量六轴机械臂**（松灵科技） | 6 自由度，1.5 kg 负载，626 mm 工作半径，±0.1 mm 重复定位精度；ROS 2 / Python SDK / 拖动示教，机械臂开发入门练手台 |
| **PICO 4 Ultra VR 一体机** | 第一视角遥操示教与 VLA 数据采集——你的操作就是训练数据 |

## 📚 文档索引

| 文档 | 内容 |
| --- | --- |
| [部门介绍](docs/00-部门介绍.md) | 部门定位、设备亮点、学习方向、培养路径 |
| [新生学习路线](docs/01-新生学习路线.md) | 从零基础到真机部署的完整成长路径 |
| [VLA 入门讲义](docs/02-VLA入门讲义.md) | Vision-Language-Action 是什么、怎么训、代表模型 |
| [OpenFlex 资源汇总](docs/03-OpenFlex资源汇总.md) | 开箱教程 / B站视频 / GitHub 源码 / 电气机械文档 / 架构报告 |
| [Pi0.5 数采-训练-推理手册精要](docs/04-OpenFlex-PI0.5数采训练推理手册精要.md) | 官方操作手册提炼：环境初始化、三终端启动、录制、训练、部署推理 |
| [OpenFlex 导航开发指南精要](docs/05-OpenFlex导航开发指南精要.md) | 建图→定位→导航三层链路、Nav2/NMPC 架构、调参排查 |

官方原版 PDF 手册见 [`docs/pdf/`](docs/pdf/) 目录。

## 🔗 核心资源速达

- **GitHub 源码**：<https://github.com/OpenFleX-Wheeled-Humanoid/OpenFleX>（全球首款 27 自由度开源轮式双臂机器人，持续更新）
- **官方技术文档**：<http://docs.openarmx.com/>
- **启动系列教程（B站）**：<https://b23.tv/xJIrJYF>
- **开箱教程（飞书）**：<https://ix81a8yrmke.feishu.cn/wiki/OEgiw44kwiIqlIksSOgc6CDVnde>
- **电气及机械文档（飞书，密码见内部群）**：<https://ix81a8yrmke.feishu.cn/wiki/ZNnEwP1ZDiAKowkWrQ4cu4GUnbf>
- **项目架构设计报告（金山文档）**：<https://www.kdocs.cn/l/ctZCGTwcscGB>

## 🧭 学习方向

- 🦾 **机械臂**：运动学 / 轨迹规划 / 双臂协同抓取
- 🧠 **VLA**：视觉-语言-动作大模型，具身智能的「大脑」
- 🚗 **底盘导航**：SLAM 建图 / 自主定位 / 路径规划
- 🧰 **通用基础**：Linux / ROS 2 / Python·C++ / 仿真

---

*人工智能学院 · 人工智能实验室 · 机器人部门*
