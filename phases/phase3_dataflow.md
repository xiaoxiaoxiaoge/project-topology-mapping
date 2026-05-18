# 阶段 3: 数据流分析 + 前后端定位速查

**⚠️ 重要：阶段 2 的变异点预检结果决定了阶段 3 的采样策略。**

- 如果阶段 2 发现"无变异" → 阶段 3 只采样 1 个代表性页面/类
- 如果阶段 2 发现"极度变异" → 阶段 3 必须逐个分析所有子模块/控制器

**⚠️ 这是最容易被跳过的部分，也是最重要的改进点。**

**⚠️ 全栈项目必须生成"前后端定位速查表"，用于快速定位功能修改位置。**

---

## ⚠️ 用户确认：定位速查表详细程度

**在生成定位速查表之前，必须询问用户：**

> **定位速查表需要生成哪种版本？**
>
> | 版本 | 内容 | 适用场景 |
> |------|------|---------|
> | **详细版** | 每个前端页面的所有 API 调用（约 100+ 条映射） | 需要精确定位每个页面细节 |
> | **精简版** | 高频核心 API（约 20-30 条） | 快速概览，重点了解主要流程 |
>
> **注意**：
> - 服务端路由（后端 Controller）不受此选项影响，始终为**完整版**
> - 全栈项目默认建议**精简版**，减少冗余
> - 如果项目前端页面多、API 调用复杂，建议**精简版**
> - 如果需要前端到后端的完整映射追溯，建议**详细版**

---

## 3.1 HTTP 请求层分析（Agent 6）

### 前端项目
```bash
cat src/utils/request.ts 2>/dev/null || cat src/utils/api.ts 2>/dev/null
```

### 后端项目
```bash
# 检查 HTTP 客户端配置（RestTemplate, WebClient, OkHttp 等）
grep -rh "RestTemplate\|WebClient\|OkHttp\|HttpClient" src/ 2>/dev/null | head -20

# 检查过滤器/拦截器
find src -type f -name "*Filter.java" 2>/dev/null | head -10
find src -type f -name "*Interceptor.java" 2>/dev/null | head -10
```

**重点提取：**
- 请求拦截器（如何添加 token、tenantId / Authorization）
- 响应拦截器（错误处理、token 刷新）
- 通用数据流图

---

## 3.2 状态管理层分析（Agent 7）

### 前端项目
```bash
cat src/store/*.ts
```

### 后端项目
```bash
# 检查缓存配置（Redis, Caffeine, Guava 等）
grep -rh "@Cacheable\|@CacheEvict\|StringRedisTemplate\|RedisTemplate" src/ 2>/dev/null | head -20

# 检查会话管理
grep -rh "HttpSession\|@SessionAttribute\|session\|Cookie" src/ 2>/dev/null | head -20
```

**重点提取：**
- 每个 store 的状态结构 / 缓存键值结构
- 持久化方式（sessionStorage/localStorage/内存 / Redis/本地缓存）
- 被引用次数和主要消费者

---

## 3.3 路由守卫分析（Agent 8）

### 前端项目
```bash
cat src/hooks/useRouteGuard.ts 2>/dev/null
cat src/router/index.tsx 2>/dev/null
```

### 后端项目
```bash
# 检查安全配置（Spring Security, Shiro, JWT 等）
find src -type f \( -name "*SecurityConfig.java" -o -name "*ShiroConfig.java" -o -name "*WebSecurityConfig.java" \) 2>/dev/null | head -5

# 检查拦截器/过滤器链
grep -rh "FilterChain\|doFilter\|@PreAuthorize\|@Secured" src/ 2>/dev/null | head -20
```

**重点提取：**
- 路由守卫数据流 / 安全过滤链
- 权限校验逻辑
- 跳转流程图

---

## 3.4 核心页面数据流分析（关键改进！避免采样偏差）

**⚠️ 采样策略的致命缺陷：AI 容易假设"同类模块逻辑一致"，导致修改时出错。**

