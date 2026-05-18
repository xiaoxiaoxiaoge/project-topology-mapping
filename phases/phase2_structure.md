# 阶段 2: 结构扫描 + 变异点预检 + 服务端深度扫描

**原则：** 超过 3 个并行查询时，使用 `Explore` agent 并行扫描。
**关键改进：** 结构扫描时同步进行变异点预检，避免采样偏差。

---

## 2.0 服务端深度扫描要求（重要！）

**⚠️ 服务端扫描必须深入到 Controller/Service 层，不能只停留在目录结构。**

### 扫描层级定义

| 层级 | 前端项目 | 后端项目 |
|------|---------|---------|
| **L1 表面** | 目录结构 | 目录结构、pom.xml/go.mod |
| **L2 入口** | pages/*/*.tsx | **所有 *Controller.java 的路由映射** |
| **L3 业务** | components/, hooks/, store/ | **所有 *Service.java 的 public 方法** |
| **L4 细节** | config.ts 差异 | Feign Client、@Transactional、异常处理 |

### Java/Spring 必须扫描的内容

```bash
# 1. 所有 Controller 及路由映射（L2 入口）
find src -type f -name "*Controller.java" -exec echo "=== {} ===" \; -exec grep -h "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" {} \;

# 2. 所有 Service 接口的方法签名（L3 业务）
find src -type f -name "*Service.java" -exec echo "=== {} ===" \; -exec grep -h "public.*(" {} \;

# 3. Feign Client 接口（L4 细节）
find src -type f -name "*Feign.java" -exec echo "=== {} ===" \; -exec cat {} \;

# 4. @Transactional 使用场景
grep -rh "@Transactional" src/ --include="*.java" | sort | uniq

# 5. 异常处理配置
grep -rh "@ExceptionHandler\|@ControllerAdvice\|@RestControllerAdvice" src/ --include="*.java"

# 6. 读取 2-3 个关键 Controller 完整内容
cat src/.../ GwForensicController.java  # 读取示例
cat src/.../ GwDocWatermarkController.java
```

### Go 项目必须扫描的内容

```bash
# 1. 所有 Handler/Controller 文件
find . -type f -name "*handler*.go" -o -name "*controller*.go" | grep -v vendor

# 2. 路由定义
grep -rh "r\.Get\|r\.Post\|r\.Put\|r\.Delete\|r\.Group" . --include="*.go" | grep -v vendor | head -50

# 3. Middleware 链
grep -rh "Use\|Middleware\|func.*Middleware" . --include="*.go" | grep -v vendor

# 4. 读取主要路由文件
cat router/router.go
cat controllers/*_water_mark.go
```

---

## 2.0 根据项目类型选择检测命令

### 前端项目检测命令

```bash
# 1. 目录深度分析（使用4层深度）
find src -maxdepth 4 -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \) 2>/dev/null | head -150

# 2. 一级目录
find src -maxdepth 1 -type d | sort

# 3. pages 直接子模块
find src/pages -maxdepth 1 -type d | sort

# 4. pages 二级子模块（确保穿透 pages/trace/*/list.tsx）
find src/pages -maxdepth 2 -type d | sort

# 5. 组件目录
ls -la src/components/ src/layouts/ src/router/ 2>/dev/null

# 6. services 目录结构
find src/services -maxdepth 1 -type d | sort
find src/services -type f -name "*.ts" | sort

# 7. hooks 目录
find src/hooks -maxdepth 1 -type f -name "*.ts" | sort

# 8. store 目录
find src/store -maxdepth 1 -type f -name "*.ts" | sort

