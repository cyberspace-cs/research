# REA (Reverse Engineer Anything) 深度调研

> **一句话定位**：给 AI Coding Agent 装上一套"逆向工程工具箱"，让 Agent 自己拆解任何软件——没有源码也能看懂功能怎么实现，每个结论都带证据链。

- **GitHub**: https://github.com/morluto/rea （16.8万+ ⭐，MIT）
- **npm**: `rea-agents` · **官网**: https://rea.tools/
- **爆火时间**: 2026年10月（6 天涨 10 万星，登顶 GitHub Trending）
- **作者**: morluto

---

## 1. 它为什么爆火？

2026年10月，REA 在 GitHub 上创造了增长奇迹：

| 时间 | Star 数 |
|------|---------|
| 10月6日 | < 1万 |
| 10月10日 | ~7.2万 |
| 10月12日 | 11万+ |
| 10月13日 | 16.8万 |

对比同期爆火项目：年初的 OpenClaw 第 3 天累计 6 万星，前阵的 Jev 前 3 天下载量 3.5 万。**REA 三天接近 8 万星，一周破 12 万。**

为什么这么火？
- 它直接击中了开发者的刚需：**"看到别人 App 里的好功能，想知道它是怎么实现的"**
- 它让"闭源护城河"变得可笑——无需源码，Agent 就能克隆一切软件
- 它把专业逆向工具（Ghidra/Hopper/IDA/JADX）封装成一行命令，让普通开发者也能用

> 注意：REA 本身**不是大模型**，而是**工具链**。它通过 MCP 给 Claude Code / Codex / Cursor 等 Agent 装上"手"和"眼睛"。

---

## 2. 核心架构

```
┌─────────────────────────────────────────────────────┐
│               你的 Coding Agent                     │
│        (Claude Code / Codex / Cursor / Gemini CLI)  │
└──────────────────────┬──────────────────────────────┘
                       │ MCP (Model Context Protocol)
┌──────────────────────▼──────────────────────────────┐
│                    REA                              │
│  ┌──────────────────────────────────────────────┐  │
│  │            MCP Server（工具注册表）           │  │
│  │  · 41 个原生检查工具                         │  │
│  │  · 14 个调查工作流                           │  │
│  └──────┬───────────────┬───────────────┬───────┘  │
│         │               │               │          │
│  ┌──────▼─────┐  ┌──────▼──────┐  ┌─────▼────────┐ │
│  │ Native 层  │  │ 应用层      │  │  运行时层     │ │
│  │ Hopper     │  │ JS/Electron │  │ Frida 插桩    │ │
│  │ Ghidra     │  │ .NET 程序集 │  │ 进程捕获      │ │
│  │ IDA Pro    │  │ Android APK │  │ 浏览器观察    │ │
│  │ JEB        │  │ 固件        │  │ 网络抓包      │ │
│  └────────────┘  └─────────────┘  └───────────────┘ │
│                                                     │
│  统一输出：Evidence（证据链）+ Limitations（未知项）  │
└─────────────────────────────────────────────────────┘
```

**三层设计**：

| 层 | 职责 | 关键点 |
|----|------|--------|
| **Agent 层** | 描述目标 → 追问 → 解释 → 实现 → 测试 | 用自然语言驱动 |
| **REA 核心** | MCP Server + CLI，工具注册、工作流编排 | 41 工具 / 14 工作流 |
| **分析引擎层** | 对接专业逆向工具 | Hopper / Ghidra / IDA / JADX / Binwalk / Frida |

**证据链机制（最有价值的设计）**：
- 每个结论都附带 `evidence_id`（可溯源到具体反编译代码/汇编/引用）
- 每个分析都明确标注 `limitations` / `unknowns`（不假装全懂）
- Agent 可以用证据追问、对比、导出分析包

---

## 3. 工作原理

