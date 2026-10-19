# 04 · OpenFlex-PI0.5 数据采集-训练-推理 手册精要

> 提炼自官方《OpenFlex-PI0.5机器人数据采集-训练-推理完整操作手册》（原版 PDF 见 `docs/pdf/`）。
> 环境：Ubuntu + ROS 2 Humble。命令与参数保持官方原样，实际部署以工程内最新配置为准。

## 0. 全流程总览

```
控制中心开KAN通讯 → 3终端环境初始化 → 终端1相机录制服务 / 终端2 VR遥操 / 终端3录制主程序
→ 批量录制episodes → LeRobot Studio筛数据 → 云端A100训练 → 权重传回本地 → 离线推理验证
```

## 1. 前置准备

- OpenFlex 本体 + VR 遥操设备供电正常；主机、机器人、VR **同一局域网同网段**
- OpenFlex 控制中心手动开启 **KAN 通讯服务**，确认无通讯报错

## 2. 三终端统一环境初始化

**每个新终端都要执行一遍**（终端关闭后环境变量不保留）：

```bash
cd $HOME/openflex_all/openflex_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export OPENFLEX_WS=$HOME/openflex_all/openflex_ws
export OPENFLEX_VLA_DIR=$OPENFLEX_WS/src/openflex_vla
export OPENFLEX_SCHEMA=$OPENFLEX_VLA_DIR/config/schemas/chassis_lio_act_vel_base.yaml
export OPENFLEX_CAMERA_CONFIG=$OPENFLEX_VLA_DIR/config/cameras/cameras_config_30fps.yaml
```

校验：`echo $OPENFLEX_WS` 输出路径即正常。

## 3. 分终端启动服务

**终端一**——相机采集 + 全身 VLA 录制基础服务：

```bash
ros2 launch openarmx_lerobot wholebody_vla_record.launch.py \
  enable_base_camera:=true \
  camera_config:=$OPENFLEX_CAMERA_CONFIG
```

**终端二**——VR 遥操服务（观察日志，无超时/IP 报错即连接成功）：

```bash
ros2 launch openarmx_lerobot wholebody_vr_teleop.launch.py
```

**终端三**——保持初始化状态，跑录制脚本。

## 4. 数据集录制

```bash
export OPENFLEX_KEY_DEBOUNCE_S=2.0
export OPENFLEX_KEY_MIN_SEGMENT_S=3.0

./src/openflex_vla/lerobot_scripts/lerobot-record-openflex.sh \
  --schema_config=$OPENFLEX_SCHEMA \
  --camera_config=$OPENFLEX_CAMERA_CONFIG \
  --robot.type=openarmx_follower_ros2 \
  --robot.skip_send_action=false \
  --teleop.type=openarmx_leader_ros2 \
  --teleop.ros2.command_timeout_s=0.0 \
  --display_data=true \
  --dataset.repo_id=YOUR_NAME/pick_cube \
  --dataset.push_to_hub=false \
  --dataset.num_episodes=31 \
  --dataset.fps=30 \
  --dataset.episode_time_s=70 \
  --dataset.reset_time_s=50 \
  --dataset.single_task="grasp_black_cube_to_blue_box" \
  --resume=false
```

### 键盘控制（录制中）

| 按键 | 作用 |
| --- | --- |
| → 右箭头 | 提前结束当前单集并保存 |
| ← 左箭头 | 作废当前片段，自动重录本集 |

### 录制规范（直接决定数据质量）

- **全程慢速**：禁止快速移动、快速夹取，动作缓慢平稳
- **起始姿态统一**：每次录制前校准，夹爪相机正对目标、画面居中，杜绝偏视/盲区数据

### 关键参数

| 参数 | 含义 |
| --- | --- |
| `num_episodes=31` | 批量录制 31 组任务片段 |
| `fps=30` | 与相机配置匹配，帧率必须一致 |
| `episode_time_s=70` | 单集最大时长 |
| `reset_time_s=50` | 每集完成后的复位等待 |
| `single_task` | 任务定义，如「黑色方块抓取至蓝色盒子」 |
| `push_to_hub=false` | 仅本地保存不上传 |

## 5. 数据校验（LeRobot Studio）

数据集本地路径：`$HOME/.cache/huggingface/lerobot/YOUR_NAME/<dataset>/`

打开 <https://lerobot.studio/> → 授权本地目录 → Local Directory → 选择数据集根目录 → 在线预览视频帧、校验动作轨迹，**筛掉模糊/偏视/过快的无效 episode**，只留高质量数据训练。

## 6. 云端训练

### 硬件硬性要求

- **Pi0.5 训练最低 40G 显存 A100**，低于此配置会显存溢出/训练中断，禁止低显存卡硬跑
- 部门本地 RTX 4090（24G）适合训 **ACT** 等轻量模型；Pi0.5 建议云端租卡

### 服务器环境部署要点

