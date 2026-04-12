# Decision-Focused Learning 因果营销论文精选

> 筛选范围：DFL + 因果推断 + 营销端到端决策优化（2024–2025）

📄 **HTML 版本（深色主题，手机适配）**：[dfp_report.html](dfp_report.html)

---

## 核心论文（必读）

### ① DFCL — 美团
**Decision Focused Causal Learning for Direct Counterfactual Marketing Optimization**  
📎 [2407.13664](https://arxiv.org/abs/2407.13664) · 美团 · 2024-07

**一句话总结**：将因果推断（ITE估计）和运筹优化（整数随机规划）端到端结合，优惠券分配直接反事实优化，已在美团多场景上线A/B测试有效。

**三大技术挑战 & 解决方案**：
- **整数随机规划**：将预算不确定性建模为随机变量
- **反事实损失不可得**：通过因果推断估计ITE，用ITE代理反事实损失
- **求解器成本高**：采样梯度估计 + 历史数据缓存策略

**核心方法**：
1. 因果估计模块：估计每个用户对每种券的ITE（uplift）
2. DFL决策模块：基于ITE预测做组合优化（top-K用户 × 券面额）
3. 端到端训练：DFL regret指导ITE预测损失更新

✅ 美团外卖/到店多场景 A/B 测试，GMV 提升显著

---

### ② HCL — Lightspeed × 多伦多大学
**Heterogeneous Causal Learning for Optimizing Aggregated Functions in User Growth**  
📎 [2507.05510](https://arxiv.org/abs/2507.05510) · Lightspeed × 多伦多大学 × Angle.ac · 2025-07

**一句话总结**：跳过uplift显式建模，用Softmax门控直接端到端优化ROI，配合Barrier Function + Annealing处理硬约束，已全球部署。

**DRM 核心公式**：优化目标 = max_θ τ_r*(θ,X) / τ_c*(θ,X)
- p_i = softmax(f(x_i|θ)) → 可微的概率权重
- 分子 = ΣY_rⁱ · p_i · (I_{T=1} − I_{T=0}) → 加权增量收益
- 分母 = ΣY_cⁱ · p_i · (I_{T=1} − I_{T=0}) → 加权增量成本

**CRM：Barrier Function + Annealing 约束处理**：
- σ_i = sigmoid(T · (f(x_i) − d*)) → Sigmoid型Barrier动态"屏蔽"低效用户
- Annealing：T从小→大（软约束探索→硬约束聚焦）
- 支持比例约束（Top-40%用户）和预算约束（总成本≤$B）

✅ 线上ROI相比Causal Forest提升超20%，已全球部署于HCLW生产系统

---

## 后续发展

### Bi-DFCL — 美团
📎 [2510.19517](https://arxiv.org/abs/2510.19517) · 2025-10

在DFCL基础上解决「偏差-方差困境」：观测数据有偏但样本多；实验数据无偏但样本少。提出双层优化框架，统一观测+实验数据，通过隐函数微分联合求解，已在美团上线。

### FCA-RL — 滴滴
📎 [2507.02244](https://arxiv.org/abs/2507.02244) · PKDD'25 · 2025-07

网约车聚合平台竞争补贴场景。FCA（贝叶斯后验快速跟踪IRR）+ RLA（Actor-Critic调整拉格朗日乘子λ）。核心创新：将订单完成率分解为 IRR × In-Range/Out-Range完成率。

---

## 方法论对比

| 论文 | 优化目标 | 方法类别 | 约束处理 | 端到端 |
|------|----------|----------|----------|--------|
| DFCL | 反事实 GMV 最大化 | 因果推断 + 整数规划 | 随机规划 | ✅ |
| HCL-DRM | ROI（收益/成本） | Softmax门控 + 可微优化 | Barrier + Annealing | ✅ |
| HCL-CRM | 约束下 ROI 最大化 | Softmax门控 + 硬约束软化 | Sigmoid Barrier | ✅ |
| Bi-DFCL | 偏差-方差权衡决策质量 | 双层优化 + 隐函数微分 | 实验数据无偏估计引导 | ✅ |
| FCA-RL | 订单完成 + 预算率控制 | RL（MDP）+ 贝叶斯竞争适应 | Full-Achievement函数 | △ RL范式 |

---

## 关键洞察

1. **DFCL vs HCL 的本质区别**：DFCL是两阶段（先因果建模ITE → 再OR求解），HCL是真正单阶段端到端（Softmax群体级优化直接可微）

2. **HCL的核心洞察**：不追求每个用户的uplift预测准确，而是学准「哪个群体组合整体ROI最大」的打分函数f(x)

3. **反事实不可得的通用解法**：用ITE代理反事实损失，这是所有因果DFL方法的共同基础

4. **约束处理的两种范式**：① DFCL的随机规划（预算不确定性）；② HCL的Barrier+Annealing（比例/预算硬约束）

5. **端到端 vs 两阶段的核心矛盾**：两阶段方法训练目标（预测MSE）≠ 业务目标（决策ROI）

6. **工业落地共性**：三篇论文均已线上部署，因果DFL在营销场景已从学术研究进入工业实践阶段

---

## 发展时间线

- **2024-07**：DFCL（美团）首个因果DFL+营销端到端部署案例
- **2025-07**：HCL（Softmax端到端ROI）+ FCA-RL（RL竞争动态）并行
- **2025-10**：Bi-DFCL（双层优化统一观测+实验数据）

---

## 推荐阅读路径

1. **入门必读**：DFCL（2407.13664）—— 工业级因果DFL最完整案例
2. **进阶理解**：HCL DRM（2507.05510）—— 真正端到端Softmax门控
3. **方法深化**：HCL CRM部分 —— Barrier Function + Annealing约束处理
4. **数据融合**：Bi-DFCL（2510.19517）—— 观测+实验数据联合建模
5. **扩展视野**：FCA-RL（2507.02244）—— 竞争动态场景的RL解法

---

*整理时间：2026-04-13 · 数据来源：arXiv + 知乎专栏（苏漓、贝子涵）*
