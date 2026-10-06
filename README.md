# Research: Agent & Decision Model 前沿论文调研

> 整理时间：2026-10-05
> 用途：科研参考、技术选型、Agent 框架设计灵感

---

## 📚 一、Skill 生成与蒸馏

### 1. Repo-To-Skill (DisCo) — 从 GitHub 仓库蒸馏 Agent Skill

| 项目 | 内容 |
|------|------|
| **机构** | BAAI（智源）× 中科大 × 人大 |
| **论文** | arXiv 2609.02749（48 页） |
| **代码** | https://github.com/VectorSpaceLab/AREX-Skill |
| **方向** | 从 GitHub 仓库自动蒸馏可复用的 Agent Skill |

#### 核心思想

自主 Agent 开始做端到端的机器学习研究了，但它们缺少 **operational knowledge**（操作知识）——就是那种"知道怎么做"的 know-how，而不是"知道是什么"的知识。

#### 技术流程

```
1000 个常用 ML 仓库（开源社区）
        ↓
   自动爬取 + 过滤
        ↓
   Repo2Skill 生成候选 skill 包
        ↓
   专家审核：保留、修改、丢弃
        ↓
   AREX-Skill 库：5000+ 验证过的 skill
   覆盖 20 个领域、178 个可执行工作流
```

#### 关键结果

| Benchmark | 提升 |
|-----------|------|
| MLE-bench | **+134.3%**（31.11% → 72.89%） |
| PaperBench | +34.4% |
| FrontierCS | +9.22% |
| PassNet | +14.0% |

#### 核心主张

> Agent 缺的不是更强的控制循环，是「操作知识」这一层。

---

### 2. Repo2Skill-Evo — 仓库 Skill 会过时（字节 × 北大）

| 项目 | 内容 |
|------|------|
| **机构** | ByteDance × 北京大学 × 北京交通大学 |
| **论文** | arXiv 2608.21964v1 |
| **方向** | 仓库 Skill 会静默过时，需要持续演化 |

#### 核心问题

Repo2Skill 生成的 skill 是静态的，但 GitHub 仓库一直在更新——依赖版本变了、API 变了、代码重构了，原来的 skill 就失效了。

#### 解决方案：Repo2Skill-Evo

```
旧 skill v1
    ↓
监控仓库变化（git commit）
    ↓
自动检测：哪些 skill 过时了？
    ↓
专家审核：更新、修正、验证
    ↓
新 skill v2
```

#### 对我们的启发

- 我们的 agent-skills 仓库也需要一个"skill 保鲜"机制
- Skill 不是一劳永逸的，需要持续维护
- RSI 的闭环：Skill → 用 → 发现问题 → 更新 Skill → 更好的 Skill

---

### 3. Skill2Env — 用社区 Skills 训练 Agent（NVIDIA）

| 项目 | 内容 |
|------|------|
| **机构** | NVIDIA Labs (NVlabs) |
| **仓库** | https://github.com/NVlabs/Skill2Env |
| **Stars** | 158 ⭐ |
| **许可** | Apache-2.0 |

#### 核心思想

把开源社区已有的 Agent Skills，通过自动化 pipeline 转化为可用于强化学习的真实任务环境。

#### 技术架构

```
开源 Skills 仓库（成千上万）
        ↓
   自动化 Pipeline
        ↓
┌─────────────────────────┐
│  1. 任务提取             │
│  2. 环境构建             │
│  3. 奖励设计             │
│     programmatic tests    │
│  4. 质量评估             │
│     rubric 评估质量与对齐 │
└─────────────────────────┘
        ↓
   RL 训练框架
   ├─ Molt（训练）
   ├─ Polar（sandbox rollout）
   ├─ vLLM（推理）
   └─ Ray（分布式调度）
```

#### 关键结果

- 训练模型：Qwen3.8-27B
- 仅 300 步 RL 即在 TerminalBench 2.1 和 S2EBench 取得提升

---

## 📚 二、Benchmark 自动生成

### 4. AutoBenchmark — 让 AI 自己出考卷（Meta RAM）

| 项目 | 内容 |
|------|------|
| **机构** | Meta AI (RAM 团队) |
| **博客** | https://facebookresearch.github.io/RAM/blogs/autobench/ |
| **方向** | 自动创建 Benchmark + 评测创建 Benchmark 的能力 + Human-in-the-loop |