# 9. router modules
ls src/router/modules/*.tsx 2>/dev/null | wc -l
ls src/router/modules/*.tsx 2>/dev/null

# 10. package.json 分析
cat package.json 2>/dev/null | head -40
```

### 后端项目检测命令

```bash
# Java/Spring 项目
# 1. 源码目录结构
find src/main/java -maxdepth 5 -type f -name "*.java" 2>/dev/null | head -100

# 2. 包结构
ls -la src/main/java/ 2>/dev/null
find src/main/java -maxdepth 2 -type d | sort

# 3. 配置文件
ls -la src/main/resources/ 2>/dev/null
cat pom.xml 2>/dev/null | head -50

# 4. Controller 层
find src -type f -name "*Controller.java" 2>/dev/null | head -30

# 5. Service 层
find src -type f -name "*Service.java" 2>/dev/null | head -30

# 6. Repository 层
find src -type f -name "*Repository.java" 2>/dev/null | head -30

# Go 项目
# 1. 源码目录结构
find . -maxdepth 4 -type f -name "*.go" 2>/dev/null | grep -v vendor | head -100

# 2. 模块结构
ls -la cmd/ pkg/ internal/ 2>/dev/null

# 3. go.mod 分析
cat go.mod 2>/dev/null | head -30

# Python/Django 项目
# 1. 源码目录结构
find . -maxdepth 4 -type f -name "*.py" 2>/dev/null | grep -v venv | grep -v __pycache__ | head -100

# 2. 应用结构
ls -la apps/ core/ models/ views/ 2>/dev/null

# 3. settings.py
find . -maxdepth 3 -name "settings.py" 2>/dev/null | head -5

# 4. requirements.txt
cat requirements.txt 2>/dev/null | head -30
```

### HarmonyOS 项目检测命令

```bash
# 1. ArkTS 源码扫描（.ets 文件）
find entry/src/main/ets -maxdepth 4 -type f \( -name "*.ets" \) 2>/dev/null | sort

# 2. Native C++ 层扫描
find entry/src/main/cpp -maxdepth 2 -type f \( -name "*.cpp" -o -name "*.h" \) 2>/dev/null | sort

# 3. Ability 入口检测
find entry/src -type d -name "entryability" -o -type d -name "entrybackupability" 2>/dev/null

# 4. services 目录（ArkTS 服务层）
find entry/src/main/ets -type d -name "services" 2>/dev/null

# 5. models 目录（ArkTS 数据模型）
find entry/src/main/ets -type d -name "models" 2>/dev/null

# 6. pages 目录（ArkTS 页面）
find entry/src/main/ets -type d -name "pages" 2>/dev/null

# 7. Native 库检测（lib*.so）
ls -la entry/libs/ 2>/dev/null

# 8. 资源文件检测
find entry/src/main/resources -maxdepth 2 -type d | sort 2>/dev/null

# 9. 解析 ArkTS import 关系
grep -rh "^import " --include="*.ets" entry/src/main/ets/ 2>/dev/null | \
  grep "from ['\"]" | \
  sed "s/.*from ['\"]\.\.?\//\//g; s/['\"]//g" | \
  sort | uniq -c | sort -rn | head -50

# 10. Native 模块依赖检测（lib*.so）
grep -rh "lib.*\.so" --include="*.ets" entry/src/main/ets/ 2>/dev/null

# 11. 系统 API 依赖统计（@kit.XXX 引用次数）
grep -rh "@kit\." --include="*.ets" entry/src/main/ets/ 2>/dev/null | \
  sed 's/.*@kit\./@kit./g' | cut -d' ' -f1 | cut -d';' -f1 | \
  sort | uniq -c | sort -rn

# 12. 服务依赖关系检测（DatabaseService 根基）
grep -rh "DatabaseService" --include="*.ets" entry/src/main/ets/services/ 2>/dev/null

# 13. 数据流检测（图像处理流程）
grep -rh "PhotoPickerDialog\|RawFileToSandbox\|testNapi" --include="*.ets" entry/src/main/ets/pages/ 2>/dev/null
```

---

## 2.1 基础目录结构扫描（Agent 1）

根据阶段 1 识别的项目类型，使用对应的检测命令（见 2.0 节），然后执行：

```bash
# 1. 一级目录
find src -maxdepth 1 -type d | sort

# 2. 二级目录
find src -maxdepth 2 -type d | sort

# 3. 三级目录（确保穿透 pages/trace/*/list.tsx）
find src -maxdepth 3 -type d | sort

# 4. 一级 TypeScript 文件
find src -maxdepth 1 -type f \( -name "*.ts" -o -name "*.tsx" \) | sort

# 5. 关键目录内容
ls -la src/components/ src/layouts/ src/router/
```

---

## 2.2 页面模块扫描（Agent 2）

```bash
# 1. pages 一级子模块
find src/pages -maxdepth 1 -type d | sort

# 2. pages 二级子模块
find src/pages -maxdepth 2 -type d | sort

# 3. 所有页面 TSX 文件（按目录分组）
find src/pages -type f -name "*.tsx" | sort

