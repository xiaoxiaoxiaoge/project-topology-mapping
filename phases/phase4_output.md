# 阶段 4: 生成拓扑文件

**根据用户确认的事项生成拓扑文件：**

---

## 4.0 文件分离决策

**全栈项目（多端）** 默认生成**分离式拓扑文件**：

| 文件 | 内容 | 适用场景 |
|------|------|---------|
| `.project-topology.md` | 主索引文件，汇总信息 + 跳转链接 | 快速了解全项目 |
| `.project-topology-frontend.md` | 前端完整拓扑 | 修改前端页面/组件/状态 |
| `.project-topology-backend.md` | 后端完整拓扑 | 修改后端 Controller/Service/微服务 |

**单端项目**（纯前端或纯后端）生成**单一文件** `.project-topology.md`

---

## 4.1 分离式拓扑文件结构

### 主索引文件 `.project-topology.md`

```markdown
# 项目拓扑 - [项目名称]

## 项目拓扑索引

| 文件 | 内容 |
|------|------|
| [.project-topology-frontend.md](./.project-topology-frontend.md) | 前端完整拓扑 |
| [.project-topology-backend.md](./.project-topology-backend.md) | 后端完整拓扑 |

## 快速定位

| 定位需求 | 跳转文件 |
|---------|---------|
| 前端页面结构 | [.project-topology-frontend.md](./.project-topology-frontend.md) |
| 后端微服务结构 | [.project-topology-backend.md](./.project-topology-backend.md) |

## 技术栈概览

## 微服务概览

## 数据流概览

## 入口点
```

### 前端拓扑文件 `.project-topology-frontend.md`

```markdown
# 前端项目拓扑 - [项目名称]

**关联文件:** [.project-topology.md](./.project-topology.md) | [.project-topology-backend.md](./.project-topology-backend.md)

## 1. 基础信息
## 2. 目录结构
## 3. 页面模块地图
## 4. 组件关系图
## 5. Store 状态管理
## 6. 数据流图
## 7. API 服务层
## 8. 入口点
## 9. 依赖列表
## 10. 前端定位速查表（精简版）

[只包含前端到 API 的映射，高频核心 API]
```

### 后端拓扑文件 `.project-topology-backend.md`

```markdown
# 后端项目拓扑 - [项目名称]

**关联文件:** [.project-topology.md](./.project-topology.md) | [.project-topology-frontend.md](./.project-topology-frontend.md)

## 1. 基础信息
## 2. 目录结构
## 3. 微服务概览
## 4. 服务详情
## 5. 服务端完整路由清单
## 6. 取证策略实现
## 7. Feign 调用链路
## 8. 数据流图
## 9. 微服务定位速查
## 10. 入口点
## 11. 依赖列表
## 12. 存储
```

---

**必须包含的章节：**

- [ ] **4.1** 基础信息（生成时间、项目名称、语言、框架、项目类型）
- [ ] **4.2** 完整目录结构树
- [ ] **4.3** 模块地图表（模块名、路由前缀、子模块数、主要页面）
- [ ] **4.4** 组件关系图
  - [ ] 组件层级树
  - [ ] 页面模块统一结构模式（前端）/ 包结构（后端）
  - [ ] 共享组件引用表（Top 20 按引用次数排序）
  - [ ] 组件/类依赖关系示例
- [ ] **4.5** 数据流图（见阶段 3 的成果）
  - [ ] HTTP 请求数据流
  - [ ] Store/缓存 状态数据流
  - [ ] 路由守卫/安全过滤链 数据流
  - [ ] 核心页面/Controller 数据流（至少一个完整示例）
  - [ ] 模块间关系图
- [ ] **4.6** 入口点（命令、端口）
- [ ] **4.7** 路由模块表
- [ ] **4.8** 依赖列表
- [ ] **4.9** 前后端定位速查表（全栈项目必须）
  - **详细版**：每个前端页面的所有 API 调用（约 100+ 条）
  - **精简版**：高频核心 API（约 20-30 条）
- [ ] **4.10** 微服务定位速查（全栈项目必须）
- [ ] **4.11** 服务名交叉验证（后端项目必须）
  - [ ] 列出所有 Java @FeignClient 的 name 值
  - [ ] 列出所有服务名常量定义（如 GwServiceNameConstant）
  - [ ] 列出所有 Go 服务的 Eureka server_name
  - [ ] 验证@FeignClient name 与 Go server_name 是否指向同一服务
  - [ ] 如有不同服务（如 T1SDK-MODULE-* 和 GW-*），必须分开描述

各项目类型的输出模板见 `../templates/` 目录。

---

# 阶段 5: 存储到记忆

- [ ] **5.1** 将拓扑文件保存到 `.project-topology.md`
- [ ] **5.2** 将项目摘要添加到个人记忆