- 四套独立 Conda 环境：基础 / PI0 / PI0.5 / XVLA，**禁止混用**
- 必备文件：`/opt/ros/humble/setup.bash`、`openflex_ws/install/setup.bash`、`lerobot-0.4.3.zip`、`transformers-fix-lerobot_openpi.zip`（部署后 sha256 校验）
- 适配 CUDA 12.6 / Python 3.10 / Torch 2.7.1，清华源加速
- 装完必须跑环境校验脚本（lerobot 版本、torch、CUDA、OpenFlex 插件、transformers 补丁全生效）

### 训练启动脚本

```bash
export DATASET_NAME="YOUR_NAME/pick_cube"
export DATASET_ROOT="$HOME/.cache/huggingface/lerobot"
export PI05_BASE_MODEL="你的PI0.5预训练模型路径"
export OUTPUT_DIR="$HOME/train_output/pi05_pick_black_screw_all_v2_run3"

torchrun --nproc_per_node=1 "$(which lerobot-train)" \
  --job_name=pi05_pick_black_screw_all_v2_run3 \
  --dataset.repo_id="$DATASET_NAME" \
  --dataset.root="$DATASET_ROOT" \
  --dataset.video_backend=pyav \
  --policy.type=pi05 \
  --policy.pretrained_path="$PI05_BASE_MODEL" \
  --policy.push_to_hub=false \
  --policy.dtype=bfloat16 \
  --policy.device=cuda \
  --policy.train_expert_only=true \
  --policy.gradient_checkpointing=true \
  --policy.normalization_mapping='{"STATE":"QUANTILES","ACTION":"QUANTILES"}' \
  --policy.optimizer_lr=1e-05 \
  --policy.scheduler_decay_lr=1e-06 \
  --policy.scheduler_warmup_steps=1000 \
  --batch_size=8 \
  --num_workers=8 \
  --steps=30000 \
  --save_freq=2000 \
  --log_freq=20 \
  --eval_freq=4000 \
  --seed=1000 \
  --wandb.enable=false \
  --output_dir="$OUTPUT_DIR" \
  2>&1 | tee "$HOME/pi05_${DATASET_NAME}_run3.txt"
```

### 训练核心规范

- 全程独占 A100 40G，禁止多任务抢显存
- 仅用 `lerobot_pi05` 独立环境
- 仅用 LeRobot Studio 校验后的高质量数据
- 固定随机种子保证可复现

## 7. 部署与离线推理

**模型权重上传至本地固定目录**（路径全局统一，禁止修改）：

```
/home/openflex/openflex_all/openflex_models/pick_cube_all_pi05_02/pretrained_model
```

**推理终端**（激活 PI0.5 推理环境 + 完整初始化）：

```bash
source /home/openflex/anaconda3/etc/profile.d/conda.sh
conda activate lerobot_pi05
cd $HOME/openflex_all/openflex_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export OPENFLEX_WS=$HOME/openflex_all/openflex_ws
export OPENFLEX_VLA_DIR=$OPENFLEX_WS/src/openflex_vla
export OPENFLEX_SCHEMA=$OPENFLEX_VLA_DIR/config/schemas/chassis_lio_act_vel_base.yaml
export OPENFLEX_CAMERA_CONFIG=$OPENFLEX_VLA_DIR/config/cameras/cameras_config_30fps.yaml
```

**离线推理指令**（`HF_HUB_OFFLINE=1` 强制加载本地权重、禁止联网拉取）：

```bash
HF_HUB_OFFLINE=1 lerobot-record \
  --robot.type=openflex_follower_ros2 \
  --robot.skip_send_action=false \
  --robot.schema_config=$OPENFLEX_SCHEMA \
  --robot.camera_config=$OPENFLEX_CAMERA_CONFIG \
  --dataset.repo_id=local/eval_hi2 \
  --dataset.single_task="hi" \
  --dataset.num_episodes=10 \
  --dataset.episode_time_s=120 \
  --dataset.push_to_hub=false \
  --display_data=true \
  --policy.path="/home/openflex/openflex_all/openflex_models/pick_cube_all_pi05_02/pretrained_model"
```

**推理缓存必清**（否则数据覆盖 / episode 冲突 / 推理失效），每次推理前二选一：

```bash
rm -rf /home/openflex/.cache/huggingface/lerobot/local/eval_hi2
# 或带时间戳备份
mv /home/openflex/.cache/huggingface/lerobot/local/eval_hi2 \
   /home/openflex/.cache/huggingface/lerobot/local/eval_hi2_bak_$(date +%Y%m%d%H%M%S)
```

## 8. 常见问题速查

| 现象 | 处理 |
| --- | --- |
| 通讯失败 | 检查 KAN 通讯是否开启、设备 IP 是否同网段，重启服务或重连 VR |
| 环境变量失效 | 每个新终端都要重新执行全套初始化 |
| 相机无数据 | 确认相机配置路径正确、`enable_base_camera` 已开启、硬件连接正常 |
| 显存溢出 | Pi0.5 换 A100 40G；或本地改训 ACT |
| 推理异常 | 先清缓存，确认 `HF_HUB_OFFLINE=1`，模型路径用固定路径 |
