# 🚗 ROS 2 Sim-to-Real: Autonomous Navigation & RL Control
> 基于 ROS 2 (Jazzy) 与 Gazebo 的移动机器人自主导航与端到端强化学习部署平台

![ROS2](https://img.shields.io/badge/ROS_2-Jazzy-22314E?logo=ros)
![Gazebo](https://img.shields.io/badge/Gazebo-Harmonic-FF8A00)
![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?logo=pytorch)

## 📖 项目简介
本项目旨在构建一个完整的 **"感知-决策-控制"** 闭环系统。利用机械运动学知识从零构建机器人 URDF 模型，打通 Gazebo 物理引擎与 ROS 2 TF 坐标树的通信链路，并分别实现了基于**有限状态机(FSM)的规则避障**与**基于 PPO 算法的端到端自动驾驶**。

## 🎬 效果演示 (Demo)
| 基于 FSM 的动态角落逃逸 | PPO 端到端视觉/激光雷达避障 |
| :---: | :---: |
| <img src="docs/fsm_demo.gif" width="400"> | <img src="docs/ppo_demo.gif" width="400"> |
*(请在这里放你之前乌龟贴墙跑的GIF，以及小车在Gazebo里躲箱子的GIF)*

## 🚀 核心工程实现

### 1. 机械建模与物理仿真 (URDF & Gazebo)
- **Kinematics & Dynamics**: 独立编写双轮差速底盘及万向轮的 URDF，精确配置惯性张量 (Inertia Matrix) 与碰撞体积 (Collision)，解决底盘干涉与滑动摩擦导致的转向死锁问题。
- **Sensor Integration**: 挂载单线激光雷达 (2D Lidar)，完成 `gz.msgs` 到 `sensor_msgs/LaserScan` 的数据桥接，并在 Rviz2 中实现点云与 TF 树的高精度对齐。

### 2. 传统控制：状态机与角落逃逸算法
- 提取 LiDAR 前、左、右扇区特征，构建 P-Controller 实现平滑贴墙巡航。
- **Corner Deadlock Escape**: 针对直角地形引发的死锁问题，创新性地引入“强制切向扰动速度”，打破平衡态，实现流畅的死角逃逸。

### 3. 强化学习部署 (Sim-to-Sim/Real)
- **Feature Alignment**: 编写 Adapter 节点，将 360 维高频雷达数据降采样并清洗（处理 `inf` 异常值），对齐 PPO 网络的观测空间，有效消除 OOD (分布外) 干扰。
- **Safety Control**: 对神经网络输出的连续动作空间（Action Space）进行 `np.clip` 物理限幅，将张量映射为安全的 `geometry_msgs/Twist` 速度指令，并部署底层安全刹车机制。
    ```

---