```
你（用户）：理解一下 Notes 应用里的搜索是怎么做的，给我证据，然后帮我做一个类似的
    ↓
Coding Agent：调用 REA 的 MCP 工具
    ↓
REA：检查目标应用 → 反编译/拆包 → 追踪相关代码路径 → 返回代码+引用+未知项
    ↓
Coding Agent：基于证据解释功能原理
    ↓
Coding Agent：为你写出实现代码并运行测试
```

典型调查工作流（14 个之一）：
1. **识别目标**：确定 App 的二进制/包/目录位置
2. **静态分析**：反编译、提取模块、导入、资源
3. **追踪调用路径**：从入口函数追到具体实现（`trace_call_path`）
4. **运行时观察**（可选）：Frida 插桩、进程捕获、浏览器观察
5. **交叉验证**：静态代码 vs 运行时行为比对
6. **证据归集**：导出 `Evidence Bundle`，形成可审计的分析报告

---

## 4. 能分析什么（19+ 类目标）

| 目标类型 | REA 返回什么 | 依赖工具 |
|---------|-------------|---------|
| **原生二进制** (Mach-O/ELF/PE) | 伪代码、汇编、字符串、符号、调用引用 | Hopper / Ghidra / IDA |
| **Go 二进制** | 编译器版本、模块、替换、构建设置 | 内置静态检查 |
| **WASM** | 校验、段、精确 WAT 导入/导出 | WABT |
| **EVM 字节码** | 分发选择器、字节偏移、参数推断 | 内置 |
| **SQLite 数据库** | 表、列、索引、视图、触发器、行数据 | 内置 |
| **JavaScript / Electron** | 模块、导入、source map、路由、IPC | Node.js（免原生引擎） |
| **网站** | 页面结构、脚本、网络观察、截图 | Chrome 系浏览器 |
| **网络抓包 (HAR/SAZ)** | 请求、响应、暴露的 payload、源码位置 | 内置 / mitmproxy |
| **.NET 程序集** | 元数据、CIL 指令、原生依赖 | 静态即可 |
| **Android APK** | Manifest、类、反编译方法、引用 | JADX + JDK |
| **Android 设备** | 设备、包、进程、日志、截图 | adb |
| **JEB 项目** | 引擎单元、类型/方法伪代码 | JEB 客户端 |
| **Apktool 资源** | 解码 Manifest、字符串表、语言、版本 | Apktool |
| **Flutter App** | Dart 快照哈希、ABI 库摘要 | 静态 APK 解析 |
| **固件** | 分区、提取结果、原生分析交接 | Binwalk / Unblob |
| **pcap 抓包** | 端点、DNS、TLS SNI、协议分布、HTTP | tshark |
| **逻辑信号** | 解码器注释、采样范围、诊断 | sigrok-cli |
| **包/资源** | 文件清单、摘要、plist、Apple 包解剖 | 内置 |
| **进程行为** | 终端输出、交互、退出、文件系统观察 | PTY |

---

## 5. 核心代码架构（src 目录）

REA 的代码库非常工程化（1698 个源文件，TypeScript）：

```
src/
├── application/     # 应用层：分析编排、证据管理、能力清单
│   ├── CallPathTracing.ts          # 调用路径追踪
│   ├── EvidenceBundleCommands.ts   # 证据包导出
│   ├── AnalysisProvider.ts         # 分析提供者抽象
│   └── BrowserObservationService.ts # 浏览器观察
├── android/         # Android 适配
│   ├── JadxProvider.ts             # JADX 反编译提供者
│   ├── JadxMcpTransport.ts         # JADX MCP 传输
│   └── AndroidTargetSnapshot.ts    # 目标快照
├── adb/             # ADB 设备分析
├── apktool/         # Apktool 资源解码
├── native/          # 原生二进制分析引擎
├── bridge/          # 与外部工具的桥接层
├── frida/           # Frida 运行时插桩
└── tests/           # 114+ 测试
```

