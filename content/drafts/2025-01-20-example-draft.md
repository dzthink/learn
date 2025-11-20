# 示例 Draft：RLHF 奖励模型相关问题（Draft）

## 1. 当前困惑

- 奖励模型为什么采用 pairwise preference 进行训练？
- 直接回归标量奖励是否可行？

## 2. 学习笔记（示例）

- RLHF 基本流程：
  - SFT 模型
  - 奖励模型
  - 策略优化（PPO/DPO）
