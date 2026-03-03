
# 🚗 ROS 2 Sim-to-Real: Autonomous Navigation & RL Control
> 基于 ROS 2 (Jazzy) 与 Gazebo 的移动机器人自主导航与端到端强化学习部署平台

![ROS2](https://img.shields.io/badge/ROS_2-Jazzy-22314E?logo=ros)
![Gazebo](https://img.shields.io/badge/Gazebo-Harmonic-FF8A00)
![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?logo=pytorch)
![Stable-Baselines3](https://img.shields.io/badge/Algorithm-PPO-0052CC)

## 📖 项目简介
本项目旨在构建一个完整的 **"感知-决策-控制"** 具身智能闭环系统。利用机械运动学知识从零构建机器人 URDF 物理模型，打通 Gazebo 物理引擎与 ROS 2 TF 坐标树的异步通信链路，并分别实现了基于**基于 LiDAR 扇区划分的自主避障**与**基于 PPO 算法的端到端自动驾驶**。

本项目展示了从传统的几何逻辑控制，向现代数据驱动(Data-Driven)与 Sim-to-Real 部署的完整工程演进。

## 🎬 核心成果演示 (Demos)

| 🧭 传统控制：基于 LiDAR 扇区划分的自主避障 (smart_driver) | 🧠 强化学习：PPO 端到端激光雷达避障 (`rl_driver`) |
| :---: | :---: |
| <img src="docs/smart_driver_demo.gif" width="400"> | <img src="docs/rl_driver_demo.gif" width="400"> |
| **技术点**:扇区降维, 阈值状态机, 运动学 | **技术点**: 观测空间降维, Tensor 映射, 安全限幅 |

## 🛠️ 技术深度与工程突破

### 1. 机械建模与物理仿真 (URDF & Gazebo)
- **Kinematics & Dynamics**: 独立编写双轮差速底盘及万向轮(Caster wheel)的 URDF。精确配置惯性张量 (Inertia Matrix) 与碰撞体积 (Collision)，通过设定万向轮的极低摩擦系数 ($\mu=0.001$)，彻底解决底盘干涉与滑动摩擦导致的转向死锁问题。
- **Sensor Integration**: 挂载 2D Lidar，完成 `gz.msgs` 到 `sensor_msgs/LaserScan` 的底层 QoS (Quality of Service) 桥接配置，解决 `Best Effort` 导致的数据丢失问题，并在 Rviz2 中实现点云与 TF 树的高精度对齐。

### 2. 经典控制：：基于 LiDAR 的反应式导航 (Reactive Navigation)
- 数据清洗与降维 (Data Pre-processing)：实时监听 /scan 点云数据，利用列表推导式处理超出量程的 inf 异常值，并将其归一化至安全阈值内，保证控制系统的稳定性。
- 动态扇区切割 (Sector Slicing)：摒弃对全量 360 维点云的复杂运算，将感知视野在空间上切分为三大核心特征区（前 front、左 left、右 right），将空间测距转化为三个降维特征输入。
- 阈值状态机避障 (Threshold FSM)：设计基于最小安全距离 (warn_dis = 1.0m) 的分层控制逻辑。当触发危险阈值时，自动对比左右两侧空旷度 (min(left) > min(right))，动态分配角速度实现最优避让。

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