# 4. 识别页面模块统一结构模式
find src/pages -name "list.tsx" | sed 's|/list\.tsx||' | xargs -I {} dirname {} | xargs -I {} basename {}
```

---

## 2.3 服务/Hooks/状态扫描（Agent 3）

```bash
# 1. services 所有文件
find src/services -type f -name "*.ts" | sort

# 2. services 子目录
find src/services -maxdepth 1 -type d | sort

# 3. hooks 所有文件
find src/hooks -type f -name "*.ts" | sort

# 4. store 文件
find src/store -type f -name "*.ts" | sort

# 5. constant 文件
find src/constant -type f -name "*.ts" | sort

# 6. enums 文件
find src/enums -type f -name "*.ts" | sort

# 7. utils 文件
find src/utils -type f -name "*.ts" | sort
```

---

## 2.4 组件引用关系扫描（Agent 4）

```bash
# 1. 组件目录完整结构
find src/components -type f \( -name "*.tsx" -o -name "*.ts" \) | sort

# 2. 共享组件被引用情况
grep -rh "from ['\"]@/components/" --include="*.tsx" --include="*.jsx" src/ 2>/dev/null | \
  sed "s/.*from ['\"]@\/components\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 3. hooks 被引用情况
grep -rh "from ['\"]@/hooks/" --include="*.tsx" --include="*.ts" src/ 2>/dev/null | \
  sed "s/.*from ['\"]@\/hooks\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 4. services 被引用情况
grep -rh "from ['\"]@/services/" --include="*.tsx" --include="*.ts" src/ 2>/dev/null | \
  sed "s/.*from ['\"]@\/services\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn | head -30

# 5. router modules 数量
ls src/router/modules/*.tsx 2>/dev/null | wc -l
```

---

## 2.5 路由/构建配置扫描（Agent 5）

```bash
# 1. router 目录结构
ls -la src/router/
ls -la src/router/modules/

# 2. 路由模块文件列表
ls src/router/modules/*.tsx

# 3. package.json 分析
cat package.json | head -40
```

---

## 2.6 变异点预检（针对页面模块的子模块变异检测）

**⚠️ 关键：不同项目类型的变异点检测命令不同！**

### 前端项目变异点检测命令

```bash
# 1. 列出所有子模块（识别有多少个同级的相似模块）
find src/pages/[module] -maxdepth 1 -type d | sort

# 2. 对比各子模块的 config.ts 中的关键配置（检测步骤数差异）
find src/pages/trace -name "config.ts" -exec echo "=== {} ===" \; -exec cat {} \;

# 3. 检查动态步骤生成机制（stepsFilter、totalStep 等）
grep -rh "totalStep\|stepsFilter\|steps\[" src/pages/[module]/*/config.ts

# 4. 检查关键组件是否只在某些子模块中存在（如 ImgCrop 只在 pc-screen）
grep -rh "ImgCrop\|ImgRotate\|ImgCorrect\|ImgExtract\|DocExtract" \
  src/pages/trace/*/trace.tsx | sort | uniq -c | sort -rn

# 5. 检查枚举或常量定义（WaterMarkEnum 等）映射到不同取证类型
grep -rh "WaterMarkEnum\|watermarkType" src/pages/trace/ --include="*.ts" --include="*.tsx" | head -20

# 6. API 调用差异（不同子模块调用的 services 是否不同）
grep -rh "from ['\"]@/services/" src/pages/trace/*/list.tsx | \
  sed "s/.*from ['\"]@\/services\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 7. 路由/跳转逻辑差异
grep -rh "router\.\|navigate\|useNavigate\|history\." src/pages/trace/*/trace.tsx | \
  sort | uniq

# 8. 表单处理差异
grep -rh "onFinish\|onSubmit\|handleSubmit" src/pages/trace/*/trace.tsx | \
  sort | uniq

# 9. 条件渲染差异
grep -rh "visible\|show\|display\|render" src/pages/trace/*/trace.tsx | \
  grep -E "step|Step" | sort | uniq

# 10. 页面间状态传递方式
grep -rh "useState\|useContext\|createContext\|Provider" src/pages/trace/*/trace.tsx | \
  sort | uniq -c | sort -rn

