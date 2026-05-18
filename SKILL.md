# 项目拓扑映射

## 概述

在开始实现之前，生成项目的**完整、持久化**结构图。这让模型清晰了解代码位置、模块边界、依赖关系和数据流——减少重复扫描的 token 消耗，提高代码定位准确性。

**核心原则：** 扫描一次，持久化，多次复用。

## ⚠️ 重要警告

**如果未完成拓扑扫描就开始实现，是在盲目工作。**

本技能的每个步骤都是**必选项**，不是可选项。数据流分析是**必须包含**的部分。

## 文件结构

```
project-topology-mapping/
├── SKILL.md              # 主文件（索引 + 执行清单）
├── phases/
│   ├── phase1_detection.md     # 阶段1：项目类型检测
│   ├── phase2_structure.md     # 阶段2：结构扫描 + 变异点预检
│   ├── phase3_dataflow.md      # 阶段3：数据流分析
│   └── phase4_output.md         # 阶段4-6：生成拓扑 + 校验 + 存储
├── types/
│   ├── frontend.md             # 前端项目检测命令
│   ├── backend.md              # 后端项目检测命令
│   └── harmonyos.md            # HarmonyOS项目检测命令
└── templates/
    ├── frontend_output.md      # 前端项目输出模板
    ├── backend_output.md       # 后端项目输出模板
    └── harmonyos_output.md    # HarmonyOS项目输出模板
```

## 执行清单

| 阶段 | 名称 | 产出 | 全栈项目额外要求 |
|------|------|------|----------------|
| 阶段 1 | 项目类型检测 | 确定项目类型（前端/后端/全栈/HarmonyOS） | 检测前端+后端特征 |
| 阶段 2 | 结构扫描 + 变异点预检 | 目录结构、组件关系、变异程度 | **服务端深度扫描（L2-L4层）** |
| 阶段 3 | 数据流分析 | HTTP请求流、状态流、路由守卫、核心页面流 | **前后端定位速查表** |
| 阶段 4 | 生成拓扑文件 | `.project-topology.md` | **多端项目分离生成** |
| 阶段 5 | 存储到记忆 | 项目摘要保存到记忆 | **包含微服务定位速查** |
| 阶段 6 | 完整性校验 | 逐项检查是否完整 | 验证定位速查完整性 |
| 阶段 7 | 同类模块差异识别 | 模块差异表（极度变异时） | - |

## 用户确认事项

**⚠️ 阶段 3 完成后，必须询问用户以下事项：**

### 1. 定位速查表详细程度

> 定位速查表需要生成**详细版**还是**精简版**？
> - **详细版**：包含每个前端页面的所有 API 调用（映射表行数多）
> - **精简版**：只包含高频核心 API（约 20-30 条）
> - **说明**：服务端路由不受此选项影响，始终为完整版

### 2. 多端项目文件分离

> 本项目涉及多个端（前端 + 后端），拓扑文件是否分离生成？
> - **分离**：生成 `.project-topology.md`（主索引）、`.project-topology-frontend.md`（前端）、`.project-topology-backend.md`（后端）
> - **合并**：生成单一 `.project-topology.md` 文件
> - **说明**：多端项目建议分离，便于不同团队/角色专注各自领域

## 快速开始

1. **阶段 1**: 检测项目类型 → 确定使用哪套检测命令
2. **阶段 2**: 执行结构扫描 + 变异点预检
3. **阶段 3**: 根据变异程度决定采样策略，执行数据流分析
4. **阶段 4-6**: 生成拓扑文件并校验
5. **阶段 7**: 如果有同类模块，识别差异

详细命令见各阶段文件。

---

## 项目类型速查表

| 项目类型 | 关键检测文件/目录 | 变异点检测重点 |
|---------|------------------|---------------|
| **前端（React/Vue）** | package.json, src/, pages/ | 页面 config.ts, 组件差异 |
| **前端（Angular）** | package.json, src/app/ | module.ts, component.ts |
| **后端（Java/Spring）** | pom.xml, src/main/java | Controller 方法数, Service 逻辑 |
| **后端（Go）** | go.mod, cmd/, pkg/ | Handler 路由, Middleware 链 |
| **后端（Python/Django）** | requirements.txt, views.py | View 方法, Model 字段差异 |
| **后端（Python/Flask）** | requirements.txt, app.py | Blueprint 路由, 装饰器 |
| **HarmonyOS** | entry/src/main/ets, *.ets | 页面路由, Native 调用 |
| **全栈** | 同时存在前端和后端特征 | 前端+后端变异点都要检测 |