**核心设计模式**：
1. **Provider 抽象**：每个分析引擎（Ghidra/Hopper/IDA/JADX）都是一个 Provider，实现统一接口。换引擎不换上层逻辑。
2. **Evidence 一等公民**：所有工具返回标准 `Evidence` 记录（含 `evidence_id`、`normalized_result`、`raw_result`、`analysis_profile`），可被下游工具直接消费（如 `analyze_function` 的证据可直接传给 `compare_functions`）。
3. **Session 管理**：分析结果在会话中持久化，可通过 `get_evidence_bundle` 导出。
4. **安全护栏**：权限检查、敏感路径保护、URL 校验、危险操作拦截。

---

## 6. 安装与快速上手

### 安装（一条命令）

```bash
npx rea-agents setup
```

- 选择你的 Agent（支持 Claude Code / Codex / Cursor / Gemini CLI / Grok Build）
- 审查并批准配置变更（自动备份原配置）
- 重启 Agent 生效
- 原生二进制分析可选配 Hopper / Ghidra / IDA；静态 JS 分析无需任何引擎

### 终端直接使用

```bash
# 分析一个解包后的 JS/Electron 应用
npx -y rea-agents@latest analyze-javascript-application /绝对/路径/应用 --json

# 安装全局命令
npm install --global rea-agents
rea --help
```

### 对 Agent 说一句话

```
理解 Notes 应用里的搜索是怎么工作的，给我证据，然后在我们的项目里做一个类似功能。
```

---

## 7. 逆向示例

### 示例 A：JavaScript / Electron 应用分析（完整演示）

对任意解包后的 Electron 应用目录或 ASAR：

```bash
npx -y rea-agents@latest analyze-javascript-application /path/to/app --json
```

返回：**模块列表、imports、Electron 边界（主进程/渲染进程）、IPC 通道及其证据**。

之后可以让 Agent 继续追问：
- "这个应用的自动更新是怎么实现的？"
- "它的本地数据库用了什么 schema？"
- "这些路由对应的页面组件在哪？"

### 示例 B：豆包 Web 端（公开可访问层面分析）

以豆包（www.doubao.com）为例，用 **浏览器公开可访问的内容**（任何用户访问网页时浏览器都会加载的资源）做技术分析，属于常规网页技术观察：

| 分析层面 | 观察内容 | 性质 |
|---------|---------|------|
| 页面结构 | HTML 骨架、DOM 组织、容器分层 | 浏览器公开渲染结果 |
| 前端脚本 | JS 模块加载、框架识别（如 React/Vue）、构建产物特征 | 公开加载资源 |
| 网络请求 | API 端点、流式响应（SSE）、鉴权方式（公开调用层面） | 自己浏览器发出的请求 |
| 产品行为 | 功能入口 → 请求 → 响应的对应关系 | 用户可见行为 |

> **法律边界（重要）**：以上仅限"自己访问网页时浏览器公开加载/展示的内容"。**不包含**：破解豆包客户端二进制、绕过付费墙/会员校验、抓取未授权数据、逆向其专有算法细节用于复制。REA 官方 DISCLAIMER 明确禁止：绕过 DRM/授权检查/反作弊、开发外挂、绕过认证、复制专有软件代码与资产。逆向工程须遵守目标软件的许可协议与当地法律。

---

## 8. 为什么值得关注（对我们的启发）

1. **Agent 能力边界的又一次扩展**：继"写代码"之后，Agent 现在能"读别人代码（二进制）"了。REA = MCP 工具链 + 证据链设计，是 Agent 工具生态的标杆案例。
2. **证据链设计值得学习**：`Evidence` 记录、`limitations` 显式标注、可审计导出——这正是我们 DIY Agent Harness 里应该有的"可验证输出"设计。
3. **Provider 抽象模式**：一个接口对接多个引擎（Ghidra/Hopper/IDA/JADX），与我们的多模型/多工具适配思路一致。
4. **产品化技巧**：一行命令安装（npx setup）、自动备份配置、多 Agent 支持、多语言 README（20+ 语言）、Skills 集成（skills.sh）——增长黑客典范。

---

## 9. 参考资料

