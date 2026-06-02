# 变更日志 (Changelog)

本项目所有显著变更都会记录在此文件。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

---

## [2.0.0] - 2026-06-01

### 🎉 重大重构

本版本是一次**完整的架构重构**，目标是把 skill 从"凭感觉堆功能"升级为"工程化、可维护、有边界"。

### ✨ 新增

- **SKILL.md 顶部 frontmatter** — `name` + `description` + `trigger`，让 Claude Code / Mavis 能自动触发
- **7 个独立 phase 文件** — 拆掉原 `phase4_output.md`（一个文件装 4 阶段），每个阶段独立成文
- **3 档扫描深度** — 快速（~2k token） / 标准（~8k） / 深度（~20k+）
  - 📌 **注意**：这里的 token 估算指**扫描用户项目时产生的 `.project-topology.md` 输出大小**，**不是** skill 自身文档大小。skill 自身文档是给 AI 阅读的，扫描用户项目时**只读取必要的子集**（按项目类型分发的检测命令 + 对应 phases），所以实际 token 消耗受扫描深度档位控制。
- **触发条件** — 顶部加 "When to use / When NOT to use" 明确边界
- **`types/` 作为 single source of truth** — phases/ 不再重复检测命令，改用 `@see` 引用
- **3 个新项目类型** — 移动端（Android/iOS/RN/Flutter）、小程序（微信/支付宝/抖音/百度）、桌面端（Electron/Tauri）
- **后端扩展** — types/backend.md 新增 Node/TS (Express/NestJS/Fastify/Koa)、.NET (ASP.NET Core)、Rust (Actix/Axum/Rocket) 三类
- **3 个新 templates** — mobile_output.md / miniprogram_output.md / desktop_output.md
- **`examples/sample-topology.md`** — 真实 worked example（自指示例）
- **完整 README** — Logo + 中文 + TOC + 安装/使用/卸载/升级/Troubleshooting/Star 引导
- **`CONTRIBUTING.md`** — 贡献指南
- **`CHANGELOG.md`** — 本文件
- **`.gitee/` + `.github/ISSUE_TEMPLATE/`** — 码云 / GitHub 镜像的 Issue 模板

### 🐛 修复

- **文件结构与文档声明不一致** — 拆 phase4 之前，SKILL.md 说"7 阶段"但 phases/ 只有 4 个文件
- **`phase2_structure.md` 章节编号错乱** — 两个 2.0，缺 2.7
- **`.project-topology.md` 与 `.gitignore` 矛盾** — 已 ignore 又在仓库里
- **Windows PowerShell `head` 不存在** — 改用 `git ls-files` 等跨平台写法 + 平台映射表
- **服务名交叉验证 4 处重复** — 集中到 types/backend.md
- **差异点检测命令 3 处重复** — 集中到各自 types/*.md
- **完整性 checklist 重复** — 集中到 phase6_validation.md
- **Agent 编号跳号** — 原 phase3 用 "Agent 6, 7, 8, 10"（跳 9），改用 task 命名
- **README 与 SKILL 项目类型覆盖不一致** — SKILL.md 速查表遗漏"移动端跨端 (uni-app/Taro)"与"桌面端 Tauri"两行，已同步 README
- **`superpowers:xxx` 集成点悬空** — 改为"可选"描述

### 📚 文档

- README 加 ASCII Logo + 一句话定位
- README 加码云 / GitHub 徽章
- README 加 TOC（章节目录）
- SKILL.md 加 "When to use" / "When NOT to use"
- SKILL.md 加 "扫描深度选择"
- SKILL.md 加 "平台兼容性" 小节 + Git Bash ↔ PowerShell 映射表
- 统一术语："变异点" → "差异点"（保留"变异"作为括号注释）

### 🔧 重构

- `phases/phase2_structure.md` 全部 bash 命令改用 `git ls-files` / `git grep`
- `types/*.md` 统一加"差异点检测"小节
- 阶段 1-3 全部使用 git 原生命令，跨 Windows / macOS / Linux

---

## [1.0.0] - 2026-05-18

### 🎉 首发

- 7 阶段执行流程
- 支持前端 / 后端 / 全栈 / HarmonyOS 4 种项目类型
- L1-L4 服务端深度扫描
- 前后端定位速查表
- 服务名交叉验证
- 同类模块差异识别

---

## 计划中 (Roadmap)

### [2.1.0] - 2026 Q3

- [ ] 新增 `types/kmp.md`（Kotlin Multiplatform）
- [ ] 新增 `types/smart-tv.md`（tvOS / Android TV）
- [ ] `types/` 增加 SQLite 数据库大小检测（>100MB 警告）
- [ ] `phases/phase2` 增加 "技术债务热力图"（基于 git log 改动频率）

### [3.0.0] - 2026 Q4

- [ ] 支持"增量扫描"模式（只扫描 git diff 的文件）
- [ ] 拓扑文件可视化（HTML / SVG 输出）
- [ ] 拓扑 diff（对比两个时间点的拓扑变化）

---

[1.0.0]: https://gitee.com/www_mao_com/project-topology-mapping/releases/tag/v1.0.0
[2.0.0]: https://gitee.com/www_mao_com/project-topology-mapping/releases/tag/v2.0.0