# 11. 错误处理逻辑差异
grep -rh "message\.\|notification\|Modal\.error\|onError" src/pages/trace/*/trace.tsx | \
  sort | uniq

# 12. 加载状态处理差异
grep -rh "loading\|isLoading\|Loading\|Spin" src/pages/trace/*/trace.tsx | \
  sort | uniq
```

### 后端项目变异点检测命令

```bash
# 1. 列出所有 Controller（识别有多少个同级的 API 模块）
find src -type f -name "*Controller.java" 2>/dev/null | sort

# 2. 检查各 Controller 的路由映射（@RequestMapping, @GetMapping 等）
grep -rh "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" \
  src/ 2>/dev/null | sort | uniq

# 3. 检查 Service 层的业务逻辑差异
find src -type f -name "*Service.java" 2>/dev/null | head -30
grep -rh "public.*\(.*\).*\{" src/*Service.java 2>/dev/null | head -50

# 4. 检查 Entity/Model 层的字段差异
find src -type f -name "*.java" \( -name "*Entity.java" -o -name "*Model.java" -o -name "*DO.java" \) 2>/dev/null | head -20

# 5. 检查是否有不同的数据源配置
grep -rh "@DataSource\|@Primary\|MultiDataSource" src/ 2>/dev/null

# 6. 检查事务配置差异
grep -rh "@Transactional\|@Transactional(readOnly" src/ 2>/dev/null | sort | uniq

# 7. 检查缓存配置差异
grep -rh "@Cacheable\|@CacheEvict\|@RedisCache" src/ 2>/dev/null | sort | uniq

# 8. 检查日志级别配置
grep -rh "log\.\|logger\." src/ 2>/dev/null | grep -E "debug|info|warn|error" | sort | uniq

# 9. 检查异常处理差异
grep -rh "@ExceptionHandler\|@ControllerAdvice\|@RestControllerAdvice" src/ 2>/dev/null | sort | uniq

# 10. 检查验证注解差异（@Valid, @Validated）
grep -rh "@Valid\|@Validated" src/ 2>/dev/null | sort | uniq

# 11. 检查 Feign Client 调用关系（新增）
grep -rh "@FeignClient\|Feign\|fallback" src/ --include="*.java" 2>/dev/null | head -30

# 12. 检查微服务间调用链路（新增）
grep -rh "GwBIZFeign\|GwDocsyFeign\|GwPicsyFeign\|GwVideosyFeign" src/ --include="*.java" 2>/dev/null | head -20
```

### HarmonyOS 项目变异点检测命令

```bash
# 1. 列出所有页面（识别有多少个同级的页面模块）
find entry/src/main/ets/pages -maxdepth 1 -type d | sort

# 2. 检查各页面的路由配置
grep -rh "router\.\|push\|replace\|back" entry/src/main/ets/pages/ 2>/dev/null | sort | uniq

# 3. 检查 Ability 生命周期差异
grep -rh "onCreate\|onDestroy\|onWindowDisplay\|onFocus\|onBlur" \
  entry/src/main/ets/entryability/*.ets 2>/dev/null | sort | uniq

# 4. 检查服务层依赖差异
grep -rh "import.*services" entry/src/main/ets/pages/*.ets 2>/dev/null | \
  sed 's/.*services\///g; s/\.ets.*//g' | sort | uniq

# 5. 检查 Native 调用差异（lib*.so）
grep -rh "import.*\.so\|testNapi\|napi" entry/src/main/ets/ 2>/dev/null | \
  sort | uniq

