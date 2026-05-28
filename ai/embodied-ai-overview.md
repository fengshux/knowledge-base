---
title: 具身智能（Embodied AI）学习路线图
date: 2026-05-14 17:20
tags: [具身智能, Embodied AI, 机器人, 强化学习, VLA, 计算机视觉]
category: ai
---

# 具身智能（Embodied AI）学习路线图

## 核心概念

**具身智能** = 智能体（Agent）+ 身体（Body）+ 环境（Environment）

传统 AI 像「大脑在瓶子里」——只处理文字/图片信息，但没有身体去触碰世界。具身智能则让 AI「长出身体」，像人类一样通过与环境互动来学习和完成任务。

典型应用场景：
- 机器人做家务（倒咖啡、叠衣服）
- 机械臂在工厂中精准组装零件
- 自动驾驶汽车感知路况并做出决策

## 五大核心知识模块

### 模块一：基础数学与编程（地基）

| 知识点 | 说明 |
|-------|------|
| 线性代数 | 向量、矩阵、空间变换——机器人的坐标、姿态全靠它 |
| 微积分 & 优化 | 梯度下降、反向传播——训练模型的核心 |
| 概率论与统计 | 贝叶斯滤波、卡尔曼滤波——处理传感器不确定性 |
| Python | 主力语言，尤其是 NumPy、PyTorch/TensorFlow |
| Linux & ROS | 机器人操作系统的基础环境 |

### 模块二：机器人学基础（身体）

具身智能的「物理载体」部分：

| 知识点 | 说明 |
|-------|------|
| 运动学 & 动力学 | 正/逆运动学、牛顿-欧拉方程 |
| 路径规划 | A* 算法、RRT（快速搜索随机树）、Dijkstra |
| 控制理论 | PID 控制、MPC（模型预测控制）、阻抗控制 |
| 传感器与执行器 | 激光雷达、IMU、力矩传感器、电机控制 |
| ROS/ROS2 | 机器人操作系统的消息通信、节点管理、仿真 |

推荐学习资源：崔克《机器人学导论》、Coursera 宾大 Robotics Specialization

### 模块三：计算机视觉（眼睛）

让机器人「看懂」世界：

| 知识点 | 说明 |
|-------|------|
| 图像处理基础 | 滤波、边缘检测、特征提取（SIFT/ORB） |
| 深度估计 | 单目/双目深度估计、3D 点云处理 |
| 物体检测与分割 | YOLO、Mask R-CNN、SAM（Segment Anything） |
| 视觉 SLAM | ORB-SLAM、VINS-Mono——同时定位与地图构建 |
| 3D 视觉 | 点云处理（PointNet++）、NeRF、3DGS |
| 多模态感知 | 视觉 + 触觉 + 听觉融合 |

### 模块四：AI 与决策（大脑）

这是最近爆发最快的部分，也是大模型与具身智能结合的关键：

| 知识点 | 说明 |
|-------|------|
| 强化学习（RL） | Q-learning、PPO、SAC——让机器人通过试错学技能 |
| 模仿学习（IL） | 行为克隆（BC）、逆强化学习（IRL）——看人类示范学动作 |
| 大语言模型（LLM） | GPT、Claude 等——给机器人「常识」和语言理解能力 |
| 视觉-语言-动作模型（VLA） | RT-2、PaLM-E——端到端的「看-理解-动」|
| 扩散策略（Diffusion Policy） | 用扩散模型生成机器人动作轨迹 |
| Sim-to-Real | 在仿真中训练，迁移到真实机器人 |

**当前前沿方向：**
- RT-2 / RT-X（Google DeepMind）：用互联网数据训练机器人 VLA 模型
- π0（Physical Intelligence）：通用家务机器人大模型
- Figure 02 + OpenAI：人形机器人 + LLM 推理
- 英伟达 GR00T：人形机器人通用基础模型

### 模块五：仿真与系统集成（连接身体与大脑）

| 知识点 | 说明 |
|-------|------|
| 物理仿真 | MuJoCo、Isaac Gym、SAPIEN、PyBullet |
| 高质量仿真器 | NVIDIA Isaac Sim、Genesis |
| 遥操作数据采集 | 通过 VR/力反馈设备采集人类示范数据 |
| 系统集成 | 把感知 → 规划 → 控制整个链路串起来 |

## 推荐学习路径

### 第一阶段（1-2个月）——打好基础
- Python + PyTorch 基础
- 线性代数 & 概率论 回顾
- ROS 入门教程

### 第二阶段（2-3个月）——机器人入门
- Coursera: Robotics Specialization (UPenn)
- 动手搭一个简单机器人（如 TurtleBot 仿真）
- 学习 PID 控制、路径规划

### 第三阶段（2-3个月）——AI 决策
- 强化学习：李宏毅 RL 课程 / Spinning Up in RL (OpenAI)
- 模仿学习入门
- 动手项目：让机械臂学会抓取

### 第四阶段（持续进阶）——前沿跟进
- 阅读具身智能顶会论文（CoRL, RSS, ICRA, IROS）
- 关注：RT-2、π0、GR00T 等大模型进展
- 动手：在 sim 中跑通一个 VLA 模型

## 常用资源

**课程**
- CS 287: Advanced Robotics (UC Berkeley)
- Coursera: Robotics Specialization
- OpenAI Spinning Up — 强化学习

**开源框架**
- robosuite / ManiSkill — 机器人操作仿真
- LeRobot (Hugging Face) — 具身智能数据集与模型
- Isaac Gym / Isaac Sim — NVIDIA 仿真平台

**数据集**
- Open X-Embodiment — 跨机器人通用数据集
- DROID — 大规模遥操作数据集

## 最佳实践

1. 先别急着追最前沿——先把 ROS + 运动学 + 简单控制跑通
2. 从仿真开始——不需要买真实机器人，MuJoCo / Isaac Gym 里就能做实验
3. 动手是最好的学习方式——找个开源项目跑一遍，比看书有用得多
4. 关注多模态大模型——LLM + VLM + 机器人控制的结合是当前最大的机会点

## 注意事项
- ROS 1 已逐渐停更，建议直接学 ROS 2（Humble / Jazzy 版本）
- 具身智能领域发展极快，建议持续跟进顶会论文和开源项目
- 硬件成本高，初期建议以仿真为主

## 参考资料
- https://spinningup.openai.com/
- https://robotics-transformer-x.github.io/
- https://github.com/huggingface/lerobot

---

*最后更新：2026-05-14*