正确做法：**先识别模块内部是否存在变异点（不同子模块有不同逻辑），再决定采样数量。**

### 3.4.1 变异点分析策略

根据阶段 2 的变异点预检结果，决定采样数量：

| 变异程度 | 判断标准 | 采样策略 |
|---------|---------|---------|
| **无变异** | 所有子模块的配置/逻辑相同，无差异点 | 只采样 1 个代表性页面/类 |
| **轻度变异** | 有差异但步骤流程相似 | 采样 2-3 个代表性子模块 |
| **高度变异** | 存在动态生成、关键组件不同、步骤数差异大 | **每个子模块都要采样** |
| **极度变异** | 多种不同类型，每种步骤数差异大 | 必须生成完整的子模块差异表 |

### 3.4.2 前端项目采样命令

```bash
# 分析一个典型列表页
cat src/pages/xxx/list.tsx 2>/dev/null

# 分析一个表单提交页
cat src/pages/xxx/create.tsx 2>/dev/null

# 如果是极度变异（如 trace 模块）
# 必须逐个分析所有子模块
cat src/pages/trace/file-flow/list.tsx 2>/dev/null
cat src/pages/trace/pc-screen/list.tsx 2>/dev/null
cat src/pages/trace/Video/list.tsx 2>/dev/null
# ... 其他子模块
```

### 3.4.3 后端项目采样命令

```bash
# 分析一个典型 Controller
cat src/main/java/com/example/controller/xxxController.java 2>/dev/null

# 分析一个典型 Service
cat src/main/java/com/example/service/xxxService.java 2>/dev/null

# 如果是极度变异，必须逐个分析所有 Controller
cat src/main/java/com/example/controller/FileFlowController.java 2>/dev/null
cat src/main/java/com/example/controller/PcScreenController.java 2>/dev/null
# ... 其他 Controller
```

### 3.4.4 生成模块间差异表（极度变异时必须生成）

当发现模块内存在高度变异时，拓扑中必须包含以下表格：

**前端项目：**
```markdown
### [模块名] 取证流程差异分析

| 子模块 | 步骤数 | config.ts关键配置 | 核心组件 | 差异原因 |
|--------|--------|------------------|---------|---------|
| file-flow | 2步 | totalStep=2 | DocExtract | 文档直接解析 |
| pc-screen | 3-5步 | stepsFilter | ImgCrop | 5步(部分屏拍屏) |
| ... | ... | ... | ... | ... |
```

**后端项目：**
```markdown
### [模块名] API 流程差异分析

| 控制器 | 路径 | 主要方法 | 业务逻辑差异 |
|--------|------|---------|-------------|
| FileFlowController | /api/file-flow | postUpload, getDocExtract | 文档服务器端解析 |
| PcScreenController | /api/pc-screen | postCrop, getRotate | 屏幕区域裁剪，5步流程 |
| ... | ... | ... | ... |
```

**警告：不要假设"所有子模块步骤相同"。即使目录结构相似，逻辑也可能完全不同。**

---

## 3.5 模块间关系分析（Agent 10）

### 前端项目
```bash
grep -rh "WaterMarkEnum\|watermarkType" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -20
```

### 后端项目
```bash
# 检查模块间依赖关系
grep -rh "import\|@Autowired\|@Inject" src/ --include="*.java" 2>/dev/null | \
  grep "com\.example\." | \
  sed 's/.*import //g; s/;//g' | \
  sort | uniq -c | sort -rn | head -30
```

---

## 3.6 数据流分析输出格式

```markdown
### 1. HTTP请求数据流
```
请求 → 拦截器 → API → 响应拦截器 → 业务代码
```

### 2. Store/缓存 状态数据流
```
useUserStore: token/userinfo/userMenus → sessionStorage
useThemeStore: layout/colors → 内存
useConfigStore: systemConfig → 内存
```

### 3. 路由守卫/安全过滤链 数据流
```
访问 → Token校验 → 菜单加载 → 权限码校验 → 放行/重定向
```

### 4. 核心页面/Controller 数据流
（选取项目中最具代表性的页面/Controller，绘制完整数据流图）

### 5. 模块间关系
（展示核心模块间的依赖和调用关系）
```

