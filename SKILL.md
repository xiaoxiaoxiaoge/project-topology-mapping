---
name: project-topology-mapping
description: 在开始编码前生成项目的完整、持久化结构图（含目录、组件、模块差异矩阵、HTTP/状态/守卫数据流、前后端定位速查表），输出到 .project-topology.md 并保存项目摘要到记忆。适用于前端/后端/全栈/HarmonyOS/移动端/小程序/桌面端项目。当用户提到「接手新项目」「先理解项目结构」「开始做 X 功能前」时触发。
---

# 项目拓扑映射 (Project Topology Mapping)

> **核心原则：扫描一次，持久化，多次复用。**

在开始实现之前，生成项目的**完整、持久化**结构图。让 AI 清晰了解代码位置、模块边界、依赖关系和数据流——减少重复扫描的 token 消耗，提高代码定位准确性。

---

## ⚠️ 前置要求（必读）

### 1. 工作目录

**所有命令假设你在被扫描项目的 git 根目录执行。**

```bash
# 验证当前是不是项目根
cd <your-project>           # 先 cd 进去
git rev-parse --show-toplevel  # 应该输出 <your-project> 绝对路径
```

如果不是 git 仓库（罕见），需要先 `git init` 或手动 cd 到项目根。

> **如果工作目录不对，命令会找不到文件或产生错误结果。**

### 2. 推荐运行环境

| 平台 | 推荐 Shell | 备注 |
|------|----------|------|
| macOS / Linux | `bash` / `zsh` | 原生支持 |
| Windows | **Git Bash** 或 **WSL** | **强烈推荐** |
| Windows (备选) | PowerShell 7+ | 见下方映射表 |

