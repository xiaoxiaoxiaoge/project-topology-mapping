# Project Topology Mapping

[![Claude compatible](https://img.shields.io/badge/Claude-ready-blueviolet)](https://claude.com)
[![GitHub stars](https://img.shields.io/github/stars/xiaoxiaoxiaoge/project-topology-mapping?style=social)](https://github.com/xiaoxiaoxiaoge/project-topology-mapping)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow)](https://opensource.org/licenses/MIT)

> 让 AI 在开始编码前，先完整理解你的项目。

**核心原则：扫描一次，持久化，多次复用。**

传统开发流程中，AI 每次对话都要重新理解项目结构，浪费大量 token 和时间。这个 Skill 通过系统化扫描，生成**完整、持久化**的项目拓扑图——让 AI 真正"懂"你的代码。

---

## 功能特点

### 🔍 智能项目检测
自动识别前端/后端/全栈/HarmonyOS 项目类型，选择对应的检测策略，**不漏掉任何后端代码**。

### 📊 变异点检测
发现同类模块间的逻辑差异。例如：`trace/file-flow` 是2步流程，`trace/pc-screen` 是5步流程——AI 不会混淆它们。

### 🔗 前后端定位速查
| 前端页面 | → | 后端 Controller |
|---------|---|----------------|
| 取证列表 | → | `GwForensicController` |
| 水印嵌入 | → | `GwDocWatermarkController` |

修改功能时直接定位，无需猜测。

### 🕵️ 服务端深度扫描
深入到 Controller/Service 层，追踪 Feign 调用链路，理解微服务间依赖关系。

---

## 快速开始

### 1. 安装

将 `project-topology-mapping` 目录放入 `~/.claude/skills/`。

### 2. 执行扫描

```
/project-topology-mapping
```

### 3. 扫描流程

| 阶段 | 内容 | 产出 |
|------|------|------|
| Phase 1 | 项目类型检测 | 确定前端/后端/全栈 |
| Phase 2 | 结构扫描 + 变异点预检 | 目录结构、组件关系 |
| Phase 3 | 数据流分析 | HTTP请求流、状态流、路由守卫 |
| Phase 4-6 | 生成拓扑文件 | `.project-topology.md` |
| Phase 7 | 同类模块差异识别 | 模块差异表 |

### 4. 输出示例

```
.project-topology.md          # 完整拓扑（含数据流）
~/.claude/memory/            # 项目记忆（跨会话复用）
```

---

## 项目结构

```
project-topology-mapping/
├── SKILL.md                    # 主文件（索引 + 执行清单）
├── phases/
│   ├── phase1_detection.md     # 项目类型检测
│   ├── phase2_structure.md     # 结构扫描 + 变异点预检
│   ├── phase3_dataflow.md     # 数据流分析
│   └── phase4_output.md       # 生成拓扑 + 校验 + 存储
├── types/                      # 各类型项目检测命令
├── templates/                  # 输出模板
└── README.md
```

---

## 支持的项目类型

| 类型 | 关键特征 | 检测命令 |
|------|---------|---------|
| 前端 React/Vue | `package.json`, `src/` | 前端专用检测 |
| 前端 Angular | `package.json`, `src/app/` | 前端专用检测 |
| 后端 Java/Spring | `pom.xml`, `src/main/java` | 深度扫描 Controller 层 |
| 后端 Go | `go.mod`, `cmd/`, `pkg/` | Handler + Middleware |
| 后端 Python/Django | `requirements.txt`, `views.py` | View + Model |
| HarmonyOS | `entry/src/main/ets`, `*.ets` | ArkTS + Native |
| 全栈 | 同时存在前端和后端特征 | **前后端都要检测** |

---

## 为什么需要这个 Skill？

| 痛点 | 传统方式 | 使用本 Skill |
|------|---------|-------------|
| AI 不懂项目 | 每次都要重新解释上下文 | 拓扑图一次性说明白 |
| 修改时定位难 | 搜索关键词猜测文件 | 定位速查表直接跳转 |
| 同类模块被混淆 | AI 假设所有模块逻辑相同 | 变异点检测识别差异 |
| 后端代码被忽略 | 用前端命令检测后端 | 分类型使用不同策略 |
| 重复扫描 | 每次对话都重新扫描 | 持久化 + 记忆复用 |

---

## 适用场景

- 🏗️ **新项目接手** — 快速理解代码结构和数据流
- 🐛 **Bug 修复** — 直接定位到问题所在文件
- ✨ **功能开发** — 了解模块边界和依赖关系
- 🔄 **重构规划** — 识别高变异区域和关键依赖

---

##License

MIT