# 6. 检查 UI 组件使用差异
grep -rh "@Component\|@State\|@Link\|@Prop" entry/src/main/ets/pages/*.ets 2>/dev/null | \
  sed 's/.*@//g' | cut -d' ' -f1 | sort | uniq -c | sort -rn

# 7. 检查数据持久化差异
grep -rh "relationalStore\|rdb\|Preferences" entry/src/main/ets/ 2>/dev/null | \
  sort | uniq

# 8. 检查网络请求差异
grep -rh "http\.\|request\|axios\|fetch" entry/src/main/ets/ 2>/dev/null | \
  sort | uniq
```

---

## 2.8 服务名交叉验证（必须执行！）

**⚠️ 关键警告：命名相似 ≠ 同一服务**

当项目涉及 Java 微服务调用 Go 服务时（如 Feign Client），容易犯的错误是：根据命名相似性推断服务关系，导致拓扑错误。

**错误示例：**
```
gw-docsy-service（Go）注册到 Eureka → GW-DOCSY-SERVICE
Java 侧常量 → GW_ALGORITHM_DZD_SERVICE_NAME = "T1SDK-MODULE-DZD-SERVER"

错误推断：gw-docsy-service 被 T1SDK-MODULE-DZD-SERVER 代理
实际上：这是两个完全不同的服务！
```

### 如何正确验证服务名关系

**步骤 1：找到所有服务名常量**
```bash
# 在 Java项目中搜索服务名常量定义
grep -rh "SERVICE_NAME.*=\|ServiceName.*=" src/ --include="*.java" 2>/dev/null | grep -E "T1SDK|GW-"
```

**步骤 2：找到 @FeignClient 的 name 值**
```bash
# 列出所有 Feign Client 及其服务名
grep -rh "@FeignClient.*name\|@FeignClient(name" src/ --include="*.java" 2>/dev/null
```

**步骤 3：找到 Go 服务的 Eureka 注册名**
```bash
# 在 Go 项目中搜索 server_name 配置
grep -rh "server_name:" --include="*.yaml" --include="*.yml" . 2>/dev/null

# 或搜索 eureka 注册代码
grep -rh "eureka\|Register\|ServerName" . --include="*.go" 2>/dev/null | grep -v vendor | head -20
```

**步骤 4：交叉验证**

| Java @FeignClient 的 name | Java 常量定义 | Go Eureka 注册名 | 结论 |
|---------------------------|--------------|-----------------|------|
| `T1SDK-MODULE-DZD-SERVER` | `GW_ALGORITHM_DZD_SERVICE_NAME` | `GW-DOCSY-SERVICE` | **不同服务，不要合并** |
| `GW-DOCSY-SERVICE` | `GW_DOCSY_SERVICE_NAME` | `GW-DOCSY-SERVICE` | **同一服务，可以关联** |

**规则：**
- 如果 `@FeignClient(name = "X")` 中的 `X` 与 Go 服务的 `server_name` 完全一致 → 同一服务
- 如果 `@FeignClient(name = "X")` 中的 `X` 与 Go 服务的 `server_name` 不同（如 `T1SDK-XXX` vs `GW-XXX`）→ **不同服务**，在拓扑中分开描述
- 命名相似（如都包含 "DZD"）**不是**合并的依据，必须验证实际注册名

### 后端项目检测命令（新增）

```bash
# 1. 查找服务名常量定义文件
find . -name "GwServiceNameConstant.java" -o -name "*ServiceNameConstant.java" 2>/dev/null | head -5

# 2. 读取常量文件，列出所有服务名
cat src/.../ GwServiceNameConstant.java 2>/dev/null | grep "public static final String.*SERVICE_NAME"

# 3. 列出所有 Feign Client 的服务名
find src -name "*Feign.java" -exec echo "=== {} ===" \; -exec grep "@FeignClient" {} \;

# 4. 查找 Go 服务的 server_name 配置
find . -name "config.yaml" -o -name "bootstrap*.yml" 2>/dev/null | xargs grep -l "server_name" 2>/dev/null
```

**检测结果记录格式：**

```markdown
### 服务名映射表

| Java 侧服务名 | Java 常量 | Go Eureka 注册名 | 是否同一服务 |
|--------------|----------|-----------------|------------|
| T1SDK-MODULE-DZD-SERVER | GW_ALGORITHM_DZD_SERVICE_NAME | GW-DOCSY-SERVICE | 否 |
| GW-DOCSY-SERVICE | GW_DOCSY_SERVICE_NAME | GW-DOCSY-SERVICE | 是 |
```

---

根据预检结果，判定变异程度：

| 变异程度 | 判断标准 | 采样策略 |
|---------|---------|---------|
| **无变异** | 所有子模块的配置/逻辑相同，无差异点 | 只采样 1 个代表性页面/类 |
| **轻度变异** | 有差异但步骤流程相似 | 采样 2-3 个代表性子模块 |
| **高度变异** | 存在动态生成、关键组件不同、步骤数差异大 | **每个子模块都要采样** |
| **极度变异** | 多种不同类型，每种步骤数差异大 | 必须生成完整的子模块差异表 |

**判定结果记录：**
```markdown
**变异程度:** <无变异/轻度变异/高度变异/极度变异>
**变异原因:** <具体差异点>
```