#### 三件事

1. **自动创建 benchmark**：端到端完成，产出 Harbor 格式的 benchmark 包
2. **评测创建 benchmark 的能力**：给"创建 benchmark"这件事本身打分
3. **研究 human-in-the-loop**：人类反馈什么时候有用？

#### 关键发现

| 反馈方式 | 效果 |
|----------|------|
| No feedback | 基线 |
| Coarse-grained feedback | 一般 |
| **Fine-grained feedback** | **最好** |

---

### 5. DESIGNER — 基于设计逻辑的多学科数据合成

| 项目 | 内容 |
|------|------|
| **方向** | 多学科推理数据合成 |
| **项目主页** | attention-is-all-i-need.github.io/Design-Logic-Reasoning |

#### 核心思想

现有合成数据方法的两大困境：
1. **以问题为中心**：受限于种子问题的覆盖范围和模型 bias
2. **以文档为中心**：难以控制难度，退化为简单知识回忆

DESIGNER 提出由 **"设计逻辑"（Design Logic）** 引导的全新范式。

#### 什么是 Design Logic？

人类专家出题时遵循的结构化流程：
```
识别知识点 → 构建题目场景 → 设计推理路径 → 预设干扰项
```

这是一种可复用的元知识，使 LLMs 能从完全不同的源文本中生成具有相同复杂推理模式的新问题。

#### 数据集规模

| 数据集 | 题量 | 来源 |
|--------|------|------|
| DLR-Book | **304 万题** | 书籍语料 |
| DLR-Web | **166 万题** | 网页语料 |
| 覆盖学科 | **75 个** | STEM + 人文等 |

#### 关键结果

- 仅用这些数据对 Qwen3 / Llama3 base 版本做 SFT，性能**超过官方完整 post-training 版本**
- 数据也适合做 SFT-style 的 mid-training

---

## 📚 三、自进化智能体

### 6. GenericAgent — 复旦肖仰华团队的自进化通用智能体

| 项目 | 内容 |
|------|------|
| **机构** | 复旦大学 知识工场实验室 / A3 实验室 |
| **作者** | lsdefine |
| **仓库** | https://github.com/lsdefine/GenericAgent |
| **你的 Fork** | https://github.com/cyberspace-cs/GenericAgent |
| **Stars** | **14,280 ⭐** |
| **Forks** | 1,665 |
| **语言** | Python |
| **开源时间** | 2026-01-16 |

#### 核心思想

一个能**自主学习、自我进化**的通用智能体。你给它一个目标，它能自己思考、操作电脑完成，而不是你说一句它答一句。

**最出圈的操作**：学会了像人一样自己刷微信、发朋友圈——甚至帮肖仰华教授发朋友圈和好友互动，好友都没看出来是 AI。

#### 核心亮点

| 亮点 | 说明 |
|------|------|
| **极简架构** | 核心代码仅 **3,300 行**，传统架构需要几十万行 |
| **自组织记忆** | 有自己的"大脑"，持续学习整理信息，越用越熟练 |
| **自主成长** | Fork 模式（复制自己试不同策略）+ 探索模式（空闲时自己学新技能） |
| **直接操控浏览器** | 接管你正在用的浏览器，无需重新登录，真正的人机接力 |
| **硬件要求低** | 只要 Python 环境，甚至手机都能运行 |
| **模型无关** | 不依赖特定大模型，Claude、Gemini、Kimi 都可以当"大脑" |
| **Token 高效** | Token 消耗仅为同类产品（OpenClaw 等）的 **1/3 ~ 1/10** |

#### 技术架构

`
用户目标
    ↓
┌─────────────────────────┐
│   GenericAgent 核心     │
│   (3,300 行代码)       │
├─────────────────────────┤
│  1. 观察环境             │
│     截图、DOM、屏幕元素  │
│                         │
│  2. 规划与决策           │
│     分解任务、选择工具    │
│                         │
│  3. 执行操作             │
│     点击、输入、滚动     │
│                         │
│  4. 学习与沉淀           │
│     成功经验存入技能库    │
│     下次直接复用         │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│   技能库（自生长）       │
│   5 大类、9 个原子工具   │
│   20+ 轮工具扩展         │
│   不会"上下文爆炸"       │
└─────────────────────────┘
`

