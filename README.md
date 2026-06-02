# Project Topology Mapping

```
   ____            __     __         __  __                                  __
  / __ \__  ____  / /_   / /_  ___  / /_/ /_  ___  ____ ___  ___  _________ _/ /_____ _____
 / __/_/ / / / __ \/ __/  / __ \/ _ \/ __/ __ \/ _ \/ __ `__ \/ _ \/ ___/ __ `/ //_/ _ \/ ___/
/ /_/ / /_/ / / / / /_   / /_/ /  __/ /_/ / / /  __/ / / / / /  __/ /  / /_/ / ,< /  __/ /
\____/\__,_/_/ /_/\__/  /_.___/\___/\__/_/ /_/\___/_/ /_/ /_/\___/_/   \__,_/_/|_|\___/_/
```

> **让 AI 在开始编码前，先完整理解你的项目。**
> **扫描一次 · 持久化 · 多次复用**

[![MIT License](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.com)
[![Gitee](https://img.shields.io/badge/Gitee-推荐项目-c71d23?logo=gitee)](https://gitee.com/www_mao_com/project-topology-mapping)
[![GitHub](https://img.shields.io/badge/GitHub-镜像-181717?logo=github)](https://github.com/www_mao_com/project-topology-mapping)
[![Version](https://img.shields.io/badge/version-2.0.0-blue)](./CHANGELOG.md)

---

## 目录

- [这是什么](#这是什么)
- [为什么需要](#为什么需要)
- [核心特性](#核心特性)
- [支持的项目类型](#支持的项目类型)
- [快速开始](#快速开始)
- [执行流程](#执行流程)
- [项目结构](#项目结构)
- [使用示例](#使用示例)
- [安装与配置](#安装与配置)
- [故障排除](#故障排除)
- [贡献](#贡献)
- [版本](#版本)
- [引用](#引用)
- [许可](#许可)
- [Star / Watch / Fork](#star--watch--fork)

---

## 这是什么

`project-topology-mapping` 是一个 **Claude Code 自定义技能（Skill）**，用于在 AI 开始编码前，**自动扫描并生成项目的完整、持久化结构图**。包括：

- 📁 目录结构 + 模块地图
- 🔗 组件引用关系（Top 20 共享组件）
- 🌊 完整数据流（HTTP 请求 / 状态管理 / 路由守卫 / 核心页面）
- 🧩 前后端定位速查表（从前端页面一键定位到后端 Controller）
- 🕵️ 同类模块的差异矩阵（识别"看起来相似实则不同"的边界）
- 🔐 服务名交叉验证（防止误判微服务归属）

**核心原则：扫描一次，持久化，多次复用。**

---

## 为什么需要

| 痛点 | 传统方式 | 使用本 Skill |
|------|---------|-------------|
| AI 不懂项目 | 每次对话都要重新解释上下文 | 拓扑图一次性说明白 |
| 修改时定位难 | 搜索关键词猜测文件 | 定位速查表直接跳转 |
| 同类模块被混淆 | AI 假设所有模块逻辑相同 | 差异点检测 + 差异矩阵 |
| 后端代码被忽略 | 用前端命令检测后端 | 分类型使用不同检测策略 |
| 重复扫描 | 每次对话都重新扫描 | 持久化 + 记忆系统复用 |
| 服务名误判 | 命名相似就合并 | 三重交叉验证服务归属 |

---

## 核心特性

### 🔍 智能项目类型检测

自动识别 **前端 / 后端 / 全栈 / 移动端 / 小程序 / 桌面端 / HarmonyOS / 混合项目**，动态选择对应的检测策略。基于文件特征指纹和目录结构树双重验证，确保不漏掉任何代码模块。

### 📊 多维差异点检测

通过静态分析 + 动态采样双重机制，发现代码库中的"结构性异类"：

- 识别同类模块间的**流程步骤数差异**（如同级目录却执行不同步数的业务）
- 发现**关键组件的分布规律**（哪些组件只在特定子模块中存在）
- 对比**API 调用链路差异**（不同子模块触发的后端路由完全不同）
- 生成**模块差异矩阵**，让 AI 精准理解"看起来相似实则不同"的边界

### 🔗 前后端定位速查

构建从前端页面到后端 Controller 的**双向映射网络**：

| 维度 | 映射内容 |
|------|---------|
| 前端 → API | 每个页面调用的所有 API 路径 |
| API → 服务 | API 路径前缀 → 微服务归属 |
| 服务 → 文件 | 微服务 → 具体 Controller/Service 文件 |
| 服务间调用 | Feign Client / gRPC Service → 目标服务 → 服务名交叉验证 |

### 🕵️ 服务端深度扫描（L1-L4 四层穿透）

不止步于目录结构，深入业务核心层：

| 层级 | 前端项目扫描深度 | 后端项目扫描深度 |
|------|----------------|-----------------|
| **L1 表面** | 目录结构 | 目录结构 + 依赖配置 |
| **L2 入口** | `pages/*/*.tsx` | **所有 Controller 的路由映射** |
| **L3 业务** | `components/hooks/store` | **所有 Service 的 public 方法签名** |
| **L4 细节** | `config.ts` 差异 | **Feign 调用链、事务边界、异常处理配置** |

### 🌊 完整数据流分析

追踪数据在系统中的完整旅程：

- **HTTP 请求流** — 从前端 request utility 到后端 Controller 的完整链路
- **状态管理流** — Store/缓存层的读写策略与持久化机制
- **路由守卫流** — 安全过滤链的全路径（含权限校验、Token 刷新、跳转逻辑）
- **核心页面/Controller 数据流** — 选取代表性业务场景，绘制端到端数据流图

### ⚠️ 服务名交叉验证

微服务架构中常见"命名相似实则不同"陷阱：

- Java `@FeignClient` 的 `name` 值
- 服务名常量定义（如 `<ServiceNameConstant>`）
- Go 服务的 Eureka 注册名 / Node 服务的 `server_name`

**三重交叉验证**，防止误判服务归属。

### 📚 跨平台友好

所有检测命令都做了 Windows / macOS / Linux 跨平台适配，优先使用 `git ls-files` 等跨平台命令。详见 [SKILL.md § 平台兼容性](./SKILL.md#平台兼容性)。

---

## 支持的项目类型

| 类型 | 关键特征 | 扫描深度 | 检测命令 |
|------|---------|---------|---------|
| 前端 React/Vue | `package.json`, `src/`, `pages/` | 页面 → 组件 → hooks → services | [types/frontend.md](./types/frontend.md) |
| 前端 Angular | `package.json`, `src/app/` | Module → Component → Service | [types/frontend.md](./types/frontend.md) |
| 后端 Java/Spring | `pom.xml`, `src/main/java` | Controller → Service → Feign | [types/backend.md](./types/backend.md) |
| 后端 Go | `go.mod`, `cmd/`, `pkg/` | Handler → Middleware → Database | [types/backend.md](./types/backend.md) |
| 后端 Node/TS | `package.json` + `express`/`@nestjs`/`fastify`/`koa` | Router → Middleware → ORM | [types/backend.md](./types/backend.md) |
| 后端 Python | `requirements.txt` + Django/Flask/FastAPI | View → Model → ORM | [types/backend.md](./types/backend.md) |
| 后端 .NET | `*.csproj` + `Program.cs` | Controller → Service → EF Core | [types/backend.md](./types/backend.md) |
| 后端 Rust | `Cargo.toml` | Handler → Middleware → sqlx/diesel | [types/backend.md](./types/backend.md) |
| 移动端 Android | `app/src/main/AndroidManifest.xml` | Activity → Fragment → ViewModel | [types/mobile.md](./types/mobile.md) |
| 移动端 iOS | `*.xcodeproj` + `Info.plist` | ViewController → Storyboard → SwiftUI | [types/mobile.md](./types/mobile.md) |
| 移动端 RN/Flutter | `package.json`/`pubspec.yaml` | Screen → Widget → State | [types/mobile.md](./types/mobile.md) |
| 移动端跨端 (uni-app/Taro) | `src/manifest.json` | 条件编译 → 平台差异 | [types/mobile.md](./types/mobile.md) |
| 小程序 微信/支付宝/抖音/百度 | `app.json` + `wx.`/`my.`/`tt.`/`swan.` API | 页面 → 组件 → API 差异 | [types/miniprogram.md](./types/miniprogram.md) |
| 桌面端 Electron | `package.json` + `electron` | Main / Renderer / IPC | [types/desktop.md](./types/desktop.md) |
| 桌面端 Tauri | `src-tauri/Cargo.toml` + `tauri.conf.json` | Rust Command + Webview | [types/desktop.md](./types/desktop.md) |
| HarmonyOS | `entry/src/main/ets`, `*.ets` | ArkTS → Native (lib*.so) | [types/harmonyos.md](./types/harmonyos.md) |
| 全栈 | 同时存在多种端特征 | **每个端深度扫描都要执行** | 多个 types/ 组合使用 |

---

## 快速开始

### 安装

将 `project-topology-mapping` 目录放入 `~/.claude/skills/`：

```bash
# macOS / Linux
git clone https://gitee.com/www_mao_com/project-topology-mapping.git \
  ~/.claude/skills/project-topology-mapping

# 或从 GitHub 镜像
git clone https://github.com/www_mao_com/project-topology-mapping.git \
  ~/.claude/skills/project-topology-mapping

# Windows (PowerShell)
git clone https://gitee.com/www_mao_com/project-topology-mapping.git `
  $env:USERPROFILE\.claude\skills\project-topology-mapping
```

### 触发扫描

在 Claude Code 中输入：

```
/project-topology-mapping
```

或者用自然语言：

```
扫描当前项目，生成拓扑图
接手这个新项目，先理解结构
```

### 3 秒速览流程

```
┌─────────────────────────────────────────────────┐
│  Phase 1  项目类型检测                             │
│  ↓                                               │
│  Phase 2  结构扫描 + 差异点预检                     │
│  ↓                                               │
│  Phase 3  数据流分析（HTTP/状态/守卫/核心页面）       │
│  ↓                                               │
│  Phase 4  生成 .project-topology.md               │
│  ↓                                               │
│  Phase 5  存储到 ~/.claude/memory/projects/        │
│  ↓                                               │
│  Phase 6  完整性校验（checklist 全绿）              │
│  ↓                                               │
│  Phase 7  同类模块差异识别（高度/极度差异时）         │
└─────────────────────────────────────────────────┘
```

详见 [SKILL.md § 执行清单](./SKILL.md#执行清单)。

---

## 执行流程

### Phase 1：项目类型检测

```
检测策略选择器
├── 前端特征? → package.json / src/
├── 后端特征? → pom.xml / go.mod / requirements.txt
├── 移动端特征? → AndroidManifest.xml / *.xcodeproj / pubspec.yaml
├── 小程序特征? → app.json + wx./my./tt./swan. API
├── 桌面端特征? → electron / tauri
├── 全栈特征? → 同时存在 → 每种端检测都要执行
└── HarmonyOS 特征? → entry/src/main/ets / *.ets
```

通过多维特征向量识别，避免"误判项目类型导致代码被忽略"。

### Phase 2：结构扫描 + 差异点预检

```
目录结构树
├── L1 表面扫描
├── L2 入口定位
├── L3 业务分析
└── L4 细节提取
+
差异点预检矩阵
├── 步骤数差异检测
├── 组件分布检测
└── API 调用差异检测
```

判定差异程度：**无差异 → 轻度差异 → 高度差异 → 极度差异**，决定阶段 3 的采样策略。

### Phase 3：数据流分析

```
采样策略引擎
├── 无差异: 采样 1 个代表性模块
├── 轻度差异: 采样 2-3 个代表模块
├── 高度差异: 每个子模块都要采样
└── 极度差异: 生成完整模块差异表
+
数据流追踪
├── HTTP 请求层
├── 状态管理层
├── 路由守卫层
└── 核心页面采样
```

### Phase 4-6：拓扑生成 + 校验

```
输出文件
├── .project-topology.md          # 完整拓扑（含数据流）
├── .project-topology-frontend.md # 前端拓扑（多端项目）
└── .project-topology-backend.md  # 后端拓扑（多端项目）
+
完整性逐项校验清单
```

### Phase 7：同类模块差异识别

```
⚠️ 最易遗漏的检查点

同类模块 ≠ 逻辑相同
├── <module>/<submodule-1>: 2 步流程
├── <module>/<submodule-2>: 5 步流程
├── <module>/<submodule-3>: 文档解析专属流程
└── <module>/<submodule-4>: 视频处理专属流程

生成模块差异矩阵
| 子模块 | 步骤数 | config.ts | 核心组件 | 差异原因 |
|--------|--------|-----------|---------|---------|
```

---

## 项目结构

```
project-topology-mapping/
├── SKILL.md                    # 主索引 + 触发清单 + 平台兼容性
├── README.md                   # 本文件
├── CHANGELOG.md                # 变更日志
├── CONTRIBUTING.md             # 贡献指南
├── LICENSE                     # MIT
│
├── phases/                     # 7 个执行阶段（每个文件一个阶段）
│   ├── phase1_detection.md     # 项目类型检测
│   ├── phase2_structure.md     # 结构扫描 + 差异点预检
│   ├── phase3_dataflow.md      # 数据流分析
│   ├── phase4_output.md        # 生成拓扑文件
│   ├── phase5_memory.md        # 存储到记忆
│   ├── phase6_validation.md    # 完整性校验
│   └── phase7_differences.md   # 同类模块差异识别
│
├── types/                      # 各类型项目检测命令（Single Source of Truth）
│   ├── frontend.md             # 前端（React/Vue/Angular）
│   ├── backend.md              # 后端（Java/Go/Python/Node/.NET/Rust）
│   ├── mobile.md               # 移动端（Android/iOS/RN/Flutter）
│   ├── miniprogram.md          # 小程序（微信/支付宝/抖音/百度）
│   ├── desktop.md              # 桌面端（Electron/Tauri）
│   └── harmonyos.md            # HarmonyOS
│
├── templates/                  # 输出模板
│   ├── frontend_output.md
│   ├── backend_output.md
│   ├── mobile_output.md
│   ├── miniprogram_output.md
│   ├── desktop_output.md
│   └── harmonyos_output.md
│
├── examples/                   # 示例输出
│   └── sample-topology.md      # 真实扫描示例
│
└── .gitee/ + .github/          # Issue / PR 模板
```

---

## 使用示例

### 示例 1：全栈 Java + Vue 项目

```bash
# 在 Claude Code 中
> /project-topology-mapping

# AI 输出
检测到的项目类型: 全栈 (Vue 3 + Java Spring Boot)
扫描深度: 标准
拓扑文件: .project-topology.md（主索引）
         .project-topology-frontend.md
         .project-topology-backend.md
记忆文件: ~/.claude/memory/projects/my-project.md
模块差异: <submodule-1> 与 <submodule-2> 步骤数差异大
下一步: 可以基于此拓扑开始实现
```

完整扫描示例见 [examples/sample-topology.md](./examples/sample-topology.md)。

### 示例 2：纯前端 React 项目

```bash
> /project-topology-mapping

# AI 输出
检测到的项目类型: 前端 (React 18 + TypeScript)
扫描深度: 快速
拓扑文件: .project-topology.md
记忆文件: ~/.claude/memory/projects/my-frontend.md
```

### 示例 3：移动端 Flutter 项目

```bash
> /project-topology-mapping

# AI 输出
检测到的项目类型: 移动端 (Flutter)
扫描深度: 标准
拓扑文件: .project-topology.md
关键依赖: flutter_bloc, dio, hive
```

---

## 安装与配置

### 系统要求

- Claude Code CLI（最新版本）
- Git（用于 `git ls-files` 等跨平台命令）
- macOS / Linux 任意 Shell
- Windows 推荐 Git Bash 或 WSL

### 平台兼容性

| 平台 | 推荐 Shell | 备注 |
|------|----------|------|
| macOS / Linux | `bash` / `zsh` | 原生支持 |
| Windows | **Git Bash** 或 **WSL** | 强烈推荐 |
| Windows (备选) | PowerShell 7+ | 用 [SKILL.md § 平台映射表](./SKILL.md#平台兼容性) |

### 升级

```bash
cd ~/.claude/skills/project-topology-mapping
git pull
```

### 卸载

```bash
# 仅删除 skill 目录
rm -rf ~/.claude/skills/project-topology-mapping

# 同时清理该项目已生成的拓扑文件
cd <your-project>
rm -f .project-topology*.md

# 同时清理记忆文件
rm -f ~/.claude/memory/projects/<your-project>.md
```

### 配置

本 skill **不需要任何配置**。所有检测命令基于项目自身结构自动调整。

如果想自定义输出位置，可在执行时告诉 AI：

```
/project-topology-mapping 输出到 docs/topology.md
```

---

## 故障排除

### Q: `head` 命令在 Windows PowerShell 不存在

A: 这是 Windows 原生 shell 的限制。**强烈推荐安装 Git for Windows 并使用 Git Bash**，或在 WSL 下运行。

或者用 PowerShell 7+ 并参考 [SKILL.md § 平台映射表](./SKILL.md#平台兼容性) 中的等价命令。

### Q: 扫描很慢，怎么加速？

A:
- 选择**快速**扫描深度（只到 L1 + L2）
- 大型项目先用 `git ls-files` 限制扫描范围
- 关闭实时 linting / 格式化插件

### Q: 拓扑文件没有自动生成

A: 检查以下几点：
- 是否在 git 仓库根目录执行（用 `git rev-parse --show-toplevel` 验证）
- 是否有写入权限
- 阶段 1-7 是否全部完成

### Q: 多端项目只生成了单一文件

A: 在阶段 1 显式选择"分离生成"。详见 [SKILL.md § 多端项目文件分离](./SKILL.md)。

### Q: 服务名交叉验证不准确

A: 完整的验证命令在 [types/backend.md § 服务名交叉验证](./types/backend.md#服务名交叉验证)。请确保：
- 已收集所有 `@FeignClient` 的 `name` 值
- 已收集所有服务名常量定义
- 已收集所有目标服务的 `server_name`

### Q: 找不到 .project-topology.md

A: 该文件被项目的 `.gitignore` 忽略（推荐做法）。它存在但**不会被 git 追踪**。如需分享给团队，可单独 `git add -f .project-topology.md`。

### Q: 重新扫描条件是什么？

A: 详见 [phases/phase5_memory.md § 重新扫描条件](./phases/phase5_memory.md#重新扫描条件)。简言之：

- 项目类型变化 / 新增微服务
- 数据流关键文件变化（`request.ts` / `store` / 路由守卫 / SecurityConfig）
- 距离上次扫描 > 7 天
- git 变更 > 5 个文件且都在关键路径

### Q: 怎么贡献？

A: 见 [CONTRIBUTING.md](./CONTRIBUTING.md)。欢迎：
- 提 Issue 报告 bug
- 提 PR 增加新的 `types/<your-type>.md`（如 KMP、Qt、SmartTV）
- 翻译 README / SKILL.md
- 补充 examples/

---

## 贡献

欢迎任何形式的贡献！详见 [CONTRIBUTING.md](./CONTRIBUTING.md)。

- 🐛 [提 Bug](https://gitee.com/www_mao_com/project-topology-mapping/issues)
- 💡 [提需求](https://gitee.com/www_mao_com/project-topology-mapping/issues)
- 🔧 [提 PR](https://gitee.com/www_mao_com/project-topology-mapping/pulls)
- 📖 [完善文档](https://gitee.com/www_mao_com/project-topology-mapping/wiki)

行为准则：[CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)

---

## 版本

详见 [CHANGELOG.md](./CHANGELOG.md)。

当前版本：**v2.0.0**（2026-06-01）— 见 changelog 了解重大变更。

---

## 引用

如果你在论文、博客、工具链中引用了本 skill，请使用以下格式：

```bibtex
@software{project_topology_mapping_2026,
  author = {www_mao_com},
  title = {Project Topology Mapping: A Claude Code Skill for Project Structure Scanning},
  year = {2026},
  url = {https://gitee.com/www_mao_com/project-topology-mapping},
  version = {2.0.0}
}
```

或在正文中：

> 本项目使用 [Project Topology Mapping](https://gitee.com/www_mao_com/project-topology-mapping) 生成项目拓扑。

---

## 许可

[MIT License](./LICENSE) © 2026 www_mao_com

---

## Star / Watch / Fork

如果本 skill 对你有帮助：

- ⭐ **Star** — 关注项目进展
- 👁️ **Watch** — 接收新版本通知
- 🍴 **Fork** — 二次开发并提 PR
- 📢 **分享** — 推荐给同事 / 朋友圈

码云项目页：[gitee.com/www_mao_com/project-topology-mapping](https://gitee.com/www_mao_com/project-topology-mapping)
