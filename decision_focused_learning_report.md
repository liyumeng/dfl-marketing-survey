# Decision-Focused Learning 因果营销论文精选

> 筛选范围：DFL + 因果推断 + 营销端到端决策优化

---

## 1. 核心论文（必读）

### DFCL — 美团 ⭐⭐⭐
**Decision Focused Causal Learning for Direct Counterfactual Marketing Optimization**

- **arXiv**: [2407.13664](https://arxiv.org/abs/2407.13664)
- **作者**: Hao Zhou, Rongxiao Huang, Shaoming Li, Guibin Jiang, Jiaqi Zheng, Bing Cheng, Wei Lin
- **机构**: 美团（Meituan）
- **时间**: 2024-07

**摘要**: 营销优化是提升用户参与度的关键。现有方法将问题拆解为ML（机器学习）+ OR（运筹优化）两阶段，但ML的训练目标不考虑下游OR任务，导致预测精度与决策质量脱节。DFL将ML和OR统一为端到端框架，但将DFL部署到营销场景面临三大挑战：①预算分配是0-1整数随机规划问题；②反事实不可得导致决策损失无法直接计算；③OR求解器频繁调用成本极高。本文提出DFCL框架解决以上挑战，在美团多场景上线并取得显著A/B测试效果。

**三大技术挑战**：
- 整数随机规划：预算不确定且波动大
- 反事实损失不可得：最优解永远无法获得
- OR求解器成本高：大规模训练数据下不可行

**核心方法**：
1. 因果推断模块：估计每个用户对每种券的ITE（增量效果）
2. DFL决策模块：基于ITE做组合优化（用户 × 券面额）
3. 端到端训练：DFL regret指导因果模型更新方向

✅ 美团外卖/到店多场景 A/B 测试有效

---

### HCL — Lightspeed × 多伦多大学 × Angle.ac ⭐⭐⭐
**Heterogeneous Causal Learning for Optimizing Aggregated Functions in User Growth**

- **arXiv**: [2507.05510](https://arxiv.org/abs/2507.05510)
- **作者**: Shuyang Du, Jennifer Zhang, Will Y. Zou
- **机构**: Lightspeed × 多伦多大学 × Angle.ac
- **时间**: 2025-07

**摘要**: 用户增长是互联网公司的核心战略。传统uplift模型只优化单一指标，无法处理成本约束和多目标权衡。本文提出端到端因果学习框架，直接优化聚合目标函数（ROI = 增量收益/增量成本），通过深度学习+Softmax门控实现可微的群体级优化，支持灵活的业务约束，已在全球部署验证。

**DRM 核心公式**：
- 优化目标：max_θ τ_r*(θ,X) / τ_c*(θ,X)（整体ROI，而非个体uplift）
- p_i = softmax(f(x_i|θ))：可微的概率权重，归一化用户相对重要性
- 不追求每个τ(x_i)准确，而是追求选对一组用户使得整体ROI最大

**CRM：Barrier Function + Annealing**：
- σ_i = sigmoid(T · (f(x_i) − d*))：Sigmoid型Barrier动态屏蔽低效用户
- Annealing：T从小→大（软约束探索→硬约束聚焦），避免梯度断裂
- 支持比例约束（Top-40%用户）和预算约束（总成本≤$B）

✅ 线上ROI相比Causal Forest提升超20%，已全球部署于HCLW生产系统

---

## 2. 后续发展

### Bi-DFCL — 美团
- **arXiv**: [2510.19517](https://arxiv.org/abs/2510.19517)
- **机构**: 美团
- **时间**: 2025-10

在DFCL基础上提出双层优化框架，统一观测数据和实验数据的优势：观测数据有偏但样本多；实验数据无偏但样本少。通过隐函数微分（implicit differentiation）联合求解，解决偏差-方差困境，已在美团大规模上线。

### FCA-RL — 滴滴
- **arXiv**: [2507.02244](https://arxiv.org/abs/2507.02244)
- **机构**: 滴滴出行（PKDD'25）
- **时间**: 2025-07

网约车聚合平台竞争补贴场景。FCA（贝叶斯后验快速跟踪IRR）+ RLA（Actor-Critic调整拉格朗日乘子λ）。核心创新：将订单完成率分解为 IRR × In-Range/Out-Range完成率，建模竞争对手价格敏感的动态因果关系。

---

## 3. 方法论对比

| 论文 | 优化目标 | 方法类别 | 约束处理 | 端到端 |
|------|----------|----------|----------|--------|
| DFCL (2407.13664) | 反事实 GMV 最大化 | 因果推断 + 整数规划 | 随机规划 | ✅ |
| HCL-DRM (2507.05510) | ROI | Softmax门控 + 可微优化 | Barrier + Annealing | ✅ |
| HCL-CRM (2507.05510) | 约束下 ROI | Softmax门控 + 硬约束软化 | Sigmoid Barrier | ✅ |
| Bi-DFCL (2510.19517) | 偏差-方差权衡 | 双层优化 + 隐函数微分 | 实验数据引导 | ✅ |
| FCA-RL (2507.02244) | 订单完成 + 预算控制 | MDP + 贝叶斯 | Full-Achievement | △ |

---

## 4. 关键洞察

1. **DFCL vs HCL**：DFCL是两阶段（先ITE → 再OR求解）；HCL是真正单阶段（Softmax直接可微）
2. **HCL核心洞察**：不追求个体uplift准确，而是学准「哪个群体组合整体ROI最大」
3. **反事实不可得通用解**：用ITE代理反事实损失
4. **约束处理两种范式**：DFCL随机规划 vs HCL Barrier+Annealing
5. **两阶段核心矛盾**：训练目标（MSE）≠ 业务目标（决策ROI）
6. **工业落地**：三篇论文均已部署，因果DFL已进入工业实践阶段

---

## 5. 发展时间线

```
2024-07  DFCL（美团）— 首个因果DFL+营销端到端部署
2025-07  HCL（Lightspeed）— Softmax端到端ROI优化
          FCA-RL（滴滴）— RL竞争动态适应
2025-10  Bi-DFCL（美团）— 双层优化统一观测+实验数据
```

---

## 6. 推荐阅读路径

1. **入门**：DFCL（2407.13664）— 工业级因果DFL最完整案例
2. **进阶**：HCL DRM（2507.05510）— 真正端到端Softmax门控
3. **深化**：HCL CRM — Barrier Function + Annealing约束处理
4. **融合**：Bi-DFCL（2510.19517）— 观测+实验数据联合建模
5. **扩展**：FCA-RL（2507.02244）— 竞争动态RL解法

---

*整理时间：2026-04-13*