#### 与传统 Agent 的区别

| 对比项 | 传统聊天机器人 | GenericAgent |
|--------|---------------|--------------|
| 交互方式 | 你说一句它答一句 | 给目标，自己完成 |
| 能力增长 | 固定，需要开发者更新 | 自进化，越用越强 |
| 代码量 | 几十万行 | **3,300 行种子** |
| 记忆 | 对话级，用完即忘 | 持久化技能库，跨会话积累 |
| 工具 | 预设固定工具 | 自己学习新技能、组合工具 |

#### 对我们的启发

1. **RSI 的极简实现**：3000 行代码就能实现自进化——核心不是多复杂，而是"种子 + 生长机制"
2. **技能库是关键**：不是模型越强越好，而是技能库越丰富、越精准越好
3. **我们可以做什么**：
   - 学习它的自组织记忆架构
   - 把我们的 agent-skills 仓库做成它的技能库
   - 用 Jev 做它的决策引擎（判断"这一步该做什么"）

---
### 7. AutoDataBench — 让 Agent 自动写数据（RSI 关键环节）

| 项目 | 内容 |
|------|------|
| **论文** | arXiv 2609.35025 |
| **仓库** | https://github.com/StarDewXXX/AutoDataBench |
| **你的 Fork** | https://github.com/cyberspace-cs/AutoDataBench |
| **Stars** | 23 ⭐（新开源） |
| **方向** | Agent 自动合成训练数据 |

#### 核心问题

数据对 LLM 来说是最重要的，这也是 **RSI（递归自我改进）的关键环节**。

现在造数据还是 human-in-the-loop 的过程——研究员、领域专家、coding agent 一起写。但 Agent 越来越强了，能不能让 Agent 针对某个 benchmark 自主完成数据生产？

#### 技术思路

`
┌─────────────────────┐     ┌─────────────────────┐
│  (a) 现在：人类主导   │     │  (b) AutoDataBench  │
│     human in loop    │     │    fully autonomous │
├─────────────────────┤     ├─────────────────────┤
│  Benchmark (固定)    │────→│  Benchmark (同一个)  │
│         ↓            │     │         ↓            │
│  谁写数据？          │     │  谁写数据？          │
│  · 研究员            │     │  · 自主 Agent        │
│  · 领域专家          │     │  · 目标模型 API      │
│  · coding agent      │     │  · web               │
│         ↓            │     │         ↓            │
│  质检（pass rate、   │     │  同样的质检          │
│  rubric）            │     │  + 给 Agent 打分     │
│         ↓            │     │         ↓            │
│  新题目（人工格式）   │     │  新题目（suite 格式） │
└─────────────────────┘     └─────────────────────┘
`

#### 选用的 Benchmark

| Benchmark | 领域 |
|-----------|------|
| Terminal Bench | Coding |
| Terminal Bench Science | Science |
| AutomationBench | Automation |

#### 核心创新

1. **全自动化数据生产**：从 human-in-the-loop 变成 fully autonomous
2. **质检标准复用**：用业内常用的 pass rate 和 rubric 作为合成数据的衡量标准
3. **最基础 setting**：针对一个目标模型 + 一道题目，合成一道新题

#### 为什么重要？

这是 RSI 的闭环关键：
`
Agent 变强 → 自动写数据 → 训练更强的 Agent → 再写更好的数据 → ...
`

没有自动数据生产，RSI 就卡在"需要人工造数据"这一步。

#### 对我们的启发

- 我们的对话副驾也可以自动生成训练数据：用户对话 → Agent 分析好坏 → 自动生成新的对话样本
- Skill2Env + AutoDataBench = 完整的 RSI 数据飞轮

---
### 8. Harness Engineering — 11 个生产级 Coding Agent 的源码解剖

| 项目 | 内容 |
|------|------|
| **论文** | arXiv 2609.00006 |
| **标题** | Harness Engineering: Anatomy, Architecture, and Evolution of Coding Agents |
| **作者** | Paul Barbaste 等（Inclusive Brains, Wavestone AI Lab） |
| **时间** | 2026 年 7 月 |
| **分析对象** | 11 个生产级 coding harness + 1 个 meta-harness |

