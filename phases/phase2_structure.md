# 阶段 2: 结构扫描 + 差异点预检 + 服务端深度扫描

**原则：** 超过 3 个并行查询时，使用 `Explore` agent 并行扫描。
**关键改进：** 结构扫描时同步进行差异点预检（原文"变异点"），避免采样偏差。

> 前置：阶段 1 已识别项目类型。
> 后置：进入 [阶段 3: 数据流分析](./phase3_dataflow.md)

---

## 2.0 服务端深度扫描要求

**⚠️ 服务端扫描必须深入到 Controller/Service 层，不能只停留在目录结构。**

### 扫描层级定义

| 层级 | 前端项目 | 后端项目 |
|------|---------|---------|
| **L1 表面** | 目录结构 | 目录结构、pom.xml/go.mod |
| **L2 入口** | `pages/*/*.tsx` | **所有 `*Controller.java` 的路由映射** |
| **L3 业务** | `components/` `hooks/` `store/` | **所有 `*Service.java` 的 public 方法** |
| **L4 细节** | `config.ts` 差异 | Feign Client、`@Transactional`、异常处理 |

### Java/Spring 必须扫描的内容

```bash
# 1. 所有 Controller 及路由映射（L2 入口）
find src -type f -name "*Controller.java" -exec echo "=== {} ===" \; \
  -exec grep -h "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" {} +

# 2. 所有 Service 接口的方法签名（L3 业务）
find src -type f -name "*Service.java" -exec echo "=== {} ===" \; \
  -exec grep -h "public.*(" {} +

# 3. Feign Client 接口（L4 细节）
find src -type f -name "*Feign.java" -exec echo "=== {} ===" \; -exec cat {} +

# 4. @Transactional 使用场景
git grep -h "@Transactional" -- '*.java' 2>/dev/null | sort -u

# 5. 异常处理配置
git grep -h "@ExceptionHandler\|@ControllerAdvice\|@RestControllerAdvice" -- '*.java' 2>/dev/null

# 6. 读取 2-3 个关键 Controller 完整内容
cat src/main/java/com/example/controller/<Name1>Controller.java
cat src/main/java/com/example/controller/<Name2>Controller.java
```

### Go 项目必须扫描的内容

```bash
# 1. 所有 Handler/Controller 文件
find . -type f \( -name "*handler*.go" -o -name "*controller*.go" \) -not -path "./vendor/*"

# 2. 路由定义
git grep -h "r\.Get\|r\.Post\|r\.Put\|r\.Delete\|r\.Group" -- '*.go' 2>/dev/null \
  | grep -v vendor | head -n 50

# 3. Middleware 链
git grep -h "Use\|Middleware\|func.*Middleware" -- '*.go' 2>/dev/null | grep -v vendor

# 4. 读取主要路由文件
cat router/router.go
cat controllers/*_water_mark.go
```

> Node/TS / .NET / Rust / Python 的深度扫描命令见 [../types/backend.md](../types/backend.md)。

---

## 2.1 根据项目类型选择检测命令

不同项目类型使用**不同的检测命令集**。完整命令集见 `../types/` 目录。

| 项目类型 | 检测命令文档 |
|---------|------------|
| 前端（React/Vue/Angular） | [../types/frontend.md](../types/frontend.md) |
| 后端（Java/Go/Python/Node/.NET/Rust） | [../types/backend.md](../types/backend.md) |
| 移动端（Android/iOS/RN/Flutter） | [../types/mobile.md](../types/mobile.md) |
| 小程序（微信/支付宝/抖音/百度） | [../types/miniprogram.md](../types/miniprogram.md) |
| 桌面端（Electron/Tauri） | [../types/desktop.md](../types/desktop.md) |
| HarmonyOS | [../types/harmonyos.md](../types/harmonyos.md) |

**全栈项目：** 同时执行前端+后端检测命令。
**混合项目：** 根据主要用途选择 + 辅以次要类型的命令。

---

## 2.2 基础目录结构扫描 (Agent 1)

