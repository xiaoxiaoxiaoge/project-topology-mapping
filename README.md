# Project Topology Mapping

在开始实现之前，生成项目的**完整、持久化**结构图。让 AI 模型清晰了解代码位置、模块边界、依赖关系和数据流。

## 核心原则

**扫描一次，持久化，多次复用。**

## 项目结构

```
project-topology-mapping/
├── SKILL.md              # 主文件（索引 + 执行清单）
├── phases/
│   ├── phase1_detection.md     # 阶段1：项目类型检测
│   ├── phase2_structure.md    # 阶段2：结构扫描 + 变异点预检
│   ├── phase3_dataflow.md     # 阶段3：数据流分析
│   └── phase4_output.md      # 阶段4-7：生成拓扑 + 校验 + 存储 + 差异识别
├── types/
│   ├── frontend.md           # 前端项目检测命令
│   ├── backend.md            # 后端项目检测命令
│   └── harmonyos.md         # HarmonyOS项目检测命令
└── templates/
    ├── frontend_output.md    # 前端项目输出模板
    ├── backend_output.md     # 后端项目输出模板
    └── harmonyos_output.md  # HarmonyOS项目输出模板
```

## 支持的项目类型

| 项目类型 | 关键检测文件 |
|---------|-------------|
| 前端（React/Vue） | package.json, src/ |
| 前端（Angular） | package.json, src/app/ |
| 后端（Java/Spring） | pom.xml, src/main/java |
| 后端（Go） | go.mod, cmd/, pkg/ |
| 后端（Python/Django） | requirements.txt, views.py |
| HarmonyOS | entry/src/main/ets, *.ets |
| 全栈 | 同时存在前端和后端特征 |

## 执行流程

1. **阶段 1**: 检测项目类型
2. **阶段 2**: 结构扫描 + 变异点预检
3. **阶段 3**: 数据流分析
4. **阶段 4-6**: 生成拓扑文件并校验
5. **阶段 7**: 同类模块差异识别

详细命令见 [SKILL.md](SKILL.md)。

## 输出文件

| 文件 | 用途 |
|------|------|
| `.project-topology.md` | 完整拓扑（含数据流） |
| `~/.claude/memory/projects/<name>.md` | 项目参考 + 定位速查（记忆） |

## 主要功能

- **项目类型检测** — 自动识别前端/后端/全栈/HarmonyOS
- **变异点检测** — 发现同类模块间的逻辑差异
- **数据流分析** — HTTP请求流、状态流、路由守卫
- **前后端定位速查** — 快速定位功能修改位置
- **服务端深度扫描** — 深入到 Controller/Service 层