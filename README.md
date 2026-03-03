
# 🚗 ROS 2 Sim-to-Real: Autonomous Navigation & RL Control
> 基于 ROS 2 (Jazzy) 与 Gazebo 的移动机器人自主导航与端到端强化学习部署平台

![ROS2](https://img.shields.io/badge/ROS_2-Jazzy-22314E?logo=ros)
![Gazebo](https://img.shields.io/badge/Gazebo-Harmonic-FF8A00)
![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?logo=pytorch)
![Stable-Baselines3](https://img.shields.io/badge/Algorithm-PPO-0052CC)

## 📖 项目简介
本项目旨在构建一个完整的 **"感知-决策-控制"** 具身智能闭环系统。利用机械运动学知识从零构建机器人 URDF 物理模型，打通 Gazebo 物理引擎与 ROS 2 TF 坐标树的异步通信链路，并分别实现了基于**有限状态机(FSM)的规则避障**与**基于 PPO 算法的端到端自动驾驶**。

本项目展示了从传统的几何逻辑控制，向现代数据驱动(Data-Driven)与 Sim-to-Real 部署的完整工程演进。

## 🎬 核心成果演示 (Demos)

| 🧭 传统控制：有限状态机与角落逃逸 (`smart_driver`) | 🧠 强化学习：PPO 端到端激光雷达避障 (`rl_driver`) |
| :---: | :---: |
| <img src="docs/smart_driver_demo.gif" width="400"> | <img src="docs/rl_driver_demo.gif" width="400"> |
| **技术点**: P-Controller, 死锁破局, 运动学 | **技术点**: 观测空间降维, Tensor 映射, 安全限幅 |

## 🛠️ 技术深度与工程突破

### 1. 机械建模与物理仿真 (URDF & Gazebo)
- **Kinematics & Dynamics**: 独立编写双轮差速底盘及万向轮(Caster wheel)的 URDF。精确配置惯性张量 (Inertia Matrix) 与碰撞体积 (Collision)，通过设定万向轮的极低摩擦系数 ($\mu=0.001$)，彻底解决底盘干涉与滑动摩擦导致的转向死锁问题。
- **Sensor Integration**: 挂载 2D Lidar，完成 `gz.msgs` 到 `sensor_msgs/LaserScan` 的底层 QoS (Quality of Service) 桥接配置，解决 `Best Effort` 导致的数据丢失问题，并在 Rviz2 中实现点云与 TF 树的高精度对齐。

### 2. 经典控制：智能状态机设计 (FSM)
- 提取 LiDAR 及里程计状态，构建多级状态机实现平滑贴墙巡航。
- **Corner Deadlock Escape (角落逃逸机制)**：针对直角地形引发的死锁问题，创新性地引入“强制切向扰动速度”，在角落处打破常规的姿态对齐优先级，利用位移打破几何死锁，实现流畅过弯。

### 3. 强化学习实车部署 (Sim-to-Sim/Real Pipeline)
- **Feature Alignment (特征对齐)**：针对仿真环境与现实/高保真引擎的 Domain Gap，编写 Adapter 节点。将 360 维高频雷达数据清洗（处理 `inf` 异常值）并池化降采样至 3 维核心扇区特征，消除 OOD (Out of Distribution) 干扰。
- **Safety Control (底层执行保护)**：对 PPO 网络输出的高斯分布连续动作空间（Action Space）进行 `np.clip` 物理限幅。将无量纲的 Tensor 安全映射为真实的 `geometry_msgs/Twist` 速度指令，并部署基于心跳监测的自动刹车机制。

## 📂 仓库结构

```text
├── first_robot_bringup/  # ROS 2 Launch 启动包 (一键拉起环境、TF、桥接)
├── models/               # 训练好的 PPO 策略网络权重 (*.zip)
├── robot_description/    # 机器人的机械结构设计 (URDF 描述文件)
├── scripts/              # 核心控制节点源码
│   ├── smart_driver.py   # 传统控制：状态机壁障与贴边节点
│   ├── rl_driver.py      # AI控制：加载 PPO 模型并执行推理的节点
│   └── train_custom_env.py # Gym 强化学习数学替身环境与训练脚本
├── docs/                 # 演示素材存放处
└── README.md