根据阶段 1 识别的项目类型，加载 `../types/` 下对应类型的检测命令（参见 [§ 2.1](#21-根据项目类型选择检测命令) 的类型映射表），然后执行：

```bash
# 1. 一级目录
git ls-files | awk -F/ '{print $1}' | sort -u

# 2. 二级目录
find . -maxdepth 2 -type d -not -path './.*' -not -path './node_modules*' | sort

# 3. 三级目录（穿透 pages/trace/*/list.tsx 这类）
find . -maxdepth 3 -type d -not -path './.*' -not -path './node_modules*' | sort

# 4. 一级 TypeScript 文件
git ls-files '*.ts' '*.tsx' '*.js' '*.jsx' | awk -F/ 'NF<=2' | sort

# 5. 关键目录内容
ls -la src/components/ src/layouts/ src/router/ 2>/dev/null
```

> 跨平台说明：用 `git ls-files` 替代 `find src -maxdepth N`，避免依赖系统 `find` 的差异。

---

## 2.3 页面模块扫描 (Agent 2)

```bash
# 1. pages 一级子模块
find src/pages -maxdepth 1 -type d | sort

# 2. pages 二级子模块
find src/pages -maxdepth 2 -type d | sort

# 3. 所有页面 TSX 文件（按目录分组）
git ls-files 'src/pages/*.tsx' | sort

# 4. 识别页面模块统一结构模式
git ls-files 'src/pages/*/list.tsx' | sed 's|/list\.tsx$||' | xargs -I {} basename {}
```

---

## 2.4 服务 / Hooks / 状态扫描 (Agent 3)

```bash
# 1. services 所有文件
git ls-files 'src/services/*.ts' | sort

# 2. services 子目录
find src/services -maxdepth 1 -type d | sort

# 3. hooks 所有文件
git ls-files 'src/hooks/*.ts' | sort

# 4. store 文件
git ls-files 'src/store/*.ts' | sort

# 5. constant / enums / utils 文件
git ls-files 'src/constant/*.ts' 'src/enums/*.ts' 'src/utils/*.ts' | sort
```

---

## 2.5 组件引用关系扫描 (Agent 4)

```bash
# 1. 组件目录完整结构
git ls-files 'src/components/*' | sort

# 2. 共享组件被引用情况
git grep -h "from ['\"]@/components/" -- '*.tsx' '*.jsx' 2>/dev/null | \
  sed "s/.*from ['\"]@\/components\///g; s/['\"].*//g" | sort | uniq -c | sort -rn

# 3. hooks 被引用情况
git grep -h "from ['\"]@/hooks/" -- '*.tsx' '*.ts' 2>/dev/null | \
  sed "s/.*from ['\"]@\/hooks\///g; s/['\"].*//g" | sort | uniq -c | sort -rn

# 4. services 被引用情况
git grep -h "from ['\"]@/services/" -- '*.tsx' '*.ts' 2>/dev/null | \
  sed "s/.*from ['\"]@\/services\///g; s/['\"].*//g" | sort | uniq -c | sort -rn | head -n 30

# 5. router modules 数量
ls src/router/modules/*.tsx 2>/dev/null | wc -l
```

---

## 2.6 路由 / 构建配置扫描 (Agent 5)

```bash
# 1. router 目录结构
ls -la src/router/ src/router/modules/ 2>/dev/null

# 2. 路由模块文件列表
ls src/router/modules/*.tsx 2>/dev/null

# 3. package.json 分析
head -n 40 package.json
```

---

## 2.7 差异点(变异点)预检

**⚠️ 关键：不同项目类型的差异点检测命令不同！**
**检测命令集是 single source of truth，本节只引用，不重复。**

| 项目类型 | 差异点检测命令 |
|---------|--------------|
| 前端 | [../types/frontend.md § 差异点检测](../types/frontend.md) |
| 后端 | [../types/backend.md § 差异点检测](../types/backend.md) |
| 移动端 | [../types/mobile.md § 差异点检测](../types/mobile.md) |
| 小程序 | [../types/miniprogram.md § 差异点检测](../types/miniprogram.md) |
| 桌面端 | [../types/desktop.md § 差异点检测](../types/desktop.md) |
| HarmonyOS | [../types/harmonyos.md § 差异点检测](../types/harmonyos.md) |

### 变异程度判定

| 变异程度 | 判断标准 | 阶段 3 采样策略 |
|---------|---------|---------------|
| **无变异** | 所有子模块的配置/逻辑相同，无差异点 | 只采样 1 个代表性页面/类 |
| **轻度变异** | 有差异但步骤流程相似 | 采样 2-3 个代表性子模块 |
| **高度变异** | 存在动态生成、关键组件不同、步骤数差异大 | **每个子模块都要采样** |
| **极度变异** | 多种不同类型，每种步骤数差异大 | 必须生成完整的子模块差异表 |

**判定结果记录：**
```markdown
**变异程度:** <无变异/轻度变异/高度变异/极度变异>
**变异原因:** <具体差异点>
```

---

## 2.8 服务名交叉验证（后端项目必做）

**⚠️ 关键警告：命名相似 ≠ 同一服务**

当项目涉及 Java 微服务调用 Go 服务时（如 Feign Client），容易犯的错误是：根据命名相似性推断服务关系，导致拓扑错误。

**完整的服务名交叉验证流程**（含错误示例、验证命令、结果格式）见 [../types/backend.md § 服务名交叉验证](../types/backend.md#服务名交叉验证)。

**简要提醒：**

| 验证步骤 | 关键动作 |
|---------|---------|
| 1. 找服务名常量 | 搜索 `*ServiceNameConstant.java` |
| 2. 找 @FeignClient name | 搜索 `@FeignClient(name` |
| 3. 找 Go Eureka 注册名 | 搜索 `server_name:` / `eureka` |
| 4. 交叉验证 | name 一致 → 同一服务；不同 → 分别描述 |