```markdown
## 项目参考

**项目路径:** /path/to/project
**项目类型:** <前端/后端/全栈/HarmonyOS>
**主要语言:** TypeScript / Java / Go / ArkTS
**关键目录:** src/api, src/core, src/components / src/main/java, src/main/resources
**组件结构:** Parent > Child > GrandChild
**共享组件:** shared/Modal, shared/Alert
**数据流特点:** React Query + Zustand + Axios / Spring Security + Redis
**入口点:** npm run dev, npm test / mvn spring-boot:run
**最后扫描:** YYYY-MM-DD

---

## 微服务定位速查

| 功能领域 | 主服务 | 关键路由 |
|---------|--------|---------|
| 取证流程 | gw-forensic-service | /api/forensicService/* |
| 水印嵌入 | gw-biz-service | /api/docWatermark/*, /api/picWatermark/* |
| 溯源对象 | gw-biz-service | /api/traceability/* |
| 用户/认证 | gw-iam-service | /api/oauth/* |
| API 网关 | gw-gateway-service | 16 条路由规则 |

### 涉及多个服务的调用链

**取证流程:**
前端 → gw-gateway → gw-forensic-service → gw-biz-service, gw-docsy-service (Go)

**水印嵌入:**
前端 → gw-gateway → gw-biz-service → gw-docsy/picsy/videosy (Go)
```

---

# 阶段 6: 完整性校验

**生成拓扑后，逐项检查：**

- [ ] 是否包含项目类型？
- [ ] 是否使用对应项目类型的检测命令？
- [ ] 是否包含完整目录结构树？
- [ ] 是否有模块地图表？
- [ ] 是否有组件层级树？
- [ ] 是否有共享组件引用表（Top 引用排行）？
- [ ] 是否有页面模块统一结构模式描述（前端）/ 包结构（后端）？
- [ ] 是否有数据流章节？
  - [ ] HTTP 请求数据流图
  - [ ] Store/缓存 状态数据流图
  - [ ] 路由守卫/安全过滤链 数据流图
  - [ ] 至少一个核心页面/Controller 的完整数据流
  - [ ] 模块间关系图
- [ ] 是否列出所有入口命令和端口？
- [ ] 是否有完整的依赖列表？
- [ ] **全栈项目：是否有前后端定位速查表？**
- [ ] **全栈项目：是否有微服务定位速查？**

**如果任何一项为"否"，必须补充后再继续。**

---

# 阶段 7: 同类模块不同流程的识别

**⚠️ 重要警告：这是最容易被遗漏的检查点**

当项目中有**多个相似名称的子模块**时（如 `trace/file-flow`、`trace/word`、`trace/Picture`），即使它们看起来结构相似，**内部逻辑可能完全不同**。

**必须执行的检查（根据项目类型选择）：**

## 前端项目检查命令：
```bash
# 1. 列出所有子模块的 config.ts
find src/pages/trace -name "config.ts" -exec echo "=== {} ===" \; -exec cat {} \;

# 2. 对比 totalStep 值
grep -rh "totalStep" src/pages/trace/*/config.ts

# 3. 检查是否有 stepsFilter 或动态步骤
grep -rh "stepsFilter\|steps\[" src/pages/trace/*/config.ts

# 4. 对比关键组件差异
grep -rh "ImgCrop\|ImgRotate\|ImgCorrect\|ImgExtract\|DocExtract" src/pages/trace/*/trace.tsx | sort | uniq -c | sort -rn
```

## 后端项目检查命令：
```bash
# 1. 列出所有 Controller
find src -type f -name "*Controller.java" 2>/dev/null

# 2. 对比各 Controller 的方法数
grep -rh "public.*\(.*\).*\{" src/*Controller.java 2>/dev/null | wc -l

# 3. 检查是否有不同的业务逻辑
grep -rh "@Transactional\|@Cacheable" src/*Controller.java 2>/dev/null

# 4. 对比 Service 层实现差异
find src -type f -name "*Service.java" 2>/dev/null | head -20
```

## HarmonyOS 项目检查命令：
```bash
# 1. 列出所有页面
find entry/src/main/ets/pages -maxdepth 1 -type f -name "*.ets" 2>/dev/null

# 2. 对比页面路由配置
grep -rh "router\|push\|replace" entry/src/main/ets/pages/*.ets 2>/dev/null

# 3. 检查 Native 调用差异
grep -rh "testNapi\|lib.*\.so" entry/src/main/ets/pages/*.ets 2>/dev/null
```

**识别结果记录格式：**

当发现同类模块有不同流程时，必须在拓扑中记录：

```markdown
### [模块名] 取证/API/页面 流程差异分析

| 子模块 | 步骤数 | config.ts关键配置 | 核心组件 | 差异原因 |
|--------|--------|------------------|---------|---------|
| file-flow | 2步 | totalStep=2 | DocExtract | 文档直接解析 |
| pc-screen | 3-5步 | stepsFilter | ImgCrop | 5步(部分屏拍屏) |
| ... | ... | ... | ... | ... |
```

**错误示例：**
> "trace 模块都是 2 步取证流程" （错误！只有 file-flow 是 2 步）

**正确做法：**
> 列出 10 种水印类型各自的取证步骤，说明差异原因

---

## 重新扫描条件

满足任一条件时必须重新扫描：

- 新源文件在未跟踪目录中
- 构建配置有变化
- 新增依赖
- 目录结构变化（深度 1-2 的新文件夹）
- 组件关系变化（新增/删除 import / import 语句）
- **数据流相关的文件变化**（如 request.ts、store 文件、路由守卫）
- **项目类型变化**（如从纯前端变为全栈）

检测变化：
```bash
git diff --name-only HEAD $(cat .topology-git-ref 2>/dev/null || echo HEAD~1) 2>/dev/null | grep -E "\.(ts|js|tsx|jsx)$"
```