---

## 3.7 前后端定位速查（全栈项目必须生成）

**⚠️ 这是全栈项目的核心产出，用于快速定位功能修改位置。**

### 定位速查表生成步骤

**步骤 1: 提取前端 API 调用**
```bash
# 列出所有前端 service 文件
find src/services -type f -name "*.ts" | sort

# 提取 API 路径（以 /api/ 开头的路径）
grep -rh "url: ['\"]\/api\/" src/services/ --include="*.ts" | \
  sed "s/.*url: ['\"]\/api\///g; s/['\"]//g" | sort | uniq
```

**步骤 2: 映射到后端 Controller**
```bash
# 搜索后端 Controller 中的路由映射
grep -rh "@RequestMapping\|@GetMapping\|@PostMapping" Service/*/src \
  --include="*.java" | grep -i "forensic\|watermark\|trace" | head -30
```

**步骤 3: 构建定位速查表**

```markdown
### 前后端定位速查表

| 前端页面/功能 | 前端文件 | 调用 API 路径 | 主服务 | 涉及微服务 | 后端关键文件 |
|-------------|---------|--------------|--------|-----------|------------|
| 取证列表 | pages/trace/*/list.tsx | /api/forensicService/v1/getList | gw-forensic-service | - | GwForensicController.java |
| 文档水印嵌入 | pages/middleware/word/* | /api/docWatermark/v1/embedFileWM | gw-biz-service | gw-docsy-service | GwDocWatermarkController.java |
| 图片水印嵌入 | pages/middleware/picture/* | /api/picWatermark/v1/embedPicWM | gw-biz-service | gw-picsy-service | GwPicWatermarkController.java |
```

### 定位速查表用途

| 场景 | 如何使用速查表 |
|-----|---------------|
| 修改取证功能 | 查找 "forensicService" → 定位到 gw-forensic-service |
| 修改水印嵌入 | 查找 "watermark" → 定位到 gw-biz-service + 对应 Go 服务 |
| 排查认证问题 | 查找 "oauth" → 定位到 gw-iam-service |
| 定位涉及服务 | 查看"涉及微服务"列 → 了解完整调用链路 |

### 微服务定位规则

**按 API 路径前缀定位：**
```
/api/forensicService/*     → gw-forensic-service
/api/docWatermark/*        → gw-biz-service
/api/picWatermark/*       → gw-biz-service  
/api/videoWatermark/*     → gw-biz-service
/api/traceability/*       → gw-biz-service
/api/oauth/*              → gw-iam-service
/api/clientService/*      → gw-biz-service (终端相关)
/api/license/*            → gw-license-manage-service
```

**按功能领域定位：**
```
取证流程        → gw-forensic-service
水印嵌入        → gw-biz-service (+ Go 算法服务)
/goofysy/docsy  → gw-docsy-service (Go)
/goofysy/picsy  → gw-picsy-service (Go)
/goofysy/videosy → gw-videosy-service (Go)
用户/认证/权限  → gw-iam-service
API 网关        → gw-gateway-service
```

### 完整调用链路示例

**取证流程调用链：**
```
前端 → gw-gateway-service → gw-forensic-service → 
  ├→ gw-biz-service (溯源对象)
  ├→ gw-docsy-service (文档算法 Go)
  ├→ gw-picsy-service (图片算法 Go)  
  └→ gw-videosy-service (视频算法 Go)
```

**水印嵌入调用链：**
```
前端 → gw-gateway-service → gw-biz-service → 
  ├→ gw-docsy-service (PDF/OFD/Office)
  ├→ gw-picsy-service (图片)
  └→ gw-videosy-service (视频)
```