- 仓库：https://github.com/morluto/rea
- 官网：https://rea.tools/
- npm：https://www.npmjs.com/package/rea-agents
- 文档：https://rea.tools/guides/
- Showcases：https://rea.tools/showcase/
- 36氪报道：https://m.36kr.com/p/4022639150289033
- 腾讯云分析：https://cloud.tencent.com.cn/developer/article/2759769
- OSCHINA：https://www.oschina.net/news/502941

---

## 10. 实战演示记录（2026-10-13 实测）

> 以下全部为**本机真实运行** `npx rea-agents@latest` 的结果。

### 演示 1：逆向一个 Electron 笔记应用（demo-notes-app）

构造了一个模拟 Electron 应用（main.js + preload + 渲染进程 + better-sqlite3），REA 一条命令完整重建架构：

```bash
npx rea-agents@latest analyze-javascript-application demo-notes-app --json
```

**REA 输出摘要**：

| 维度 | 值 |
|------|-----|
| BrowserWindow | 1 |
| preload 入口 | 1 |
| contextBridge API | `notesAPI`（list / create / delete）|
| IPC 通道 | 3：`notes:list`、`notes:create`、`notes:delete` |
| 主进程 handlers | 3（全部配对）|
| 渲染进程传输 | 3（与 handler 一一对应）|
| 外部依赖 | `electron`、`path`、`better-sqlite3` |
| 解析 | 5 文件 · 331 AST 节点 · 22 findings · 0 失败 |

**证据链示例**：每个 IPC handler 都带精确位置引用，如 `main:handle:notes:create:21:31-24:1`；同时标注 7 条 limitations（"静态分析未证明运行时执行"）。

### 演示 2：豆包 Web 端公开资源分析

> 合规说明：仅分析**浏览器公开加载的页面/JS 资源**（任何用户访问网页时浏览器都会获取的内容），属于常规网页技术观察，不涉及破解客户端二进制或绕过授权。

**第一步：抓取首页公开 HTML**

```bash
curl -A "Mozilla/5.0 ... Chrome/125.0" https://www.doubao.com/
```

发现：
- 静态资源托管于 `lf-flow-web-cdn.doubao.com`（字节 CDN）
- 3 个外部 JS：`lib-polyfill`、`51.chunk`、`chat`（主包 1.7MB）
- 11 个内联脚本块

**第二步：分析公开 JS 特征**

| 文件 | 大小 | 关键特征 |
|------|------|---------|
| chat.d90c4a86dd.js | 1.7 MB | `conversation`×567、`ark`×290、`SSE`×30（流式）、`bytedance` |
| 51.d0891abea5.js | 935 KB | `react`×104（前端框架）、`ark`×11、`session_id` |
| lib-polyfill | 161 KB | polyfill + SSE |

结论（公开层面）：**豆包 Web = React 前端 + 火山方舟(ark)平台 + SSE 流式会话体系**。

**第三步：用 REA 分析公开 JS bundle**

```bash
npx rea-agents@latest analyze-javascript-application doubao-web --json
```

REA 输出（2.8MB 源码 → 图结构）：

| 指标 | 值 |
|------|-----|
| AST 节点 | 870,711 |
| findings | 186 · 解析失败 0 |
| 应用图 | 109 节点 / 193 边 |
| 端点节点 | **77** |
| 存储节点 | **22** |
| 关系 | calls×98 · persists_to×51 · exposes×35 |
| 未知项 | 36（显式标注）|

REA 同时诚实标注局限：解析大 bundle 时 worker 堆内存受限、静态分析不证明运行时执行、图覆盖不完整（"unavailable facts remain explicit and must not be read as absence"）。

### 演示小结

- REA 对**应用级**目标（Electron/JS 目录）分析能力极强：IPC、preload、contextBridge、依赖全部重建
- 对**单文件 webpack bundle** 也能拆：端点、存储、调用关系可提取，但大文件受资源限制
- 证据链 + 显式未知项设计贯穿所有输出，是 Agent 工具设计的标杆