#### 核心定义

> **Agent = Model + Harness**
>
> Harness 是模型之外的全部：循环、工具、上下文管理、安全控制、编排、扩展面。

Harness Engineering 就是设计和演化这个 runtime 的学科。

#### 分析的 11 个系统

| 类别 | 系统 |
|------|------|
| **大厂官方** | Claude Code (Anthropic), Codex CLI (OpenAI), Gemini CLI (Google), Mistral Vibe (Mistral) |
| **开源社区** | OpenHands, Aider, Mini-SWE-Agent, Hermes (Nous Research), Pi, OpenCode, OpenClaw |
| **Meta-Harness** | Omnigent (Databricks) — 第一个能跑多个 harness 的元 harness |

#### 七大核心子系统

论文把 harness 拆成 7 个 canonical subsystems：
1. **Loop engine** — think/act/observe 循环
2. **Context management** — 上下文压缩、管理
3. **Tools** — 工具集
4. **Permissions & safety** — 权限控制
5. **Memory** — 持久化记忆
6. **Orchestration** — 多 agent 编排
7. **Extension surfaces** — 扩展机制（skills、plugins、hooks）

#### 关键发现（13 个观察 + 29 个设计模式）

| 发现 | 说明 |
|------|------|
| **不用通用框架** | 没有一个 agent runtime import LangChain/LangGraph/AutoGen |
| **Google 不用自己的框架** | Gemini CLI 既不用 Google 自己的 agent 框架，也不用 vector embeddings |
| **全是手写异步循环** | 没有用通用 agent 框架，全是 hand-rolled async loops |
| **代码检索不用向量** | 全是 ripgrep、tree-sitter、glob、Markdown context files |
| **SKILL.md > MCP** | SKILL.md skills 比 MCP 更普及（9/11 vs 8/11） |
| **趋同现象** | Codex 学 Claude Code 的 hook vocabulary，OpenHands 学 Claude Code 的 plugin 格式 |
| **Meta-harness 出现** | OpenHands 能跑 Claude Code/Codex/Gemini CLI 作为可互换后端 |
| **从工具变平台** | 2026 上半年，harness 从工具变成了可 import 的 SDK / platform |

#### 最小可行 Harness（90 行代码）

论文最后给了一个 ~90 行的 minimum-viable-harness scaffold：
- 核心：线性的 	ool_call → execute → observe 循环
- 阈值式压缩
- 扁平权限门
- 单个扩展 hook

#### 对我们的启发

1. **Harness 比模型更重要**：Stanford 研究发现，orchestration code 带来的性能差异比模型选择还大
2. **不要用大框架**：生产级 harness 全是手写的，不要 import LangChain
3. **代码检索用 ripgrep 就够了**：不用 vector embeddings
4. **SKILL.md 是事实标准**：比 MCP 更普及
5. **我们的 DIY harness 应该学这个架构**：7 个子系统，90 行最小骨架

---
## 🎯 我们可以怎么做？

### 短期（1-2 周）
- [ ] 跑通 Repo-To-Skill 的最小 demo：从一个 GitHub 仓库提取 skill
- [ ] 把我们的 agent-skills 仓库标注成可训练格式
- [ ] 用 Jev 作为 reward judge 的一部分

### 中期（1-2 月）
- [ ] 搭建 Skill 保鲜机制：监控仓库变化，自动更新过时 skill
- [ ] 用 DESIGNER 思路生成我们自己的对话场景数据集
- [ ] 用 RSI 思路：Agent 自己写 Skill → 自己训练 → 自己评测

### 长期方向
- [ ] Skill → Environment → RL → Better Skills 的完整飞轮
- [ ] Human-in-the-loop 的精细反馈机制
- [ ] 多 Agent 协作 + 动态角色博弈

---

## 🔗 相关链接汇总

| 项目 | 链接 |
|------|------|
| Repo-To-Skill (DisCo) | https://github.com/VectorSpaceLab/AREX-Skill |
| Skill2Env (NVIDIA) | https://github.com/NVlabs/Skill2Env |
| AutoBenchmark (Meta) | https://facebookresearch.github.io/RAM/blogs/autobench/ |
| DESIGNER | attention-is-all-i-need.github.io/Design-Logic-Reasoning |
