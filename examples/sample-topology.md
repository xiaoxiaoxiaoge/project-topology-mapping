# 项目拓扑 - Project Topology Mapping Skill

> **本文件是 `project-topology-mapping` skill 的自指示例** — 即"用本 skill 扫描本 skill 自身"得到的输出。
> 用途：作为 worked example 展示 skill 的输出格式与质量基准。

**生成时间:** 2026-06-01
**项目名称:** project-topology-mapping
**项目类型:** Claude Code Skill 文档项目
**主要语言:** Markdown
**框架:** N/A（文档项目，无运行时框架）
**扫描深度:** 快速
**版本:** v2.0.0

---

## 1. 项目概述

`project-topology-mapping` 是一个 Claude Code 自定义技能，用于生成被扫描项目的完整、持久化结构图。

### 核心原则

- **扫描一次，持久化，多次复用**
- 减少重复扫描的 token 消耗
- 提高代码定位准确性

---

## 2. 目录结构

```
project-topology-mapping/
├── SKILL.md                         # 主索引
├── README.md                        # 项目说明
├── CHANGELOG.md                     # 变更日志
├── CONTRIBUTING.md                  # 贡献指南
├── LICENSE                          # MIT
│
├── phases/                          # 7 个执行阶段
│   ├── phase1_detection.md
│   ├── phase2_structure.md
│   ├── phase3_dataflow.md
│   ├── phase4_output.md
│   ├── phase5_memory.md
│   ├── phase6_validation.md
│   └── phase7_differences.md
│
├── types/                           # 检测命令（Single Source of Truth）
│   ├── frontend.md
│   ├── backend.md
│   ├── mobile.md
│   ├── miniprogram.md
│   ├── desktop.md
│   └── harmonyos.md
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
│   └── sample-topology.md           # ← 本文件
│
└── .gitee/ + .github/               # Issue / PR 模板
```

---

## 3. 模块地图

| 模块 | 类型 | 文件数 | 职责 |
|------|------|-------|------|
| 入口 | 文档 | 1 | SKILL.md 主索引 |
| 阶段 | 文档 | 7 | 7 阶段执行文件 |
| 类型 | 文档 | 6 | 6 种项目类型的检测命令 |
| 模板 | 文档 | 6 | 6 种项目类型的输出模板 |
| 示例 | 文档 | 1 | 自指示例 |
| 社区 | 配置 | 4 | Issue 模板 + 贡献指南 |

---

## 4. 组件关系图

### 4.1 文档层级

```
SKILL.md (主索引)
    │
    ├─→ phases/  (7 阶段执行)
    │     │
    │     ├─ phase1_detection.md ─────→ types/  (选哪个)
    │     ├─ phase2_structure.md ─────→ types/  (跑命令)
    │     ├─ phase3_dataflow.md  ─────→ types/  (数据流)
    │     ├─ phase4_output.md    ─────→ templates/  (生成模板)
    │     ├─ phase5_memory.md    ─────→ ~/.claude/memory/
    │     ├─ phase6_validation.md ────→ (checklist 校验)
    │     └─ phase7_differences.md ───→ (识别差异)
    │
    ├─→ types/    (6 种类型，single source of truth)
    │     │
    │     ├─ frontend.md
    │     ├─ backend.md
    │     ├─ mobile.md
    │     ├─ miniprogram.md
    │     ├─ desktop.md
    │     └─ harmonyos.md
    │
    └─→ templates/  (6 种类型输出模板)
          │
          └─ 与 types/ 一一对应
```

### 4.2 共享文件被引用情况

| 文件 | 引用位置数 | 引用方 |
|------|----------|--------|
| `types/backend.md` | 3 | phase2_structure.md, phase3_dataflow.md, phase4_output.md |
| `types/frontend.md` | 2 | phase2_structure.md, phase3_dataflow.md |
| `SKILL.md` | 1 | README.md |
| `README.md` | 1 | SKILL.md |
| `CHANGELOG.md` | 1 | README.md |
| `CONTRIBUTING.md` | 1 | README.md |

### 4.3 依赖关系示例

```
用户 → /project-topology-mapping
       ↓
       SKILL.md (路由到 phases)
       ↓
       phases/phase1-7
       ↓
       types/*.md (提供命令)
       ↓
       templates/*.md (提供模板)
       ↓
       输出 .project-topology.md
```

---

## 5. 数据流图

### 5.1 HTTP 请求数据流

> 本 skill 自身是文档项目，**无运行时 HTTP 数据流**。
> 此处展示 skill 的"逻辑数据流"作为对照。

```
用户输入 "/project-topology-mapping"
   ↓
Claude Code 加载 SKILL.md
   ↓
SKILL.md 路由到 phase1_detection.md
   ↓
phase1 检测项目类型 → 选对应 types/<type>.md
   ↓
phase2-3 加载 types/<type>.md 的检测命令
   ↓
phase4 加载 templates/<type>_output.md 的输出模板
   ↓
phase5 写入 .project-topology.md + 记忆文件
   ↓
phase6 逐项校验 checklist
   ↓
phase7（如需）生成差异矩阵
   ↓
最终输出给用户
```

### 5.2 "状态管理"数据流（meta 视角）

