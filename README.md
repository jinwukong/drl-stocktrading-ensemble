# Deep Reinforcement Learning for Automated Stock Trading (Ensemble Strategy)

本项目为本人2024.12在某基金量化部门实习时的所完成,和公司达成一致后开源代码.

本项目基于论文 [**《Deep Reinforcement Learning for Automated Stock Trading: An Ensemble Strategy》**]([https://arxiv.org/abs/2010.02046](https://openfin.engineering.columbia.edu/sites/default/files/content/publications/ensemble.pdf))，该论文发表于 2020 年 ACM 国际金融人工智能会议（ICAIF 2020）。论文提出了一种集成策略，将 **Proximal Policy Optimization**（PPO）、**Advantage Actor Critic**（A2C）和 **Deep Deterministic Policy Gradient**（DDPG） 三种深度强化学习算法结合起来，用于自动化股票交易。实证测试中使用了 **道琼斯 30** 只股票，结果表明在风险调整后收益方面，集成策略优于任何单一算法，以及传统基准（道指指数、最小方差投资组合策略等）。

在这个复现项目中，我们沿用了该论文思路，使用 **OpenAI Gym**（Gymnasium）来搭建交易环境，并基于 **Stable Baselines3** 训练多个 DRL 算法，最终集成成为一个可在道琼斯 30 或其他股票上进行自动化交易策略研究的工具。

> **说明**：本项目在实践中踩了很多“意料之外”的坑，尤其是 **Gym vs Gymnasium** 的兼容、**数据类型转换**、**维度不匹配** 等。我们在此 README 里做了比较详尽的介绍，以便后来者少走弯路。

---

## 依赖环境

- Python `3.9` 或以上版本  
- `stable_baselines3==2.5.0`  
- `gymnasium==1.1.0` （或旧版 `gym==0.25.x`，需改 4 元组返回）  
- `pandas==2.2.3`, `numpy==2.2.3`, `matplotlib==3.10.0`  

⚠️ **注意**：如果你想用旧 Gym 的方式（4 元组），需要卸载 `gymnasium` 或者手动兼容，这在踩坑记录里有详细说明。

---

## 数据准备

可选择从yfinace中下载数据或下载后手动更改文件读取的路径

---

## 踩坑记录

1. **Gym vs Gymnasium**  
   - SB3 2.5.0 默认习惯 4 元组 `(obs, reward, done, info)`；Gymnasium 新标准 5 元组 `(obs, reward, done, truncated, info)`。我一开始使用 Gymnasium 并返回 5 元组，导致各种解包错位报错（如 “cannot convert series to <class 'int'>”），最终通过只返回 4 元组或兼容包装解决了问题。

2. **数据类型转换**  
   - 当我在技术指标计算后，把各种 `float64`、`int64`、`object` 混在一块儿时，如果后续进行 `int(...)` 或 `np.float32(...)` 转换，就可能崩溃。

3. **数据读取**  
   - yfinace的不同版本下载得到的数据格式略有不同,我使用的数据格式以$AAPL为例

4. **多维度动作**  
   - 使用 `DummyVecEnv` 包装后，SB3 往往给出的 action shape 多出一维，需要在 `env.step()` 里 `if len(actions.shape)>1: actions = actions[0]`。否则 `actions[i]` 不是标量而是一小段数组，又会导致类型问题。
