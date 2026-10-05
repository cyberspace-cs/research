# Research: Agent & Decision Model 前沿论文调研

> 整理时间：2026-10-05
> 用途：科研参考、技术选型、Agent 框架设计灵感

---

## 📚 论文一：Skill2Env — 用社区 Skills 训练 Agent

### 基本信息

| 项目 | 内容 |
|------|------|
| **机构** | NVIDIA Labs (NVlabs) |
| **仓库** | https://github.com/NVlabs/Skill2Env |
| **论文** | https://github.com/NVlabs/Skill2Env/blob/main/paper/Skill2Env_arXiv.pdf |
| **数据集** | https://hub.harborframework.com/datasets/skill2env/skill2env |
| **Stars** | 158 ⭐ |
| **许可** | Apache-2.0 |
| **语言** | Python |
| **创建时间** | 2026-09-16 |

### 核心思想

把开源社区已有的 **Agent Skills**（写代码、处理文档、做研究、操作工具等），通过自动化 pipeline 转化为可用于**强化学习**的真实任务环境。

**一句话总结**：让模型优化的目标与价值分布，源自整个社区真实贡献的知识与需求，而非仅由少数 benchmark 或中心化机构定义。

### 技术架构

```
开源 Skills 仓库（成千上万）
        ↓
   自动化 Pipeline
        ↓
┌─────────────────────────┐
│  1. 任务提取             │
│     从 Skill 提取真实任务  │
│                         │
│  2. 环境构建             │
│     Agent 直接在 GitHub   │
│     repo / 论文 artifacts │
│     上工作                │
│                         │
│  3. 奖励设计             │
│     programmatic tests    │
│     作为可验证 reward     │
│                         │
│  4. 质量评估             │
│     基于原始 Skill 生成   │
│     rubric 评估质量与对齐 │
└─────────────────────────┘
        ↓
   RL 训练框架
   ├─ Molt（训练）
   ├─ Polar（sandbox rollout + scoring）
   ├─ vLLM（推理）
   └─ Ray（分布式调度）
```

### 关键结果

- 训练模型：**Qwen3.8-27B**
- 仅 **300 步 RL** 即在以下 benchmark 取得提升：
  - TerminalBench 2.1
  - S2EBench（自建 held-out）

### 核心价值

1. **数据飞轮**：社区不断贡献 Skills → 自动变成训练数据 → 模型更强 → 更好的 Skills
2. **可验证奖励**：用 programmatic tests 而不是 LLM judge，更客观
3. **开放生态**：不依赖中心化 benchmark，而是整个开源社区

### 对我们的启发

- 我们的 agent-skills 仓库也可以用类似思路：把每个 Skill 变成一个可训练的 RL 环境
- Jev 决策模型可以作为 reward model 的一部分
- RSI（递归自我改进）的关键就是：Skills → Environment → RL → 更好的 Skills

---

## 📚 论文二：AutoBenchmark — 让 AI 自己出考卷

### 基本信息

| 项目 | 内容 |
|------|------|
| **机构** | Meta AI (RAM 团队) |
| **博客** | https://facebookresearch.github.io/RAM/blogs/autobench/ |
| **作者 X** | https://x.com/billxbf/status/2101990023788908956 |
| **方向** | 自动创建 Benchmark + 评测创建 Benchmark 的能力 + Human-in-the-loop |

### 核心思想

研究 Agent 自己创建 Benchmark 的能力，以及人类反馈在这个过程中起什么作用。

**三件事**：
1. **自动创建 benchmark**：端到端完成，产出 Harbor 格式的 benchmark 包
2. **评测创建 benchmark 的能力**：给"创建 benchmark"这件事本身打分
3. **研究 human-in-the-loop**：人类反馈什么时候有用？ coarse-grained 还是 fine-grained？

### 技术流程

```
Step 1: Proposal（提案）
  ├─ 定义任务规范
  ├─ 收集数据来源
  └─ 搭建题目、参考答案和评分器

Step 2: Solver（求解）
  ├─ 多个 Agent 尝试解 benchmark
  └─ 收集解题结果

Step 3: Judge（评判）
  ├─ 多个 LLM judge 评判
  └─ 看 benchmark 难度是否合适

Step 4: Iteration（迭代）
  ├─ 调整 benchmark 难度
  ├─ 加入人类反馈（可选）
  └─ 直到 benchmark 足够难但又能解
```

### 关键发现

从性能曲线图可以看到：

| 反馈方式 | 效果 |
|----------|------|
| **No feedback** | 一条基线，Agent 自主选择 benchmark |
| **Coarse-grained proposal feedback** | 粗略反馈，效果一般 |
| **Fine-grained proposal feedback** | 精细反馈，效果最好 |

**核心结论**：
- 完全自主的 benchmark 创建已经能工作
- 人类反馈**能显著提升** benchmark 质量
- **fine-grained feedback > coarse-grained feedback**
- "通过所有 LLM judge" 的 marker（实心点）和"被至少一个 LLM judge catch"的 marker（空心点）形成对比

### 测试的模型

- Muse Spark（内部 solver）
- Muse Glimmer（内部 solver）
- NVIDIA Nemotron-3.5-Lightning-30B-A3B（外部 held-out solver）

### 对我们的启发

- 我们的 Jev 对话副驾可以用类似思路：让用户反馈（"这句话太冲了"）自动优化对话策略
- Benchmark 生成能力本身就是一种元能力
- Human-in-the-loop 的粒度很重要：太粗没用，要精细到具体题目

---

## 🔗 相关链接汇总

### Skill2Env
- GitHub: https://github.com/NVlabs/Skill2Env
- 论文 PDF: https://github.com/NVlabs/Skill2Env/blob/main/paper/Skill2Env_arXiv.pdf
- 数据集: https://hub.harborframework.com/datasets/skill2env/skill2env
- 作者 X: https://x.com/billxbf/status/2101990023788908956

### AutoBenchmark (Meta RAM)
- 博客: https://facebookresearch.github.io/RAM/blogs/autobench/
- RAM 团队主页: https://facebookresearch.github.io/RAM/

---

## 🎯 我们可以怎么做？

### 短期（1-2 周）
- [ ] 跑通 Skill2Env 的最小 demo：用一个 Skill 生成 RL 环境
- [ ] 把我们的 agent-skills 仓库的每个 Skill 标注成可训练格式
- [ ] 用 Jev 作为 reward judge 的一部分

### 中期（1-2 月）
- [ ] 搭建我们自己的 AutoBenchmark pipeline：
  - 输入：一个对话场景（比如"和老板谈加薪"）
  - 输出：一套自动生成的评测题 + 评分标准
- [ ] 用 RSI 思路：让 Agent 自己写 Skill → 自己训练 → 自己评测

### 长期方向
- [ ] Skill → Environment → RL → Better Skills 的完整飞轮
- [ ] Human-in-the-loop 的精细反馈机制
- [ ] 多 Agent 协作 + 动态角色博弈