## 常见错误

| 错误 | 结果 | 修复 |
|---------|--------|---------|
| 跳过项目类型检测 | 用前端命令检测后端代码，漏掉重要模块 | 阶段1必须先检测项目类型 |
| 跳过数据流分析 | 拓扑不完整，后续工作盲目 | 阶段3是必选项 |
| 每次都重新扫描 | 浪费 token | 先检查时间戳和 git 状态 |
| 不存储到记忆 | 重复扫描 | 扫描后始终更新记忆 |
| 缺少完整性校验 | 遗漏重要章节 | 阶段6逐项检查 |
| 不使用 Agent 并行 | 扫描慢且易遗漏 | 超过3个查询时用 Explore agent |
| **同类模块不同流程只分析一个** | 拓扑错误，后续工作基于错误假设 | 阶段7必须执行，检查所有子模块的配置/API |
| **不看项目类型用错检测命令** | 后端代码被忽略，全栈变"前端" | 阶段1必须检测全栈项目 |

## 集成点

**前置于：**
- `superpowers:brainstorming` — 确保问题分析前的项目上下文
- `superpowers:executing-plans` — 实现前验证路径

## 输出文件

| 文件 | 用途 | 是否 gitignore |
|------|---------|------------|
| `.project-topology.md` | 完整拓扑（含数据流） | 是 |
| `~/.claude/memory/projects/<name>.md` | 项目参考 + 定位速查 | 否（记忆） |

---

## 新增功能：前后端定位速查

**⚠️ 重要：这是本 skill 的核心产出之一，用于快速定位功能修改位置。**

### 定位速查表内容

| 前端页面/功能 | 前端文件 | 调用 API 路径 | 主服务 | 涉及微服务 | 后端关键文件 |
|-------------|---------|--------------|--------|-----------|------------|
| trace/file-flow | list.tsx, trace.tsx | /api/forensicService/* | gw-forensic-service | gw-biz-service, gw-docsy-service | GwForensicController.java |

### 定位速查的用途

1. **修改前端功能** → 直接定位到对应页面文件
2. **修改后端 API** → 直接定位到对应 Controller
3. **定位涉及服务** → 一次了解完整调用链路
4. **排查问题** → 快速定位哪个服务出了问题

### 定位速查生成时机

- **全栈项目必须生成**：前后端定位速查是全栈项目的必选项
- **阶段 3 数据流分析时同步生成**
- **存储到记忆文件**：永久保留，跨会话使用

---

## 新增功能：服务端深度扫描

**⚠️ 重要：服务端扫描必须深入到 Controller/Service 层，不能只停留在目录结构。**

### 服务端深度扫描要求

| 扫描层级 | 前端项目 | 后端项目 |
|---------|---------|---------|
| **L1 表面** | 目录结构 | 目录结构 |
| **L2 入口** | pages/*/*.tsx | *Controller.java |
| **L3 业务** | components/, hooks/, store/ | *Service.java, *ServiceImpl.java |
| **L4 细节** | config.ts 差异 | Feign 调用、事务配置 |

### 必须扫描的后端内容

**Java/Spring 项目：**
- 所有 Controller 的路由映射 (@RequestMapping, @GetMapping 等)
- 所有 Service 的 public 方法
- Feign Client 接口和 fallback
- @Transactional 注解使用场景
- 异常处理配置 (@ExceptionHandler, @ControllerAdvice)

**Go 项目：**
- 所有 Handler/Controller 文件
- 路由定义 (r.Get, r.Post, r.Group)
- Middleware 链
- 数据库模型差异

---

**违反此技能：** 如果在未完成本次会话项目的完整拓扑（包括数据流和项目类型）的情况下开始实现，你是在盲目工作——返回并先完成扫描。