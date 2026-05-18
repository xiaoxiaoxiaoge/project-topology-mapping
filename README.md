# Project Topology Mapping

[![Claude compatible](https://img.shields.io/badge/Claude-ready-blueviolet)](https://claude.com)
[![GitHub stars](https://img.shields.io/github/stars/xiaoxiaoxiaoge/project-topology-mapping?style=social)](https://github.com/xiaoxiaoxiaoge/project-topology-mapping)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow)](https://opensource.org/licenses/MIT)

> 让 AI 在开始编码前，先完整理解你的项目。

**核心原则：扫描一次，持久化，多次复用。**

传统开发流程中，AI 每次对话都要重新理解项目结构，浪费大量 token 和时间。这个 Skill 通过系统化扫描，生成**完整、持久化**的项目拓扑图——让 AI 真正"懂"你的代码。

---

## 功能特点

### 🔍 智能项目类型检测
自动识别前端/后端/全栈/HarmonyOS 项目类型，动态选择对应的检测策略。基于文件特征指纹（package.json、pom.xml、go.mod 等）和目录结构树双重验证，确保**不漏掉任何代码模块**。

### 📊 多维变异点检测
通过静态分析+动态采样双重机制，发现代码库中的"结构性异类"：
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
| 服务间调用 | Feign Client → 目标服务 → 服务名交叉验证 |

### 🕵️ 服务端深度扫描（L1-L4 四层穿透）
不止步于目录结构，深入业务核心层：

| 层级 | 前端项目扫描深度 | 后端项目扫描深度 |
|------|----------------|-----------------|
| **L1 表面** | 目录结构 | 目录结构 + 依赖配置 |
| **L2 入口** | pages/*/*.tsx | **所有 Controller 的路由映射** |
| **L3 业务** | components/hooks/store | **所有 Service 的 public 方法签名** |
| **L4 细节** | config.ts 差异 | **Feign 调用链、事务边界、异常处理配置** |

### 🌊 完整数据流分析
追踪数据在系统中的完整旅程：
- **HTTP 请求流** — 从前端 request utility 到后端 Controller 的完整链路
- **状态管理流** — Store/缓存层的读写策略与持久化机制
- **路由守卫流** — 安全过滤链的全路径（含权限校验、Token 刷新、跳转逻辑）
- **核心页面/Controller 数据流** — 选取代表性业务场景，绘制端到端数据流图

### ⚠️ 服务名交叉验证
微服务架构中常见"命名相似实则不同"陷阱：
- Java @FeignClient 的 `name` 值
- 服务名常量定义（如 `GwServiceNameConstant`）
- Go 服务的 Eureka 注册名

三重交叉验证，**防止误判服务归属**。

---

## 快速开始

### 1. 安装

将 `project-topology-mapping` 目录放入 `~/.claude/skills/`。

### 2. 执行扫描

```
/project-topology-mapping
```

### 3. 扫描流程

| 阶段 | 内容 | 核心价值 |
|------|------|---------|
| **Phase 1** | 项目类型检测 | 确定前端/后端/全栈，选择检测策略 |
| **Phase 2** | 结构扫描 + 变异点预检 | 目录结构、组件关系、**模块差异程度判定** |
| **Phase 3** | 数据流分析 | HTTP请求流、状态流、路由守卫、**定位速查表** |
| **Phase 4-6** | 生成拓扑文件 | `.project-topology.md` + **完整性逐项校验** |
| **Phase 7** | 同类模块差异识别 | **完整模块差异表**（极度变异时必选） |

### Phase 1：项目类型检测
```
检测策略选择器
├── 前端特征? → package.json / src/
├── 后端特征? → pom.xml / go.mod / requirements.txt
├── 全栈特征? → 同时存在 → 前后端检测都要执行
└── HarmonyOS 特征? → entry/src/main/ets / *.ets
```
通过多维特征向量识别，避免"误判项目类型导致后端代码被忽略"。

### Phase 2：结构扫描 + 变异点预检
```
目录结构树
├── L1 表面扫描
├── L2 入口定位
├── L3 业务分析
└── L4 细节提取
+
变异点预检矩阵
├── 步骤数差异检测
├── 组件分布检测
└── API 调用差异检测
```
判定变异程度：**无变异 → 轻度变异 → 高度变异 → 极度变异**，决定阶段 3 的采样策略。

### Phase 3：数据流分析
```
采样策略引擎
├── 无变异: 采样 1 个代表性模块
├── 轻度变异: 采样 2-3 个代表模块
├── 高度变异: 每个子模块都要采样
└── 极度变异: 生成完整模块差异表
+
数据流追踪
├── HTTP 请求层
├── 状态管理层
├── 路由守卫层
└── 核心页面采样
```
**⚠️ 采样策略基于变异程度动态调整，避免采样偏差导致拓扑错误。**

### Phase 4-6：拓扑生成 + 校验
```
输出文件
├── .project-topology.md          # 完整拓扑（含数据流）
├── .project-topology-frontend.md # 前端拓扑（多端项目）
└── .project-topology-backend.md  # 后端拓扑（多端项目）
+
完整性逐项校验清单
├── [ ] 项目类型是否正确
├── [ ] 目录结构是否完整
├── [ ] 模块地图是否覆盖所有模块
├── [ ] 组件层级树是否准确
├── [ ] 数据流章节是否包含 HTTP/Store/守卫/核心页面
├── [ ] 前后端定位速查表是否完整（全栈必选）
├── [ ] 微服务定位速查是否准确
└── [ ] 服务名交叉验证是否完成
```

### Phase 7：同类模块差异识别
```
⚠️ 最易遗漏的检查点

同类模块 ≠ 逻辑相同
├── trace/file-flow: 2步流程
├── trace/pc-screen: 5步流程
├── trace/word: 文档解析专属流程
└── trace/video: 视频处理专属流程

生成模块差异矩阵
| 子模块 | 步骤数 | config.ts | 核心组件 | 差异原因 |
|--------|--------|-----------|---------|---------|
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
│   ├── frontend.md            # 前端（React/Vue/Angular）
│   ├── backend.md             # 后端（Java/Go/Python）
│   └── harmonyos.md           # HarmonyOS
├── templates/                  # 输出模板
│   ├── frontend_output.md
│   ├── backend_output.md
│   └── harmonyos_output.md
└── README.md
```

---

## 支持的项目类型

| 类型 | 关键特征 | 检测深度 |
|------|---------|---------|
| 前端 React/Vue | `package.json`, `src/` | 页面 → 组件 → hooks → services |
| 前端 Angular | `package.json`, `src/app/` | Module → Component → Service |
| 后端 Java/Spring | `pom.xml`, `src/main/java` | Controller → Service → Feign |
| 后端 Go | `go.mod`, `cmd/`, `pkg/` | Handler → Middleware → Database |
| 后端 Python/Django | `requirements.txt`, `views.py` | View → Model → ORM |
| HarmonyOS | `entry/src/main/ets`, `*.ets` | ArkTS → Native (lib*.so) |
| 全栈 | 同时存在前端和后端特征 | **前后端深度扫描都要执行** |

---

## 为什么需要这个 Skill？

| 痛点 | 传统方式 | 使用本 Skill |
|------|---------|-------------|
| AI 不懂项目 | 每次都要重新解释上下文 | 拓扑图一次性说明白 |
| 修改时定位难 | 搜索关键词猜测文件 | 定位速查表直接跳转 |
| 同类模块被混淆 | AI 假设所有模块逻辑相同 | 变异点检测 + 差异矩阵 |
| 后端代码被忽略 | 用前端命令检测后端 | 分类型使用不同检测策略 |
| 重复扫描 | 每次对话都重新扫描 | 持久化 + 记忆系统复用 |
| 服务名误判 | 命名相似就合并 | 三重交叉验证服务归属 |

---

## 适用场景

- 🏗️ **新项目接手** — 快速理解代码结构和数据流，缩短上手周期
- 🐛 **Bug 修复** — 直接定位到问题所在文件，而非猜测搜索
- ✨ **功能开发** — 了解模块边界和依赖关系，避免改动引发连锁反应
- 🔄 **重构规划** — 识别高变异区域和关键依赖，制定安全重构策略

---

## License

MIT