```
本 skill 的状态 = 7 个 phase 文件 + 6 个 types 文件
                + 6 个 templates 文件
                + 1 个 SKILL.md 主索引

不变量:
- phases/ 总数 = 7
- types/ 总数 = 6
- templates/ 总数 = 6
- types/ 与 templates/ 一一对应
- phases/ 通过 @see 引用 types/
```

### 5.3 "路由守卫"数据流

```
Claude Code 看到 "扫描项目" / "理解项目结构" / "接手新项目" 等触发词
   ↓
命中 SKILL.md 顶部 frontmatter 的 description
   ↓
加载 SKILL.md
   ↓
按执行清单顺序跑 7 个阶段
   ↓
任一阶段失败 → 立即停止
   ↓
阶段 6 校验全绿 → 阶段 7（可选）
```

### 5.4 "核心页面"数据流

代表性场景："用户在新项目目录第一次输入 `/project-topology-mapping`"

```
[用户输入]
  ↓
[Claude Code 解析指令]
  ↓
[加载本 skill 的 SKILL.md frontmatter 匹配触发]
  ↓
[读取 SKILL.md 全文]
  ↓
[按执行清单跑阶段 1-7]
  ↓
[输出 .project-topology.md + 记忆更新 + 完成消息]
```

### 5.5 模块间关系

```
SKILL.md ←─── README.md
   ↑              ↑
   │              │
   └─── CHANGELOG.md
   ↑
   │
phases/ ←─── types/ ←─── templates/
   ↑                           
   │                           
   └─── examples/sample-topology.md
```

---

## 6. 入口点

| 命令 | 用途 | 备注 |
|------|------|------|
| `/project-topology-mapping` | 触发扫描 | Claude Code slash command |
| `git clone` | 安装 skill | 详见 README |
| `git pull` | 升级 | 在 skill 目录执行 |

---

## 7. 路由模块表

| 输入模式 | 触发 | 动作 |
|---------|------|------|
| "扫描项目" | SKILL.md frontmatter | 启动阶段 1 |
| "理解这个项目" | 同上 | 启动阶段 1 |
| "接手新项目" | 同上 | 启动阶段 1 |
| "修改 X 功能" | 不触发 | 跳过本 skill |

---

## 8. 依赖列表

无运行时依赖（纯文档 skill）。

工具链：
- Git（用于 `git ls-files`）
- Markdown 渲染器
- 可选：bash / zsh / Git Bash / WSL / PowerShell 7+

---

## 9. 定位速查表（本 skill 自身的定位）

> 注意：本表是"用本 skill 扫描本 skill"的元示例。

| 功能 | 文件 | 行数（约） |
|------|------|----------|
| 触发与平台 | SKILL.md | 200+ |
| 项目介绍 | README.md | 350+ |
| 阶段 1 | phases/phase1_detection.md | 130+ |
| 阶段 2 | phases/phase2_structure.md | 240+ |
| 阶段 3 | phases/phase3_dataflow.md | 300+ |
| 阶段 4 | phases/phase4_output.md | 150+ |
| 阶段 5 | phases/phase5_memory.md | 130+ |
| 阶段 6 | phases/phase6_validation.md | 100+ |
| 阶段 7 | phases/phase7_differences.md | 230+ |
| 前端检测 | types/frontend.md | 150+ |
| 后端检测 | types/backend.md | 570+ |
| 移动端检测 | types/mobile.md | 400+ |
| 小程序检测 | types/miniprogram.md | 270+ |
| 桌面端检测 | types/desktop.md | 320+ |
| HarmonyOS 检测 | types/harmonyos.md | 100+ |
| 前端模板 | templates/frontend_output.md | 100+ |
| 后端模板 | templates/backend_output.md | 100+ |
| HarmonyOS 模板 | templates/harmonyos_output.md | 100+ |
| 自指示例 | examples/sample-topology.md | ← 本文件 |

> **行数说明：** 表中"+"表示**行数级别**（基于 `(Get-Content).Count` 测量，向上取整到 10）。不是精确字数。文件可能随时更新，行数会变化。

---

## 10. 同类模块差异识别

> 文档项目本身**没有"逻辑步骤"差异**，但有"职责"差异。

| 阶段文件 | 职责 | 输出物 | 是否必填 |
|---------|------|--------|---------|
| phase1 | 检测项目类型 | 选 types/ | ✅ 必选 |
| phase2 | 结构扫描 + 差异预检 | 目录树 + 差异程度 | ✅ 必选 |
| phase3 | 数据流分析 | 数据流图 | ✅ 必选 |
| phase4 | 生成拓扑 | .project-topology.md | ✅ 必选 |
| phase5 | 存储到记忆 | 记忆文件 | ✅ 必选 |
| phase6 | 完整性校验 | checklist | ✅ 必选 |
| phase7 | 差异识别 | 差异矩阵 | ⚠️ 高度/极度差异时必选 |

---

## 11. 完成确认

```
✅ 拓扑扫描完成
- 项目类型: Claude Code Skill 文档项目
- 扫描深度: 快速
- 拓扑文件: .project-topology.md
- 记忆文件: ~/.claude/memory/projects/project-topology-mapping.md
- 模块差异: 无（文档项目无逻辑步骤差异）
- 下一步: 可以基于此拓扑继续维护本 skill
```

---

## 元数据

- **本示例生成方式**: 手动构造（保留作为模板参考）
- **实际生成时**: AI 会根据项目实际情况填充所有 `<占位符>`
- **参考价值**: 展示完整的输出结构、章节顺序、命名规范