> **Windows 原生 PowerShell 默认无 `head` / `tail` / `find` 等命令。** 强烈建议安装 [Git for Windows](https://git-scm.com/download/win) 后使用 **Git Bash**，或在 WSL 下运行。

### 3. 跨平台命令映射（速查）

| 用途 | Git Bash / WSL | PowerShell |
|------|---------------|-----------|
| 取前 N 行 | `head -n N` | `Select-Object -First N` |
| 取后 N 行 | `tail -n N` | `Select-Object -Last N` |
| 计数 | `wc -l` | `(Get-Content x \| Measure-Object -Line).Lines` |
| 递归找文件 | `find . -name "*.java"` | `Get-ChildItem -Recurse -Filter *.java` |
| 列出被 git 追踪的文件 | `git ls-files '*.java'` | `git ls-files '*.java'` *(Git for Windows 自带)*<br>无 git 时回退：`Get-ChildItem -Recurse -Filter *.java \| Select-Object -ExpandProperty FullName` |
| 文本搜索 | `grep -r "X" --include="*.java"` | `Select-String -Path "*.java" -Pattern "X" -Recurse` |
| 文本搜索（扩展正则） | `grep -E "pat1\|pat2"` | `Select-String -Pattern "pat1\|pat2"` |
| 去重排序 | `sort -u` | `Sort-Object -Unique` |
| 文本替换 | `sed -i 's/a/b/g' file` | `(Get-Content file) -replace 'a','b' \| Set-Content file` |
| 对比文件 | `diff a b` | `Compare-Object (Get-Content a) (Get-Content b)` |
| 字符串拼接 | `cat a b > c` | `Get-Content a, b \| Set-Content c` |
| 写文件 | `cat > f <<EOF ... EOF` | `Set-Content -Path f -Value @"..."@` |
| 删文件 | `rm f` | `Remove-Item f`（建议用 mavis-trash） |

> **本 skill 优先使用 `git ls-files` / `git grep` 等 Git 原生命令**，所有平台（macOS/Linux/Windows Git Bash/WSL）都自带 Git，跨平台一致性最好。PowerShell 7+ 也自带 `git` 命令（通过 Git for Windows），所以 `git ls-files` 在 Windows 上**优先**用 `git` 本身，**不**走 `Get-ChildItem` fallback。

> **关于正则：本 skill 大量用 `git grep "pat1\|pat2"` 做 OR。** `git grep` 默认就是**扩展正则（ERE）**，所以 `\|` 在这里就是 OR 字符。**但**为了让命令更明确且可移植到非 git 场景（如 `grep -E`），关键 OR 模式建议显式加 `-E`：`git grep -E "pat1|pat2"`。本 skill 内部混用了两种写法，新写命令推荐统一 `-E` 形式。

---

## 🎯 何时使用 / 不使用

### ✅ 应该使用

- **新项目接手** — 第一次接触代码库，需要快速建立全局认知
- **跨会话恢复上下文** — 之前的会话扫过，现在需要延续
- **项目结构大改后** — 目录重组、新增微服务、引入新框架
- **新成员 onboarding** — 让他/她 5 分钟看完一个项目的全貌
- **跨服务定位** — 全栈项目要从前端页面找到后端微服务
- **重构规划** — 识别高变异(差异)区域和关键依赖

### ❌ 不应该使用

- 只改一两个文件（性价比太低）
- 项目 < 50 行代码（杀鸡用牛刀）
- 用户明确要"快速 hack 一下"
- 项目已经扫过且时间戳 < 24h 且 git 无变更

---

## 📊 扫描深度选择

根据项目规模和 token 预算选择一档（影响阶段 2-3 的命令数量）：

| 档位 | 适用 | 扫描层级 | 预计 token |
|------|------|---------|-----------|
| **快速** | 小项目 / demo / 单文件改动 | L1 目录 + L2 入口 | ~2k |
| **标准** | 中型项目 / 日常接手 | L1-L3 + 核心数据流 | ~8k |
| **深度** | 大型 / 全栈 / 微服务 | L1-L4 + 完整数据流 + 定位速查 | ~20k+ |

**默认推荐「标准」**。用户主动要求"快"或"细"时切换。

### 按 Phase 的 token 预算分解（标准档）

| Phase | 主要内容 | 预估 token |
|-------|---------|-----------|
| 1 项目类型检测 | 特征文件 + 框架识别 | ~0.5k |
| 2 结构扫描 | 目录树 + 引用关系 + 差异点预检 | ~2k |
| 3 数据流分析 | HTTP/状态/守卫/核心页面（按差异度采样） | ~3k |
| 4 生成拓扑 | 组装上述内容为 .project-topology.md | ~0.5k |
| 5 存储到记忆 | 写 ~/.claude/memory/projects/<name>.md | ~0.5k |
| 6 完整性校验 | checklist 逐项确认 | ~0.5k |
| 7 差异识别 | 同类模块差异矩阵（仅在"高度/极度差异"时执行） | ~1k |
| **合计（标准）** | | **~8k** |

> 快速档：跳过 Phase 7、Phase 3 减半、Phase 2 只到 L2 → ~2k。
> 深度档：Phase 2 完整到 L4、Phase 3 全部采样、Phase 7 强制执行 → ~20k+。

---

## ⚠️ 重要警告

**如果未完成拓扑扫描就开始实现，是在盲目工作。**

本技能的每个步骤都是**必选项**，不是可选项。数据流分析是**必须包含**的部分。

---

## 📁 文件结构

```
project-topology-mapping/
├── SKILL.md                         # 本文件（主索引 + 触发清单）
├── README.md                        # 项目说明（码云/GitHub 展示用）
├── CHANGELOG.md                     # 变更日志
├── CONTRIBUTING.md                  # 贡献指南
├── LICENSE                          # MIT
│
├── phases/                          # 7 个执行阶段（每个文件一个阶段）
│   ├── phase1_detection.md          # 阶段 1：项目类型检测
│   ├── phase2_structure.md          # 阶段 2：结构扫描 + 差异点预检
│   ├── phase3_dataflow.md           # 阶段 3：数据流分析
│   ├── phase4_output.md             # 阶段 4：生成拓扑文件
│   ├── phase5_memory.md             # 阶段 5：存储到记忆
│   ├── phase6_validation.md         # 阶段 6：完整性校验
│   └── phase7_differences.md        # 阶段 7：同类模块差异识别
│
├── types/                           # 各类型项目检测命令（Single Source of Truth）
│   ├── frontend.md                  # 前端（React/Vue/Angular）
│   ├── backend.md                   # 后端（Java/Go/Python/Node/.NET/Rust）
│   ├── mobile.md                    # 移动端（Android/iOS/RN/Flutter）
│   ├── miniprogram.md               # 小程序（微信/支付宝/抖音/百度）
│   ├── desktop.md                   # 桌面端（Electron/Tauri）
│   └── harmonyos.md                 # HarmonyOS
│
├── templates/                       # 输出模板
│   ├── frontend_output.md
│   ├── backend_output.md
│   ├── mobile_output.md
│   ├── miniprogram_output.md
│   ├── desktop_output.md
│   └── harmonyos_output.md
│
├── examples/                        # 示例输出
│   └── sample-topology.md           # 真实扫描示例
│
└── .gitee/ + .github/               # Issue / PR 模板
```

---

## 📋 执行清单

| 阶段 | 名称 | 产出 | 关键动作 |
|------|------|------|---------|
| 1 | 项目类型检测 | 项目类型 + 检测策略 | 详见 [phase1_detection.md](./phases/phase1_detection.md) |
| 2 | 结构扫描 + 差异点预检 | 目录结构 + 组件关系 + 变异程度 | 详见 [phase2_structure.md](./phases/phase2_structure.md) |
| 3 | 数据流分析 | HTTP/状态/守卫/核心页面流 | 详见 [phase3_dataflow.md](./phases/phase3_dataflow.md) |
| 4 | 生成拓扑文件 | `.project-topology.md` | 详见 [phase4_output.md](./phases/phase4_output.md) |
| 5 | 存储到记忆 | `~/.claude/memory/projects/<name>.md` | 详见 [phase5_memory.md](./phases/phase5_memory.md) |
| 6 | 完整性校验 | checklist 全绿 | 详见 [phase6_validation.md](./phases/phase6_validation.md) |
| 7 | 同类模块差异识别 | 模块差异表 | 详见 [phase7_differences.md](./phases/phase7_differences.md) |

### 全栈项目额外要求

- 阶段 1: 同时检测前端+后端特征
- 阶段 2: 服务端深度扫描 (L1-L4)
- 阶段 3: 同步生成前后端定位速查
- 阶段 4: 分离生成 `.project-topology-frontend.md` / `.project-topology-backend.md`
- 阶段 5: 包含微服务定位速查

---

## 🗂️ 项目类型速查表

| 项目类型 | 关键检测文件/目录 | 差异点检测重点 | 检测命令 |
|---------|------------------|---------------|---------|
| **前端 (React/Vue)** | `package.json`, `src/`, `pages/` | 页面 config.ts, 组件差异 | [types/frontend.md](./types/frontend.md) |
| **前端 (Angular)** | `package.json`, `src/app/` | module.ts, component.ts | [types/frontend.md](./types/frontend.md) |
| **后端 (Java/Spring)** | `pom.xml`, `src/main/java` | Controller 方法数, Service 逻辑 | [types/backend.md](./types/backend.md) |
| **后端 (Go)** | `go.mod`, `cmd/`, `pkg/` | Handler 路由, Middleware 链 | [types/backend.md](./types/backend.md) |
| **后端 (Node/TS)** | `package.json`, `src/`, `routes/` | Router 中间件, Controller 方法 | [types/backend.md](./types/backend.md) |
| **后端 (Python)** | `requirements.txt`, `views.py` | View 方法, Model 字段差异 | [types/backend.md](./types/backend.md) |
| **后端 (.NET)** | `*.csproj`, `Controllers/` | Controller, Service, Middleware | [types/backend.md](./types/backend.md) |
| **后端 (Rust)** | `Cargo.toml`, `src/`, `handlers/` | Handler 函数, Trait 实现 | [types/backend.md](./types/backend.md) |
| **移动端 (Android)** | `build.gradle`, `app/src/main/java` | Activity, Fragment, ViewModel | [types/mobile.md](./types/mobile.md) |
| **移动端 (iOS)** | `*.xcodeproj`, `Sources/` | ViewController, Storyboard | [types/mobile.md](./types/mobile.md) |
| **移动端 (RN/Flutter)** | `package.json`/`pubspec.yaml` | 屏幕组件, 路由 | [types/mobile.md](./types/mobile.md) |
| **移动端跨端 (uni-app/Taro)** | `src/manifest.json` | 条件编译, 平台差异 | [types/mobile.md](./types/mobile.md) |
| **小程序 (微信)** | `app.json`, `pages/`, `miniprogram/` | 页面生命周期, 组件 | [types/miniprogram.md](./types/miniprogram.md) |
| **桌面端 (Electron)** | `package.json`, `main/`, `renderer/` | Main/Renderer 进程, IPC | [types/desktop.md](./types/desktop.md) |
| **桌面端 (Tauri)** | `src-tauri/Cargo.toml`, `tauri.conf.json` | Rust Command, Webview | [types/desktop.md](./types/desktop.md) |
| **HarmonyOS** | `entry/src/main/ets`, `*.ets` | 页面路由, Native 调用 | [types/harmonyos.md](./types/harmonyos.md) |
| **全栈** | 同时存在前端+后端特征 | 前端+后端差异点都要检测 | 多个 types/ 组合使用 |

---

## ⚠️ 常见错误

| 错误 | 结果 | 修复 |
|---------|--------|---------|
| 跳过项目类型检测 | 用前端命令检测后端，漏掉模块 | 阶段 1 必须先检测 |
| 跳过数据流分析 | 拓扑不完整，后续盲目 | 阶段 3 是必选项 |
| 每次都重新扫描 | 浪费 token | 先检查时间戳和 git 状态 |
| 不存储到记忆 | 重复扫描 | 阶段 5 必须更新记忆 |
| 缺少完整性校验 | 遗漏重要章节 | 阶段 6 逐项检查 |
| **同类模块不同流程只分析一个** | 拓扑错误 | 阶段 7 必须执行 |
| **不看项目类型用错检测命令** | 后端代码被忽略 | 阶段 1 必须检测全栈 |
| **跳过差异点预检** | 采样偏差 | 阶段 2 必须做差异点预检 |

---

## 🔌 集成点（可选）

本 skill 是基础能力层。可与以下系统组合（不依赖）：

- **记忆系统** — 阶段 5 的产出会被记忆系统读取
- **任务规划 skill** — 上游 brainstorm 完成后调用本 skill
- **代码检索 skill** — 下游结合本 skill 的定位速查做精准定位

如果你安装了 ClaudeCode `superpowers` 插件，可与 `superpowers:brainstorming` / `superpowers:executing-plans` 串联。

---

## 📤 输出文件

| 文件 | 用途 | 是否 gitignore |
|------|------|--------------|
| `.project-topology.md` | 完整拓扑（含数据流） | 是 |
| `.project-topology-frontend.md` | 前端拓扑（多端项目） | 是 |
| `.project-topology-backend.md` | 后端拓扑（多端项目） | 是 |
| `~/.claude/memory/projects/<name>.md` | 项目参考 + 定位速查 | 否（记忆） |

---

## 🛠️ 平台兼容性

### 推荐运行环境

| 平台 | 推荐 Shell | 备注 |
|------|----------|------|
| macOS / Linux | `bash` / `zsh` | 原生支持 |
| Windows | **Git Bash** 或 **WSL** | 强烈推荐 |
| Windows (备选) | PowerShell 7+ | 用下方映射表 |

### 常用命令映射（Git Bash ↔ PowerShell）

| 用途 | Git Bash | PowerShell |
|------|---------|-----------|
| 取前 N 行 | `head -N` | `Select-Object -First N` |
| 取后 N 行 | `tail -N` | `Select-Object -Last N` |
| 计数 | `wc -l` | `(Get-Content x \| Measure-Object -Line).Lines` |
| 递归找文件 | `find . -name "*.java"` | `Get-ChildItem -Recurse -Filter *.java` |
| 文本搜索 | `grep -r "X" --include="*.java"` | `Select-String -Path "*.java" -Pattern "X" -Recurse` |
| 写文件 | `cat > f <<EOF ... EOF` | `Set-Content -Path f -Value @"..."@` |
| 删文件 | `rm f` | `Remove-Item f`（建议用 mavis-trash） |

### 跨平台友好的命令写法

本 skill 的所有命令优先用以下写法（无需 `head` / `tail`）：

```bash
# 列出前 N 个（用 git 自带）
git ls-files '*.java' | head -N                # Git Bash OK
git ls-files '*.java' | Select-Object -First N # PowerShell

# 跨平台：直接管道到 head/Select-Object 二选一
git ls-files '*.java' | head -N
```

### 如果你看到 `head -N` 之类的旧命令

升级到本 skill 最新版即可。**`head` 在 Windows 原生 cmd/PowerShell 不存在**。

> Windows 用户如果不想装 Git Bash / WSL，可以**临时用 [win-bash](https://github.com/adoxa/ansicon) 之类的 portable bash**，或者把命令手写到 `package.json` 的 `scripts` 里用 `npm` 跑。

---

**违反此技能：** 如果在未完成本次会话项目的完整拓扑（包括数据流和项目类型）的情况下开始实现，你是在盲目工作——返回并先完成扫描。
